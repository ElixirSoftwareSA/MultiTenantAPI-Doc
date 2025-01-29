This document was moved to [another location](../seguridad/GeneracionToken.md).

# API de Creación de Facturas Latinium

## Endpoint
`POST /api/{tenantId}/Facturacion`

### Descripción
Crea una nueva factura en el sistema Latinium con validación integral y procesamiento de detalles de cliente, proyecto, subproyecto, articulo y factura.

### Autorización
- Acceso autenticado
- Requiere ID de inquilino/tenant válido en la ruta

### Estructura del Cuerpo de Solicitud

#### Campos Principales

- **Cliente**: Información del cliente (obligatorio)
	- `Nombre`: Nombre del cliente
	- `Direccion`: Dirección del cliente
	- `Ruc`: Número de identificación fiscal (obligatorio para vincular o crear un cliente en la factura)
	- `Email`: Correo electrónico del cliente (obligatorio para nuevos clientes)
	- `Telefono`: Teléfono del cliente
	- `IdTipoRuc`: Tipo de identificación fiscal (obligatorio para nuevos clientes)

- **Proyecto**: Información del centro de costos (opcional)
	- `CentroCosto`: Nombre del centro de costos (obligatorio para vincular o crear un proyecto en la factura)

- **SubProyecto**: Información del proyecto (opcional)
	- `Nombre`: Nombre del subproyecto (obligatorio para nuevos subproyectos)
	- `Codigo`: Código del subproyecto (obligatorio para vincular o crear un subproyecto en la factura)

- **Personal**: Detalles del vendedor (opcional)
	- `Cedula`: Número de identificación fiscal (obligatorio para vincular un personal)

- **Compra**: Detalles de la compra (obligatorio)
	- `FechaEmision`: Fecha de emisión de la factura (opcional, fecha actual por defecto)
	- `IdTipoFactura`: Identificador de tipo de factura (obligatorio)
	- `Bodega`: Bodega de la compra (opcional)
	- `Servicio`: Servicio de la compra (opcional)
	- `Notas`: Notas de la compra (opcional)
	- `FormaPago`: Método de pago (opcional, efectivo por defecto)
	- `Comprobante`: Comprobante de la compra (opcional)
	- `CodigoComprobante`: Código del comprobante (obligatorio)
	- `DiasCredito`: Días de crédito (opcional)
	- `Numero`: Número de factura (opcional, si no se envía será generado)

- **Detalles**: Líneas de la factura (obligatorio)
	- **Articulo**: Detalles del producto (obligatorio)
		- `Nombre`: Nombre del producto (obligatorio para nuevos artículos)
		- `Codigo`: Código de producto (obligatorio)
	- **SubProyecto**: Información del proyecto (opcional)
		- `Nombre`: Nombre del subproyecto (obligatorio para nuevos subproyectos)
		- `Codigo`: Código del subproyecto (obligatorio para vincular o crear un subproyecto al detalle de la factura)
	- **Proyecto**: Información del centro de costos (opcional)
		- `CentroCosto`: Nombre del centro de costos (obligatorio para vincular o crear un proyecto al detalle de la factura)
	- `Cantidad`: Cantidad (obligatorio)
	- `Precio`: Precio unitario (obligatorio)
	- `Descuento`: Monto de descuento (opcional)
	- `Bodega`: Bodega de la línea (opcional)
	- `CodigoIva`: Código de impuesto (opcional)


#### Cuerpo de la Solicitud
```json
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
```
# Reglas de Validación

## General
- La información de **Cliente**, **Compra** y **Detalles de la Compra** es obligatoria para guardar la factura.
- El tipo de factura debe ser válido y existir en la base de datos.

## Validaciones de Cliente
- **RUC**:
	- Es obligatorio.
	- Debe ser único (validación en base de datos).
- **Email**:
	- Es obligatorio si el cliente no existe previamente.
- **IdTipoRuc**:
	- Debe ser válido y coincidir con el listado permitido:
		- **Proveedores**: `[1, 2, 3, 8]`
		- **Clientes**: `[4, 5, 6, 7, 8, 9]`
- **Validación por Tipo de Cliente**:
	- Si es proveedor y el `IdTipoRuc` no pertenece a los valores permitidos para proveedores, se lanza error.
	- Si no es proveedor y el `IdTipoRuc` no pertenece a los valores permitidos para clientes, se lanza error.

## Validaciones de Proyecto
- **Centro de Costo**:
	- Es obligatorio.
	- Si no existe un proyecto con ese Centro de Costo, se creará automáticamente.
- **Nombre del Proyecto**:
	- Es obligatorio al crear un nuevo proyecto.

## Validaciones de Subproyecto
- **Código**:
	- Es obligatorio.
- **Nombre del Subproyecto**:
	- Es obligatorio si el subproyecto no existe previamente.

## Validaciones de Personal
- **Cédula**:
	- Es obligatoria.
	- Debe coincidir con un registro existente en la base de datos.

## Validaciones de Compra
- **Días de Crédito**:
	- Si se especifican días de crédito, se calcula la fecha de vencimiento como `FechaEmision + DiasCredito`.
- **Código de Comprobante**:
	- Se establece automáticamente según el tipo de factura:
		- `IdTipoFactura = 1`: Código `18`.
		- `IdTipoFactura = 4`: Código `1`.
	- El comprobante debe existir en la base de datos.
- **Forma de Pago**:
	- Es opcional y debe coincidir con un registro válido en la base de datos:
		- La forma de pago debe coincidir en nombre, ignorando mayúsculas/minúsculas y espacios en blanco.
		- Se establece automáticamente el metodo de pago `EFECTIVO`.
- **Número de Factura**:
	- Se genera automáticamente si no se proporciona.
	- Debe configurarse la numeración del tipo de factura correspondiente en el sistema.
- **Datos de Compra**:
	- Debe incluir obligatoriamente los siguientes datos:
		- **IdTipoFactura**.

## Validaciones Generales del Proceso
- En caso de error en cualquier validación:
	- Se revierte la transacción y no se realiza ningún cambio en la base de datos.


### Respuestas

#### Éxito (200 OK)
```json
{
	"success": true,	
	"message": "Se guardó la factura con Id: '25836' correctamente",
	"data": 25836,
	"time": 654
}
```

#### Error (400 Bad Request)
```json
{
	"success": false,
	"message": "El Codigo del sub proyecto es obligatorio!",
	"data": null,
	"time": 732
}
```

### Ejemplo de Solicitud Completa
```json
{
	"cliente": {
		"ruc": "1400505101",
		"nombre": "ERIK PATRICIO PORTILLA PESANTEZ",
		"direccion": "TADAY Y CAHUASQUI",
		"email": "erikportilla.pesantez@outlook.es",
		"telefono": "0968176565",
		"IdTipoRuc": 5
	},
	"proyecto": {
		"centrocosto": "QUITO-CENTRO"
	},
	"personal": {
		"cedula": "1723012280"
	},
	"subProyecto": {
		"nombre": "La Argelia",
		"codigo": "ARGELIA-PROYECTO"
	},
	"compra": {
		"fechaEmision": "2024-11-22",
		"idTipoFactura": 1,
		"bodega": 1,
		"servicio": 1,
		"notas": "Nota de ejemplo para nueva factura",
		"formaPago": "EFECTIVO",
		"comprobante": "Comprobante de ejemplo para nueva factura",
		"codigoComprobante": "18",
		"diasCredito": 5,
		"Numero": "001001005478"
	},
	"detalles": [
		{
			"articulo": {
				"codigo": "ACF-131",
				"NOMBRE": "Azucar"
			},
			"cantidad": 4,
			"precio": 12,
			"descuento": 2,
			"impuesto": 15,
			"subProyecto": {
				"nombre": "Cahuasqui",
				"codigo": "CAHUASQUI-PROYECTO"
			},
			"proyecto": {
				"centroCosto": "GUALA-CENTRO"
			}
		},
		{
			"articulo": {
				"codigo": "JDE-512",
				"NOMBRE": "Cargador USB"
			},
			"cantidad": 10,
			"precio": 15,
			"descuento": 10,
			"impuesto": 0,
			"subProyecto": {
				"codigo": "CAHUASQUI-PROYECTO"
			}
		}
	]
}
```

### Características Principales
- Transacciones atómicas
- Creación automática de entidades
- Validaciones exhaustivas
- Optimización de rendimiento

### Errores Comunes
| Mensaje de Error | Condición/Razón                                                  |
|-----------------|------------------------------------------------------------------|
| `La informacion del cliente, compra y detalles de la compra son requeridas para guardar la factura!` | Falta etiqueta de cliente, compra o detalles de compra           |
| `Ingrese un Id de Tipo Factura valido!` | ID de tipo de factura inválido                                   |
| `No se puso procesar el Cliente!` | Error al procesar información del cliente                        |
| `No se puso procesar el Proyecto!` | Error al procesar información del proyecto                       |
| `No se puso procesar el SubProyecto!` | Error al procesar información del subproyecto                    |
| `No se puso procesar el Personal!` | Error al procesar información del personal                       |
| `El RUC del cliente es obligatorio!` | Falta el RUC (identificación fiscal) del cliente                 |
| `El cliente requiere un 'Email' para ser guardado!` | Falta el correo electrónico del cliente al crear un nuevo cliente |
| `El IdTipoRuc '{client.IdTipoRuc}' del cliente no es valido!` | ID de tipo de RUC inválido                                       |
| `El IdTipoRuc {client.IdTipoRuc} del cliente no es valido para Proveedores` | Tipo de RUC no válido para proveedores                           |
| `El IdTipoRuc {client.IdTipoRuc} del cliente no es valido para Clientes` | Tipo de RUC no válido para clientes                              |
| `El Nombre del proyecto es obligatorio!` | Falta el centro de costo (nombre) del proyecto                   |
| `El Codigo del sub proyecto es obligatorio!` | Falta el código del subproyecto                                  |
| `El Nombre del sub proyecto es obligatorio para crearlo!` | Se requiere nombre del subproyecto al crearlo                    |
| `La cedula del personal es obligatorio!` | Falta la cédula del personal                                     |
| `No se encontro ningun vendedor con la cedula '{personal.Cedula}'!` | No se encontró vendedor con la cédula proporcionada              |
| `El comprobante con el Codigo: {compra.CodigoComprobante} no existe!` | Código de comprobante de factura no existe                       |
| `No se encontro ningun metodo de pago con la forma de pago: {paymentMethodStr}` | No se encontró método de pago para el tipo especificado          |
| `Configure la numeracion del tipo factura: '{compra.IdTipoFactura}' o ingrese la numeracion de la factura!` | Numeración de factura no configurada                             |
| `No se generó un número de factura válido.` | Fallo al generar número de factura                               |
| `El detalle de la compra no tiene un articulo asignado!` | Detalle de compra sin artículo asignado                          |
| `Hubo un error al procesaro el articulo del detalle de la compra!` | Error al procesar el artículo en el detalle de compra            |
| `El codigo del iva en el detalle es invalido!` | Código de IVA inválido en detalle de compra                      |
| `El Codigo del Articulo es obligatorio!` | Código de artículo es obligatorio                                |
| `El campo 'nombre' es obligatorio para crear un nuevo Articulo!` | Nombre de artículo requerido al crear uno nuevo                  |
| `El número de pasaporte del cliente debe tener como mínimo 3 y máximo 13 caracteres!` | Longitud de número de pasaporte inválida                         |
| `El ruc del cliente dene tener 13 digitos!` | RUC de cliente debe tener 13 dígitos                             |
| `La cedula del cliente debe tener 10 digitos!` | Cédula de cliente debe tener 10 dígitos                          |

# API de Pagos Latinium

## Endpoint
`POST /api/{tenantId}/Pagos`

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
