[[Estado de resultados]]
#canguro #odoo

Perfecto, vamos al grano. Odoo 17 sigue la misma lógica de contabilidad que versiones anteriores: todo está centralizado en la **contabilidad analítica y general**, con los movimientos contables en la tabla `account_move` y los detalles en `account_move_line`. Para un **estado de resultados** general, no necesitas todas las tablas contables, solo las relevantes para ingresos, costos y gastos. Te lo detallo:

---

## 1️⃣ Tablas principales

|Tabla|Uso principal|
|---|---|
|`account_move`|Contiene los **asientos contables**, cada registro es un asiento (factura, pago, ajuste, etc.).|
|`account_move_line`|Contiene las **líneas de los asientos**, aquí está el detalle de débitos y créditos por cuenta.|
|`account_account`|Lista de **cuentas contables** (ingresos, gastos, activos, pasivos).|
|`account_move_line_tag` + `account_account_tag`|Si usas **etiquetas contables** para agrupar cuentas (ej. gastos operativos). Opcional pero útil.|
|`res_company`|Para filtrar por compañía si hay multiempresa.|
|`res_currency`|Para manejar monedas si tu empresa usa más de una.|

---

## 2️⃣ Campos clave

### `account_move`

- `id` → PK
    
- `journal_id` → Diario contable
    
- `date` → Fecha del asiento
    
- `company_id` → Para filtrar empresa
    
- `state` → Debe ser `'posted'` para considerar solo asientos validados
    

### `account_move_line`

- `id` → PK
    
- `move_id` → FK a `account_move`
    
- `account_id` → FK a `account_account`
    
- `debit` → Débito
    
- `credit` → Crédito
    
- `balance` → Débito - Crédito
    
- `date` → Fecha de la línea (por si quieres filtrar por periodo)
    
- `analytic_account_id` → Opcional, si necesitas desglose analítico
    
- `company_id` → Por seguridad multiempresa
    

### `account_account`

- `id` → PK
    
- `code` → Código contable
    
- `name` → Nombre de la cuenta
    
- `user_type_id` → Tipo de cuenta (Ingreso, Gasto, Activo, Pasivo, Capital)
    
- `company_id` → Multiempresa
    

---

## 3️⃣ Cómo construir el estado de resultados

1. Filtrar solo **asientos validados**: `account_move.state = 'posted'`.
    
2. Filtrar solo **cuentas de resultados**: `account_account.user_type_id` = `'Income'` o `'Expense'`.
    
3. Sumar **débitos y créditos** por cuenta para el periodo deseado (`date`).
    
4. Calcular **saldo** (`sum(debit) - sum(credit)` o `balance`).
    
5. Agrupar cuentas por tipo (`Ingresos`, `Costos`, `Gastos`) y generar la jerarquía según el plan contable.
    

> 💡 Tip de rendimiento: no uses las vistas contables (`account_report` o `account_financial_report`) si la consulta tarda demasiado. Es mejor un query directo a `account_move_line` con joins hacia `account_account` y filtros de fecha, compañía y tipo de cuenta.

---

Si quieres, puedo armarte un **query SQL optimizado** para Odoo 17 que te genere directamente un estado de resultados general para un periodo específico, listo para exportar a Excel o PDF.

¿Quieres que haga eso?