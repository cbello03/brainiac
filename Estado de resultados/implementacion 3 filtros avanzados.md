[[implementacion 2 dinamico]]
Perfecto. Vamos a completar la versión del módulo para que tengas un **Estado de Resultados listo para presentar**, con **agregación por tipo de cuenta** (Ingresos y Gastos) y cálculo automático de la **Utilidad Neta** al final, directamente en la UI de Odoo 17. Esto hace que los usuarios puedan ver un P&L real sin depender del módulo pesado de contabilidad.

---

## 1️⃣ Ajuste de la vista SQL con jerarquía y utilidad neta

La idea es sumar los saldos por **tipo de cuenta** y añadir un registro final de utilidad neta:

```sql
CREATE OR REPLACE VIEW report_profit_loss AS (
    WITH lines AS (
        SELECT
            aml.id AS id,
            aml.account_id AS account_id,
            aa.code AS account_code,
            aa.name AS account_name,
            aut.name AS account_type,
            SUM(aml.debit - aml.credit) AS net_balance,
            aml.company_id AS company_id,
            aml.currency_id AS currency_id,
            aml.date AS date
        FROM account_move_line aml
        JOIN account_move am ON aml.move_id = am.id
        JOIN account_account aa ON aml.account_id = aa.id
        JOIN account_account_type aut ON aa.user_type_id = aut.id
        WHERE am.state = 'posted'
          AND aut.name IN ('Income', 'Expense')
        GROUP BY aml.id, aml.account_id, aa.code, aa.name, aut.name, aml.company_id, aml.currency_id, aml.date
    )
    SELECT *, net_balance AS balance_for_ui
    FROM lines

    UNION ALL

    SELECT
        0 AS id,
        NULL AS account_id,
        'UTILIDAD NETA' AS account_code,
        'UTILIDAD NETA' AS account_name,
        'Resultado' AS account_type,
        SUM(CASE WHEN account_type = 'Income' THEN net_balance ELSE -net_balance END) AS net_balance,
        company_id,
        currency_id,
        NULL AS date,
        SUM(CASE WHEN account_type = 'Income' THEN net_balance ELSE -net_balance END) AS balance_for_ui
    FROM lines
    GROUP BY company_id, currency_id
);
```

> 🔹 Explicación:
> 
> - `lines` = suma por cuenta y tipo.
>     
> - `UNION ALL` agrega un registro extra llamado **UTILIDAD NETA** calculada como: ingresos totales menos gastos totales.
>     
> - `balance_for_ui` = campo que muestra directamente el saldo que usarás en la UI.
>     

---

## 2️⃣ Ajustes en el modelo `profit_loss.py`

Agregamos `balance_for_ui` y filtramos para mostrar utilidad neta correctamente:

```python
from odoo import models, fields

class ProfitAndLoss(models.Model):
    _name = 'report.profit.loss'
    _description = 'Profit and Loss Report'
    _auto = False

    account_id = fields.Many2one('account.account', string='Cuenta')
    account_code = fields.Char(string='Código de Cuenta')
    account_name = fields.Char(string='Nombre de Cuenta')
    account_type = fields.Char(string='Tipo de Cuenta')
    net_balance = fields.Monetary(string='Saldo Neto', currency_field='currency_id')
    balance_for_ui = fields.Monetary(string='Saldo para UI', currency_field='currency_id')
    company_id = fields.Many2one('res.company', string='Tienda')
    currency_id = fields.Many2one('res.currency', string='Moneda')
    date = fields.Date(string='Fecha')
```

> Ahora `balance_for_ui` se puede usar en la vista para que aparezca ordenado junto a las cuentas, con un registro final de utilidad neta.

---

## 3️⃣ Ajustes en la vista tree para mostrar utilidad neta

```xml
<tree string="Estado de Resultados General">
    <field name="account_code"/>
    <field name="account_name"/>
    <field name="account_type"/>
    <field name="balance_for_ui"/>
</tree>
```

- Opcional: Puedes usar **Pivot view** para agrupar automáticamente por `account_type` y ver total de Ingresos, Gastos y Utilidad Neta.
    

---

## 4️⃣ Wizard sigue igual

El wizard (`profit.loss.wizard`) filtra por **empresa y rango de fechas**, y automáticamente el reporte muestra la **utilidad neta** al final.

---

✅ **Ventajas de esta versión final**

- Reporte interactivo en Odoo, sin depender del módulo contable pesado.
    
- Filtros dinámicos por **tienda** y **fecha**.
    
- Saldo agregado por cuenta y tipo de cuenta.
    
- Cálculo automático de **Utilidad Neta** visible en la UI.
    
- Compatible con Tree, Pivot y Gráficos para dashboards.
    

---

Si quieres, puedo hacer una **versión final con Pivot y Gráfico listos**, donde los usuarios vean **Ingresos, Gastos y Utilidad Neta** agrupados por tipo y por mes automáticamente, como un reporte profesional tipo ERP. Esto ahorra pasos y es completamente “plug & play” en Odoo.

¿Quieres que haga esa versión visual final también?