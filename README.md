<p align="center">
	<a>
		<picture>
			<source height="120px" srcset="https://elixir.ec/Imagenes/Logo-elixir-blanco-512x512-1.webp" media="(prefers-color-scheme: dark)">
			<img src="https://elixir.ec/images/logo.png" loading="eager" />
		</picture>
	</a><br>
	<b>Una empresa líder en desarrollo de software que ofrece soluciones innovadoras en todas las industrias.</b>
</p>

<p align="center">
	<a href="https://www.facebook.com/Latinium"><img src="https://img.shields.io/badge/Facebook-1877F2?style=for-the-badge&logo=facebook" alt="Facebook" /></a>
	<a href="https://www.instagram.com/latiniumsoftware/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram" alt="Instagram" /></a>
	<a href="https://www.tiktok.com/@elixirsoftwarecompany"><img src="https://img.shields.io/badge/TikTok-000000?style=for-the-badge&logo=tiktok" alt="TikTok" /></a>
	<br>
	<a href="mailto:info@elixir.ec"><img alt="Email" src="https://img.shields.io/badge/Email-0078D4?style=for-the-badge&logo=microsoft-outlook" /></a>
	<a href="https://wa.me/593988233052"><img alt="Contact Us" src="https://img.shields.io/badge/Contact%20Us-28A745?style=for-the-badge&logo=whatsapp" /></a>
</p>


# Documentación API MultiTenant Latinium

Esta API permite gestionar la autenticación, facturación y pagos en un entorno multi-tenant para el sistema Latinium.

## Endpoints Principales

### Autenticación
- **Generación de Token:** [/api/{tenantId}/auth](/seguridad/GeneracionToken.md)

### Facturación
- **Creación de una factura:** [/api/{tenantId}/facturacion](/facturacion/NuevaFactura.md)
- **Creación de facturas en lote:** [/api/{tenantId}/facturacion/collection](/facturacion/NuevaFacturaLote.md)

### Pagos
- **Inserción de pagos:** [/api/{tenantId}/pagos](/pagos/NuevoPago.md)

