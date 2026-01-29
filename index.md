# WhatsLead Clean — Documentación (MVP)

## Objetivo
WhatsLead Clean limpia, valida y prepara contactos de Excel para campañas de WhatsApp.  
Evita rechazos por teléfonos mal formateados, duplicados o con prefijos inválidos.

---

## Reglas de limpieza y validación (MVP)

### País por defecto
- Uruguay (UY)

### Normalización
1. Trim y limpieza de espacios
2. Remover todo lo no numérico, excepto `+` al inicio
3. Si empieza con `00`, se convierte a `+`
4. Si no tiene `+`, se asume Uruguay

### Formato final
- E.164 (ej: `+598XXXXXXXX`)

### Longitud
- Si el país es conocido: valida longitud por país (Uruguay por defecto)
- Si el país es desconocido: rango global de **8–15 dígitos**

### País desconocido
- Si no se puede determinar país, se marca como **`unknown`**
- No se invalida solo por ser `unknown`

### Duplicados
- Dedupe por número normalizado en E.164
- Se conserva la primera ocurrencia
- Los siguientes quedan con estado `duplicate`

### Estados finales
- `valid`: pasa formato y longitud
- `invalid`: falla formato o longitud
- `duplicate`: repetido aunque sea válido

---

## Contrato de salida (Preview + Export)

Campos por fila:
- `row_original` → número de fila original en Excel
- `value_original` → valor original recibido
- `value_normalized` → número normalizado (E.164)
- `country` → `UY` o `unknown`
- `status` → `valid` | `invalid` | `duplicate`
- `error_reason` → motivo (solo si es `invalid`)

### Preview
Muestra todas las filas con su estado.

### Export limpio
Solo filas con `status = valid` y `value_normalized`.

---

## Base de datos (Supabase + RLS)

### Tablas principales
- `organizations`
- `organization_members`
- `uploads`
- `processing_jobs`
- `processed_rows`
- `exports`
- `subscriptions`

### Multi-tenant
Todas las entidades dependen de `organization_id`.

### RLS (Row Level Security)
Acceso basado en membresía de organización:

- `organizations`: solo miembros pueden leer
- `organization_members`: solo admins/owner pueden gestionar miembros
- `uploads`, `processing_jobs`, `processed_rows`, `exports`: solo miembros de la org

### Subscripciones (Mercado Pago)
Tabla `subscriptions`:
- `provider` = `mercadopago`
- `mp_customer_id`
- `mp_preapproval_id`
- `plan` = `free` | `pro`
- `status` = `active` | `canceled` | `past_due` | `paused`

Escritura solo con `service_role`.

---

## Stack confirmado
- Front: Vue
- Back: Node.js + Express
- DB/Auth: Supabase (PostgreSQL)
- Pagos: Mercado Pago
- Deploy: Vercel (front) + Railway (back)

---

## Próximo paso
- Endpoints en Express para:
  1) Upload → parse → preview  
  2) Export limpio  
  3) Integración Mercado Pago

