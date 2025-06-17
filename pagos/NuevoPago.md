[Inicio](../README.md)

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
    - Puede ser `Contado (1)` o `Credito (2)`

- **Numero**: Número de la factura (obligatorio)
    - Debe corresponder a una factura existente

- **IdTipoFactura**: Identificador del tipo de factura (obligatorio)
    - Debe coincidir con el tipo de factura existente

- **Pagos**: Lista de pagos a realizar (obligatorio)
    - **FormaPago**: Nombre de la forma de pago (obligatorio)
    - **Valor**: Monto del pago (obligatorio)
    - **Fecha**: Fecha del pago (obligatorio)
    - **Notas**: Notas adicionales del pago (opcional)
    - **Cheque**: Número de cheque si aplica (opcional)
    - **CuentaCorriente**: Número de cuenta corriente (opcional)
        - Si no se proporciona, se utilizará la cuenta asociada a la forma de pago (si existe)

#### Cuerpo de la Solicitud
```json
{
  "contadoCredito": 2,
  "numero": "string",
  "idTipoFactura": 0,
  "pagos": [
    {
      "formaPago": "string",
      "valor": 0,
      "fecha": "2024-03-21T00:00:00.000Z",
      "notas": "string",
      "cheque": "string",
      "cuentaCorriente": "string"
    }
  ]
}
```

# Reglas de Validación

## General
- La factura debe existir en el sistema con el número y tipo de factura especificados
- El total de todos los pagos (existentes + nuevos) no puede superar el total de la factura

## Validaciones de Forma de Pago
- El nombre de la forma de pago debe existir en la tabla `CompraFormaPago`
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
  "contadoCredito": 1,
  "numero": "001002000000752",
  "idTipoFactura": 1,
  "pagos": [
    {
      "notas": "nothing",
      "cheque": "nothing",
      "cuentaCorriente": "1234567898",
      "formaPago": "Efectivo",
      "valor": 1,
      "fecha": "2025-06-17T15:38:46.585Z"
    }
  ]
}
```

### Características Principales
- Transacciones atómicas
- Validación de montos totales
- Múltiples formas de pago
- Registro detallado de pagos
- Soporte para pagos en efectivo y crédito
- Gestión flexible de cuentas corrientes

### Errores Comunes
| Mensaje de Error | Condición/Razón |
|-----------------|-----------------|
| `No se encontro la compra con el numero: {numero}` | La factura especificada no existe |
| `El total de los pagos supera el total de la factura!` | La suma de pagos excede el valor de la factura |
| `No se encontro ningun metodo de pago con el nombre: {formaPago}` | La forma de pago especificada no existe |

