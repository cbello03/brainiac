DROP MATERIALIZED VIEW IF EXISTS report_profit_loss;

  

CREATE MATERIALIZED VIEW report_profit_loss AS

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

        (aml.debit - aml.credit)  * ((x.value)::numeric / 100.0) AS balance_allocated

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

ORDER BY account_code

WITH NO DATA;

  

CREATE UNIQUE INDEX IF NOT EXISTS report_profit_loss_uq

    ON report_profit_loss (analytic_account_id, account_id);