
# API de Pagos Latinium

## Endpoint
`POST /api/{tenantId}/pagos`

### Descripción
Registra pagos para una factura existente en el sistema Latinium, con validación integral de la forma de pago y el monto total.

### Autorización
- Acceso autenticado
- Requiere ID de inquilino/tenant válido en la ruta

### Estructura del Cuerpo de Solicitud

#### Campos Principales

- **ContadoCredito**: Tipo de pago (obligatorio)
    - Debe ser `Credito (2)`
    - No se permiten pagos de tipo `Contado (1)`

- **Numero**: Número de la factura (obligatorio)
    - Debe corresponder a una factura existente

- **IdTipoFactura**: Identificador del tipo de factura (obligatorio)
    - Debe coincidir con el tipo de factura existente

- **Pagos**: Lista de pagos a realizar (obligatorio)
    - **IdFormaPago**: Identificador de la forma de pago (obligatorio)
    - **Valor**: Monto del pago (obligatorio)
    - **Notas**: Notas adicionales del pago (opcional)
    - **Cheque**: Número de cheque si aplica (opcional)

#### Cuerpo de la Solicitud
```json
{
  "contadoCredito": 2,
  "numero": "string",
  "idTipoFactura": 0,
  "pagos": [
    {
      "idFormaPago": 0,
      "valor": 0,
      "notas": "string",
      "cheque": "string"
    }
  ]
}
```

# Reglas de Validación

## General
- El método de pago debe ser exclusivamente de tipo `Credito`
- La factura debe existir en el sistema con el número y tipo de factura especificados
- El total de todos los pagos (existentes + nuevos) no puede superar el total de la factura

## Validaciones de Forma de Pago
- El ID de forma de pago debe existir en la tabla `CompraFormaPago`
- Cada pago se registra individualmente en la base de datos

## Validaciones de Montos
- Se valida la suma de:
    - Pagos existentes de la factura
    - Nuevos pagos a registrar
- El total combinado no debe exceder el valor total de la factura

### Respuestas

#### Éxito (200 OK)
```json
{
  "success": true,
  "message": "Se guardaron los pagos de la factura '001-001-000123' correctamente!",
  "data": "001-001-000123",
  "time": 543
}
```

#### Error (400 Bad Request)
```json
{
  "success": false,
  "message": "El total de los pagos supera el total de la factura!",
  "data": null,
  "time": 234
}
```

### Ejemplo de Solicitud Completa
```json
{
  "contadoCredito": 2,
  "numero": "001-001-000123",
  "idTipoFactura": 1,
  "pagos": [
    {
      "idFormaPago": 1,
      "valor": 50.00,
      "notas": "Primer pago parcial",
      "cheque": "12345"
    },
    {
      "idFormaPago": 2,
      "valor": 25.00,
      "notas": "Segundo pago parcial"
    }
  ]
}
```

### Características Principales
- Transacciones atómicas
- Validación de montos totales
- Múltiples formas de pago
- Registro detallado de pagos

### Errores Comunes
| Mensaje de Error | Condición/Razón |
|-----------------|-----------------|
| `El metodo de pago debe ser Credito!` | Se intentó registrar un pago con método Contado |
| `No se encontro la compra con el numero: {numero}` | La factura especificada no existe |
| `El total de los pagos supera el total de la factura!` | La suma de pagos excede el valor de la factura |
| `No se encontro ningun metodo de pago con el Id: {idFormaPago}` | La forma de pago especificada no existe |

