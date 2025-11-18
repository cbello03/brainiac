[[implementacion 3 filtros avanzados]]

Perfecto. Vamos a armar la **versión visual final del Estado de Resultados** en Odoo 17, lista para que los usuarios vean **Ingresos, Gastos y Utilidad Neta** agrupados por tipo de cuenta y por mes, en **Pivot y Gráfico**, totalmente interactivo. Esto convierte tu reporte en algo profesional tipo ERP.

---

## 1️⃣ Ajustes en la vista Pivot y Gráfico

### `views/profit_loss_views.xml`

Añadimos vistas Pivot y Gráfico:

```xml
<odoo>
    <!-- Vista Tree -->
    <record id="view_profit_loss_tree" model="ir.ui.view">
        <field name="name">report.profit.loss.tree</field>
        <field name="model">report.profit.loss</field>
        <field name="arch" type="xml">
            <tree string="Estado de Resultados General">
                <field name="account_code"/>
                <field name="account_name"/>
                <field name="account_type"/>
                <field name="balance_for_ui"/>
            </tree>
        </field>
    </record>

    <!-- Vista Pivot -->
    <record id="view_profit_loss_pivot" model="ir.ui.view">
        <field name="name">report.profit.loss.pivot</field>
        <field name="model">report.profit.loss</field>
        <field name="arch" type="xml">
            <pivot string="Estado de Resultados Pivot" 
                   measure="balance_for_ui"
                   colspan="1">
                <field name="account_type" type="row"/>
                <field name="account_name" type="row"/>
                <field name="date" type="col" interval="month"/>
            </pivot>
        </field>
    </record>

    <!-- Vista Gráfico -->
    <record id="view_profit_loss_graph" model="ir.ui.view">
        <field name="name">report.profit.loss.graph</field>
        <field name="model">report.profit.loss</field>
        <field name="arch" type="xml">
            <graph string="Estado de Resultados Gráfico" type="bar">
                <field name="account_type" type="col"/>
                <field name="balance_for_ui" type="measure"/>
                <field name="date" type="row" interval="month"/>
            </graph>
        </field>
    </record>

    <!-- Acción combinada -->
    <record id="action_profit_loss" model="ir.actions.act_window">
        <field name="name">Estado de Resultados</field>
        <field name="res_model">report.profit.loss</field>
        <field name="view_mode">tree,pivot,graph</field>
        <field name="help" type="html">
            <p>Reporte de Estado de Resultados por tienda y periodo</p>
        </field>
    </record>

    <!-- Menú -->
    <menuitem id="menu_profit_loss"
              name="Estado de Resultados"
              parent="account.menu_finance_reporting"
              action="action_profit_loss"/>
</odoo>
```

---

## 2️⃣ Ajustes finales en el wizard

El wizard sigue igual, pero ahora se puede abrir directamente el **Pivot o Gráfico**, mostrando:

- **Ingresos** (Income)
    
- **Gastos** (Expense)
    
- **Utilidad Neta** (resultado calculado en la vista SQL)
    

```python
def action_show_report(self):
    self.ensure_one()
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

> Con esto, la UI se vuelve totalmente interactiva: el usuario puede pivotear por tipo de cuenta, agrupar por mes y ver gráficos de barras de Ingresos vs Gastos vs Utilidad Neta.

---

## 3️⃣ Beneficios de esta versión final

- Reporte **profesional estilo ERP**, listo para mostrar en dashboards.
    
- Filtros dinámicos de **tienda** y **rango de fechas**.
    
- Saldo agregado por cuenta y tipo de cuenta.
    
- **Pivot y Gráfico listos**, con agrupación mensual y por tipo de cuenta.
    
- Cálculo automático de **Utilidad Neta** al final, visible en todos los modos.
    
- Carga ligera, ya que la agregación se hace en SQL.
    

---

Si quieres, puedo hacer un **diagrama rápido de flujo** que muestre cómo se mueve la información desde la base (`account_move_line`) hasta la UI final con Pivot y Gráfico. Esto te sirve para documentación o para mostrar al equipo.

¿Quieres que haga ese diagrama?

![[Pasted image 20251118155303.png]]