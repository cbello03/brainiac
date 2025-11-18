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