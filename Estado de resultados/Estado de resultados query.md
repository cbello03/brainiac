[[Estado de resultados]]
#odoo #canguro


```postgresql
-- Estado de Resultados General - Odoo 17
-- Reemplaza :company_id, :fecha_inicio, :fecha_fin por tus valores
SELECT
    aa.code AS cuenta_codigo,
    aa.name AS cuenta_nombre,
    CASE
        WHEN aut.name ILIKE '%Income%' THEN 'Ingreso'
        WHEN aut.name ILIKE '%Expense%' THEN 'Gasto'
        ELSE 'Otro'
    END AS tipo_cuenta,
    SUM(aml.debit) AS total_debito,
    SUM(aml.credit) AS total_credito,
    SUM(aml.balance) AS saldo
FROM
    account_move_line aml
JOIN
    account_move am ON aml.move_id = am.id
JOIN
    account_account aa ON aml.account_id = aa.id
JOIN
    account_account_type aut ON aa.user_type_id = aut.id
WHERE
    am.state = 'posted'                   -- solo asientos validados
    AND aml.company_id = :company_id      -- filtra tu empresa
    AND aml.date BETWEEN :fecha_inicio AND :fecha_fin
    AND aut.name IN ('Income', 'Expense') -- solo cuentas de resultados
GROUP BY
    aa.code,
    aa.name,
    aut.name
ORDER BY
    tipo_cuenta DESC,
    aa.code;


```

ajuste de tabla 
 account_account account_type
 
```postgresql
SELECT 
    aa.account_type,
    aa.code AS account_code,
    aa.name AS account_name,
    SUM(aml.debit) AS total_debit,
    SUM(aml.credit) AS total_credit,
    CASE 
        WHEN aa.account_type LIKE 'income%' THEN SUM(aml.credit) - SUM(aml.debit)
        WHEN aa.account_type LIKE 'expense%' THEN SUM(aml.debit) - SUM(aml.credit)
    END AS balance
FROM account_move_line aml
JOIN account_account aa ON aml.account_id = aa.id
JOIN account_move am ON aml.move_id = am.id
WHERE am.state = 'posted'
  AND aml.company_id = 1          -- empresa
  AND am.date BETWEEN '2025-01-01' AND '2025-12-31'
  AND aa.account_type IN ('income', 'income_other', 'expense', 'expense_depreciation', 'expense_direct_cost')
GROUP BY aa.account_type, aa.code, aa.name
ORDER BY aa.account_type, aa.code;

```


final
```postgresql

SELECT

aa.account_type,

aa.code AS account_code,

aa.name AS account_name,

CASE

WHEN aa.account_type LIKE 'income%' THEN 'ingreso'

WHEN aa.account_type LIKE 'expense%' THEN 'egreso'

ELSE 'otro'

END AS tipo_cuenta,

SUM(aml.debit) AS total_debit,

SUM(aml.credit) AS total_credit,

SUM(aml.balance) as balance2

FROM account_move_line aml

JOIN account_account aa ON aml.account_id = aa.id

JOIN account_move am ON aml.move_id = am.id

WHERE am.state = 'posted'

AND aml.company_id = 88 -- empresa

AND am.date BETWEEN '2025-01-01' AND '2025-12-31'

AND aa.account_type IN ('income', 'income_other', 'expense', 'expense_depreciation', 'expense_direct_cost')

GROUP BY aa.account_type, aa.code, aa.name

ORDER BY aa.account_type, aa.code;
```


view odoo

```postgresql
CREATE OR REPLACE MATERIALIZED VIEW report_profit_loss AS
SELECT
    aml.account_id,
    aa.code AS account_code,
    aa.name ->> 'es_VE' AS account_name,
    aa.account_type,
    SUM(aml.debit) AS total_debit,
    SUM(aml.credit) AS total_credit,
    SUM(aml.balance) AS net_balance,
    aml.company_id,
    rp.name AS company_name,
    aml.currency_id,
    rc.name AS currency_name,
    aml.date AS aml_date
FROM account_move_line aml
JOIN account_move am ON aml.move_id = am.id
JOIN account_account aa ON aml.account_id = aa.id
LEFT JOIN res_company rp ON aml.company_id = rp.id
LEFT JOIN res_currency rc ON aml.currency_id = rc.id
WHERE am.state = 'posted'
  AND aa.account_type IN (
      'income', 'income_other', 
      'expense', 'expense_depreciation', 'expense_direct_cost'
  )
GROUP BY 
    aml.account_id,
    aa.code, aa.name, aa.account_type,
    aml.company_id, rp.name,
    aml.currency_id, rc.name
    aml.date
WITH NO DATA;
```

index for materialized view

```postgresql
CREATE UNIQUE INDEX report_profit_loss_uidx
ON report_profit_loss (account_id, company_id, currency_id, aml_date);
```


final materialized view
```postgresql
CREATE OR REPLACE MATERIALIZED VIEW report_profit_loss AS
WITH expanded AS (
    SELECT
        aml.id AS line_id,
        aml.date AS line_date,
        aml.account_id,
        aa.code AS account_code,
        aa.name ->> 'es_VE' AS account_name,
        aa.account_type,
        aacc.id AS analytic_account_id,
        aacc.name ->> 'es_VE' AS analytic_account_name,
        (x.value)::numeric AS analytic_percent,
        aml.debit * ((x.value)::numeric / 100.0) AS debit_allocated,
        aml.credit * ((x.value)::numeric / 100.0) AS credit_allocated,
        aml.balance * ((x.value)::numeric / 100.0) AS balance_allocated
    FROM account_move_line aml
    JOIN account_move am ON am.id = aml.move_id
       AND am.state = 'posted'
    JOIN account_account aa ON aa.id = aml.account_id
    JOIN LATERAL jsonb_each_text(aml.analytic_distribution) AS x(key, value) ON x.key ~ '^[0-9]+$'
    JOIN account_analytic_account aacc ON aacc.id = x.key::int
    WHERE aa.account_type IN (
        'income',
        'income_other',
        'expense',
        'expense_depreciation',
        'expense_direct_cost'
    )
)
SELECT
    analytic_account_id,
    analytic_account_name,
    account_id,
    account_code,
    account_name,
    account_type,
    SUM(debit_allocated) AS total_debit,
    SUM(credit_allocated) AS total_credit,
    SUM(balance_allocated) AS net_balance
FROM expanded
-- Optional: filter by date dynamically via wizard parameters
-- WHERE line_date BETWEEN '2025-09-01' AND '2025-12-31'
GROUP BY
    analytic_account_id,
    analytic_account_name,
    account_id,
    account_code,
    account_name,
    account_type
ORDER BY account_code;
```