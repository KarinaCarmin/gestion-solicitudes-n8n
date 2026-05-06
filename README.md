# 👕 Flujo N8N – Gestión de Solicitudes de Camisetas

Workflow desarrollado en **N8N** que automatiza el proceso completo de solicitudes de camisetas: desde la detección de nuevas respuestas en un Google Form, hasta la verificación de inventario, actualización de registros y notificación por email al solicitante.

## 📋 Descripción del proceso

Cuando alguien completa el formulario de solicitud, el flujo se activa automáticamente, valida la solicitud, verifica si hay stock disponible en la talla requerida, descuenta el inventario, marca la orden como verificada y envía un correo con el resultado.

## 🔧 Stack técnico

- **Herramienta:** N8N
- **Trigger:** Google Sheets Trigger (polling cada minuto)
- **Integraciones:** Google Sheets · Gmail
- **Lógica personalizada:** JavaScript (Code node)

## 🗂️ Google Sheets involucrados

| Hoja | Descripción |
|---|---|
| `Registro para la obtención de camisetas (Respuestas)` | Respuestas del formulario. Se actualiza con el estado de verificación |
| `T-shirt Inventory` | Inventario de camisetas por talla |

## ▶️ Flujo de ejecución

```
Google Sheets Trigger       → Detecta nuevas filas cada minuto
  └── Unir info             → Agrupa el trigger con las solicitudes
  └── Obtener solicitudes   → Lee todas las filas del formulario
  └── Filtrar registros     → Excluye filas sin email o ya verificadas
  └── Aggregate             → Consolida solicitudes pendientes
  └── Get Inventory         → Lee el inventario actual por talla
  └── Unir datos            → Junta solicitudes + inventario
  └── Reducción inventario  → (JS) Descuenta stock por talla y marca isSuccess
        ├── Extraer órdenes      → Separa detalle de cada pedido
        │     └── Marcar verificada la orden   → Actualiza estado en Sheets
        └── Extraer inventario   → Separa detalle del inventario actualizado
              └── Marcar verificada la orden1  → Actualiza stock en Sheets
  └── Unir                  → Sincroniza ambas ramas
  └── Enviar Correo         → Notifica al solicitante con tabla HTML del resultado
```

## 🧠 Lógica de negocio (Code Node)

El nodo `Reducción de inventario` contiene la lógica principal en JavaScript:

- Cruza cada solicitud con el inventario por talla
- Si hay stock disponible: descuenta 1 unidad y marca `isSuccess = true`
- Si no hay stock o no existe la talla: la orden queda sin aprobar
- Retorna el inventario actualizado y las órdenes procesadas en paralelo

## 📧 Email de notificación

Se envía un correo HTML por Gmail a cada solicitante con una tabla que incluye:

| Campo | Detalle |
|---|---|
| Nombre | Nombre del solicitante |
| Talla | Talla solicitada |
| Verificado | `true` si había stock, `false` si no |
| Fecha de verificación | Timestamp del momento de procesamiento |

## ⚙️ Configuración necesaria

1. Importar el archivo `.json` en tu instancia de N8N
2. Conectar las credenciales de **Google Sheets** y **Gmail** (OAuth2)
3. Actualizar los IDs de los documentos de Google Sheets con los propios
4. Activar el workflow
