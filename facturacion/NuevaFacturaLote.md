
# API de Colección de Facturas Latinium

## Endpoint
`POST /api/{tenantId}/Facturacion/collection`

### Descripción
Permite crear múltiples facturas en una sola solicitud. Este endpoint procesa un array de facturas y aplica las mismas validaciones que el endpoint de factura individual para cada elemento de la colección.

### Autorización
- Acceso autenticado
- Requiere ID de inquilino/tenant válido en la ruta

### Estructura del Cuerpo de Solicitud
La solicitud debe contener un array de objetos de factura, donde cada objeto sigue la misma estructura que una factura individual.

```json
[
	{
		"cliente": {
			"nombre": "string",
			"direccion": "string",
			"ruc": "string",
			"email": "string",
			"telefono": "string",
			"idTipoRuc": 0
		},
		"proyecto": {
			"centroCosto": "string"
		},
		"subProyecto": {
			"nombre": "string",
			"codigo": "string"
		},
		"personal": {
			"cedula": "string"
		},
		"compra": {
			"fechaEmision": "2024-11-22T21:16:27.702Z",
			"idTipoFactura": 0,
			"bodega": 0,
			"servicio": 0,
			"notas": "string",
			"formaPago": "string",
			"comprobante": "string",
			"codigoComprobante": "string",
			"diasCredito": 0,
			"numero": "string"
		},
		"detalles": [
			{
				"articulo": {
					"nombre": "string",
					"codigo": "string"
				},
				"subProyecto": {
					"nombre": "string",
					"codigo": "string"
				},
				"proyecto": {
					"centroCosto": "string"
				},
				"cantidad": 0,
				"precio": 0,
				"descuento": 0,
				"bodega": 0,
				"codigoIva": "string"
			}
		]
	}
]
```

### Características Especiales
- Procesamiento por lotes
- Validación individual de cada factura
- Transacción atómica para toda la colección

### Reglas de Validación
- Se aplican todas las validaciones del endpoint de factura individual a cada elemento
- La colección no puede estar vacía
- Se recomienda un máximo de 100 facturas por solicitud para optimizar el rendimiento

### Respuestas

#### Éxito (200 OK)
```json
{
  "success": true,
  "message": "Se guardaron 2 facturas correctamente", 
  "data": [
    {
      "numero": "001002000000381",
      "success": true,
      "message": "Se guardó la factura con número: '001002000000381' correctamente"
    },
    {
      "numero": "001002000000382",
      "success": true,
      "message": "Se guardó la factura con número: '001002000000382' correctamente"
    }
  ],
  "time": 2345
}
```

#### Error (400 Bad Request)
```json
{
	"success": false,
	"message": "Se procesaron correctamente 2 facturas. Sin embargo, hubo inconvenientes al importar algunas facturas. Por favor, corrige los errores e inténtalo de nuevo.",
	"data": [
		{
			"numero": "001002000000381",
			"success": true,
			"message": "Se guardó la factura con número: '001002000000381' correctamente"
		},
		{
			"numero": "001002000000326",
			"success": false,
			"message": "Error procesando factura 001002000000326: Ya existe una factura registrada con el número especificado."
		},
		{
			"numero": "001002000000382",
			"success": true,
			"message": "Se guardó la factura con número: '001002000000382' correctamente"
		}
	],
	"time": 8908,
	"version": "1.2.0"
}
```

### Errores Específicos de Colección
| Mensaje de Error                                                                                                                                                       | Condición/Razón |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| `La colección de facturas no puede estar vacía`                                                                                                                        | Se envió un array vacío |
| `Se procesaron correctamente {cantidad} facturas. Sin embargo, hubo inconvenientes al importar algunas facturas. Por favor, corrige los errores e inténtalo de nuevo.` | Error específico en una factura de la colección |
| `Error al procesar el lote de facturas`                                                                                                                                | Error general en el procesamiento del lote |
