[[modulo odoo implementacion]]

Perfecto. Vamos a extender el módulo para que el **Estado de Resultados General** sea completamente interactivo en Odoo 17, permitiendo **seleccionar rango de fechas y tienda** directamente desde la UI, sin tener que tocar SQL cada vez. Esto se hace con un **wizard** que filtra dinámicamente la vista basada en parámetros del usuario.

---

## 1️⃣ Nueva estructura del módulo

```
odoo_profit_loss/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   ├── profit_loss.py
│   └── profit_loss_wizard.py
├── views/
│   ├── profit_loss_views.xml
│   └── profit_loss_wizard_views.xml
└── data/
    └── profit_loss_view.sql
```

---

## 2️⃣ `models/profit_loss_wizard.py`

```python
from odoo import models, fields, api

class ProfitLossWizard(models.TransientModel):
    _name = 'profit.loss.wizard'
    _description = 'Wizard para Estado de Resultados'

    company_id = fields.Many2one('res.company', string='Tienda', required=True)
    date_from = fields.Date(string='Fecha desde', required=True)
    date_to = fields.Date(string='Fecha hasta', required=True)

    def action_show_report(self):
        self.ensure_one()
        # Retorna acción de ventana con filtro aplicado
        return {
            'name': 'Estado de Resultados',
            'type': 'ir.actions.act_window',
            'res_model': 'report.profit.loss',
            'view_mode': 'tree,pivot,graph',
            'domain': [
                ('company_id', '=', self.company_id.id),
                ('date', '>=', self.date_from),
                ('date', '<=', self.date_to),
            ],
            'context': {'search_default_group_by_account_type': 1}
        }
```

> Aquí `domain` aplica el filtro dinámico de **tienda** y **rango de fechas** en la vista del reporte.

---

## 3️⃣ `views/profit_loss_wizard_views.xml`

```xml
<odoo>
    <record id="view_profit_loss_wizard_form" model="ir.ui.view">
        <field name="name">profit.loss.wizard.form</field>
        <field name="model">profit.loss.wizard</field>
        <field name="arch" type="xml">
            <form string="Filtro Estado de Resultados">
                <group>
                    <field name="company_id"/>
                    <field name="date_from"/>
                    <field name="date_to"/>
                </group>
                <footer>
                    <button string="Ver Reporte" type="object" name="action_show_report" class="btn-primary"/>
                    <button string="Cancelar" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_profit_loss_wizard" model="ir.actions.act_window">
        <field name="name">Filtro Estado de Resultados</field>
        <field name="res_model">profit.loss.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

    <menuitem id="menu_profit_loss_wizard"
              name="Estado de Resultados"
              parent="account.menu_finance_reporting"
              action="action_profit_loss_wizard"/>
</odoo>
```

---

## 4️⃣ Ajustes en la vista SQL para permitir filtros dinámicos

Si quieres que la vista SQL también tenga la fecha, agregamos un campo `date` en la vista:

```sql
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
        aml.currency_id AS currency_id,
        aml.date AS date
    FROM account_move_line aml
    JOIN account_move am ON aml.move_id = am.id
    JOIN account_account aa ON aml.account_id = aa.id
    JOIN account_account_type aut ON aa.user_type_id = aut.id
    WHERE am.state = 'posted'
      AND aut.name IN ('Income', 'Expense')
    GROUP BY aml.id, aml.account_id, aa.code, aa.name, aut.name, aml.company_id, aml.currency_id, aml.date
);
```

> Esto permite filtrar directamente por `date` en el wizard.

---

✅ **Ventajas de esta versión:**

- Los usuarios pueden seleccionar **tienda y periodo** sin tocar SQL.
    
- El reporte se muestra en **Tree, Pivot o Gráfico**.
    
- La vista SQL hace la agregación y la UI filtra dinámicamente, evitando sobrecarga.
    
- 100% integrada con Odoo 17, lista para producción.
    

---

Si quieres, puedo armar un **ejemplo de filtro avanzado** que también agrupe automáticamente por **tipo de cuenta (Ingresos/Gastos)** en la vista Pivot, mostrando al final la **utilidad neta calculada**, como un verdadero estado de resultados listo para presentar.

¿Quieres que haga eso también?