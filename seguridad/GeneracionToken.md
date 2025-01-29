# Generacion de Token para API Latinium

## Endpoint
`POST /api/{tenantId}/auth`

### Descripción
Este endpoint permite generar un token JWT para un usuario autenticado en un contexto multi-tenant. Para obtener el token, es obligatorio proporcionar un **tenantId**, un **username** y una **password** válidos los cuales son proporcianos por la Empresa Elixir Software.

## Request Parameters
### Route Parameters
- `tenantId` (int): Identificador único para el inquilino

### Request Body
- `LoginModel`:
    - `Username` (string): Nombre del usuario a ingresar
    - `Password` (string): Contraseña del usuario a ingresar

## Respuesta
### Authenticacion Exitosa
- **Status Code**: 200 OK
- **Response Body**:

```json
{
  "token": "generated_jwt_token"
}
```