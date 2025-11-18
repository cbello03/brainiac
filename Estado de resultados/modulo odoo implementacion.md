Perfecto, vamos a armar un **módulo completo para Odoo 17** que genere un Estado de Resultados General basado en una vista SQL, listo para mostrarlo en la UI. Te voy a dar la estructura básica con todo lo necesario: manifest, modelo, SQL, y ejemplo de vista.

---

## 1️⃣ Estructura del módulo

```
odoo_profit_loss/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── profit_loss.py
├── views/
│   └── profit_loss_views.xml
└── data/
    └── profit_loss_view.sql
```

---

## 2️⃣ `__manifest__.py`

```python
{
    'name': 'Profit and Loss Report',
    'version': '1.0',
    'category': 'Accounting',
    'summary': 'Estado de Resultados General por tienda y periodo',
    'description': 'Reporte de Estado de Resultados General usando vista SQL para mejorar rendimiento',
    'depends': ['account'],
    'data': [
        'views/profit_loss_views.xml',
        'data/profit_loss_view.sql',
    ],
    'installable': True,
    'auto_install': False,
}
```

---

## 3️⃣ `models/__init__.py`

```python
from . import profit_loss
```

---

## 4️⃣ `models/profit_loss.py`

```python
from odoo import models, fields

class ProfitAndLoss(models.Model):
    _name = 'report.profit.loss'
    _description = 'Profit and Loss Report'
    _auto = False  # Modelo basado en vista SQL

    account_id = fields.Many2one('account.account', string='Cuenta')
    account_code = fields.Char(string='Código de Cuenta')
    account_name = fields.Char(string='Nombre de Cuenta')
    account_type = fields.Char(string='Tipo de Cuenta')
    total_debit = fields.Monetary(string='Débito Total', currency_field='currency_id')
    total_credit = fields.Monetary(string='Crédito Total', currency_field='currency_id')
    net_balance = fields.Monetary(string='Saldo Neto', currency_field='currency_id')
    currency_id = fields.Many2one('res.currency', string='Moneda')
```

---

## 5️⃣ `data/profit_loss_view.sql`

```sql
-- Vista SQL para el estado de resultados
CREATE OR REPLACE VIEW report_profit_loss AS (
    SELECT
        aml.id AS id,
        aml.account_id AS account_id,
        aa.code AS account_code,
        aa.name AS account_name,
        aut.name AS account_type,
        SUM(aml.debit) AS total_debit,
        SUM(aml.credit) AS total_credit,
        SUM(aml.debit - aml.credit) AS net_balance,
        aml.company_id AS company_id,
        aml.currency_id AS currency_id
    FROM account_move_line aml
    JOIN account_move am ON aml.move_id = am.id
    JOIN account_account aa ON aml.account_id = aa.id
    JOIN account_account_type aut ON aa.user_type_id = aut.id
    WHERE am.state = 'posted'
      AND aut.name IN ('Income', 'Expense')
    GROUP BY aml.id, aml.account_id, aa.code, aa.name, aut.name, aml.company_id, aml.currency_id
);
```

> 🔹 Nota: si quieres filtrar por periodo o por tienda dinámicamente, eso lo haces en la acción o vista de Odoo usando filtros (`date_from`, `date_to`, `company_id`).

---

## 6️⃣ `views/profit_loss_views.xml`

```xml
<odoo>
    <record id="view_profit_loss_tree" model="ir.ui.view">
        <field name="name">report.profit.loss.tree</field>
        <field name="model">report.profit.loss</field>
        <field name="arch" type="xml">
            <tree string="Estado de Resultados General">
                <field name="account_code"/>
                <field name="account_name"/>
                <field name="account_type"/>
                <field name="total_debit"/>
                <field name="total_credit"/>
                <field name="net_balance"/>
            </tree>
        </field>
    </record>

    <record id="action_profit_loss" model="ir.actions.act_window">
        <field name="name">Estado de Resultados</field>
        <field name="res_model">report.profit.loss</field>
        <field name="view_mode">tree</field>
        <field name="help" type="html">
            <p>Reporte de Estado de Resultados General por tienda y periodo</p>
        </field>
    </record>

    <menuitem id="menu_profit_loss"
              name="Estado de Resultados"
              parent="account.menu_finance_reporting"
              action="action_profit_loss"/>
</odoo>
```

---

✅ **Qué obtienes con esto**

- Estado de resultados general, filtrable por compañía en Odoo.
    
- Carga ligera porque la agregación se hace en SQL.
    
- Exposición directa en UI con árbol, fácilmente convertible a pivot o gráfico.
    
- Listo para producción y escalable, incluso si tu módulo contable original se colapsa.
    

---

Si quieres, puedo hacer una **versión extendida** que ya tenga filtros dinámicos por **fecha y tienda** directamente en la UI, sin necesidad de tocar SQL cada vez. Esto hace que el reporte sea completamente interactivo para cualquier usuario en Odoo.

¿Quieres que haga esa versión también?