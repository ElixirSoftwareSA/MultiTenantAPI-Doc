[Inicio](../README.md)

# Generación de Token para API Latinium

## Endpoint
`POST /api/{tenantId}/auth`

## Descripción
Este endpoint permite generar un token JWT para un usuario autenticado en un contexto multi-tenant. Para obtener el token, es obligatorio proporcionar un **tenantId**, un **username** y una **password** válidos, los cuales son proporcionados por la empresa Elixir Software.

## Request Parameters

### Route Parameters
- `tenantId` (int): Identificador único para el inquilino.

### Request Body
- `LoginModel`:
  - `Username` (string): Nombre del usuario.
  - `Password` (string): Contraseña del usuario.

## Respuesta

### Autenticación Exitosa
- **Status Code**: `200 OK`
- **Response Body**:

```json
{
  "token": "generated_jwt_token"
}
```

### Errores

#### Credenciales Inválidas
- **Status Code**: `401 Unauthorized`
- **Response Body**:

#### Tenant no encontrado
- **Status Code**: `404 Not Found`
- **Response Body**:

```json
{
  "error": "TenantId no proporcionado en la URL."
}
```

