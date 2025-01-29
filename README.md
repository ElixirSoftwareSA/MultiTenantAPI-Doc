# Documentación API MultiTenant Latinium

Esta API permite gestionar la autenticación, facturación y pagos en un entorno multi-tenant.

## Endpoints Principales

### Autenticación
- **Generación de Token:** [/auth](/seguridad/GeneracionToken.md)

### Facturación
- **Creación de una factura:** [/facturacion](/facturacion/NuevaFactura.md)
- **Creación de facturas en lote:** [/facturacion/collection](/facturacion/NuevaFacturaLote.md)

### Pagos
- **Inserción de pagos:** [/api/{tenantId}/pagos](/pagos/NuevoPago.md)

---
