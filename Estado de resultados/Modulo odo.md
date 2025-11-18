Muy buena decisión pensar en algo más ágil para la UI de Odoo si el módulo contable se traba. Sí, tiene mucho más sentido usar un query SQL + una vista (o modelo _auto = False) para tu “estado de resultados general” y exponerlo en una UI de Odoo, en vez de depender de la carga pesada en el módulo estándar. Te explico cómo hacerlo técnicamente + qué recomiendo, desde la perspectiva de un senior developer.

---

## Cómo hacerlo de forma eficiente (SQL + modelo de vista en Odoo)

1. **Crear una vista SQL en la base de datos**
    
    - Define una vista (“VIEW”) con una consulta SQL que calcule los totales por cuenta (o por tipo de cuenta: ingresos, gastos) para el periodo o tienda que quieras.
        
    - Esa vista puede considerar `account_move_line`, `account_move`, `account_account`, `account_account_type` / `user_type` (dependiendo de cómo esté modelado en tu versión).
        
2. **Crear un modelo en Odoo que use esa vista**
    
    - En tu módulo custom de Odoo defines un modelo con `_auto = False`, lo que indica que no hay tabla física sino una vista SQL.
        
    - Eso te permite exponer la data de la vista en un tree view, pivot view o gráfico directamente en Odoo.
        
    - Puedes hacer un “wizard” para que el usuario seleccione el periodo (fecha inicio / fin) o la tienda y actualizar dinámicamente el reporte.
        
3. **Optimizar la consulta SQL**
    
    - Asegúrate de filtrar por `company_id` si tienes multiempresa (documentación de Odoo 17 lo soporta). ([Odoo](https://www.odoo.com/documentation/17.0/es_419/applications/finance/accounting.html?utm_source=chatgpt.com "Contabilidad y facturación — documentación de Odoo - 17.0"))
        
    - Filtra solo los asientos “posted” para evitar incluir borradores.
        
    - Agrega índices en `account_move_line` si es necesario, por ejemplo sobre `(date, company_id)` para acelerar el filtrado por periodo.
        

---

## Qué tablas y campos usar (resumen adaptado a UI)

- **`account_move_line`**:
    
    - `date` para filtrar por periodo.
        
    - `debit`, `credit`, o `balance` para los totales.
        
    - `account_id` para saber a qué cuenta pertenece la línea.
        
    - `move_id` para unirte con `account_move` para filtrar por estado del asiento.
        
    - `company_id` para filtrar por la tienda / compañía.
        
- **`account_move`**:
    
    - `id` para hacer el join con `account_move_line`.
        
    - `state` para filtrar solo asientos validados.
        
    - `date` si también quieres filtrar por fecha desde el movimiento (aunque con `account_move_line.date` suele bastar).
        
- **`account_account`**:
    
    - `id` para join con `account_move_line.account_id`.
        
    - `code` y `name` para mostrar el nombre y código de la cuenta.
        
    - `user_type_id` para determinar si es ingreso o gasto.
        
- **`account_account_type`** (o equivalente):
    
    - `id` para hacer join con `account_account.user_type_id`.
        
    - `name` o algún campo que te diga “Income / Expense / otros”.
        

---

## Ejemplo de cómo se vería la implementación (esquema)

- **SQL View**:
    
    ```sql
    CREATE OR REPLACE VIEW my_profit_and_loss AS (
      SELECT
        aa.id AS account_id,
        aa.code AS account_code,
        aa.name AS account_name,
        aut.name AS account_type,
        SUM(aml.debit) AS total_debit,
        SUM(aml.credit) AS total_credit,
        SUM(aml.debit) - SUM(aml.credit) AS net_balance
      FROM account_move_line aml
      JOIN account_move am ON aml.move_id = am.id
      JOIN account_account aa ON aml.account_id = aa.id
      JOIN account_account_type aut ON aa.user_type_id = aut.id
      WHERE am.state = 'posted'
        AND aml.company_id = %s
        AND aml.date BETWEEN %s AND %s
        AND aut.name IN ('Income', 'Expense')
      GROUP BY aa.id, aa.code, aa.name, aut.name
    );
    ```
    
- **Modelo Odoo**:
    
    ```python
    from odoo import models, fields
    
    class ProfitAndLoss(models.Model):
        _name = 'report.profit.loss'
        _auto = False  # es una vista SQL
        _description = 'Profit and Loss Report'
    
        account_id = fields.Many2one('account.account', string='Cuenta')
        account_code = fields.Char(string='Código de Cuenta')
        account_name = fields.Char(string='Nombre de Cuenta')
        account_type = fields.Char(string='Tipo de Cuenta')
        total_debit = fields.Monetary(string='Débito Total')
        total_credit = fields.Monetary(string='Crédito Total')
        net_balance = fields.Monetary(string='Saldo Neto')
    ```
    
    Con eso ya puedes definir una acción + vista (Tree, Pivot) para que el usuario seleccione el rango de fechas y la “tienda” (empresa) para ver el estado de resultados.
    

---

## Ventajas de este método para tu caso

- Evitas que el módulo contable de Odoo haga consultas muy pesadas por defecto.
    
- UI más rápida y responsiva: la vista SQL pre-agrega los datos, la carga es ligera.
    
- Flexibilidad para cambiar cómo se agrupan las cuentas (puedes editar la vista SQL cuando cambie tu esquema contable o plan de cuentas).
    
- Fácil de exponer en la UI estándar de Odoo sin depender de motores externos (no necesitas pandas ni un microservicio aparte).
    

---

Si quieres, puedo darte el **módulo completo listo para Odoo 17** (código Python + manifest + SQL) para que lo uses como base en tu instancia y te ahorres escribir todo desde cero. ¿Te lo preparo?