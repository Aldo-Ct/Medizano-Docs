# Microservicios y módulos

## Inventario técnico

| Componente | Puerto interno | Persistencia | Responsabilidad actual |
| --- | ---: | --- | --- |
| `frontend` | `80` | — | Angular 20, interfaz POS, Nginx y proxy `/api` |
| `api-gateway` | `8090` | — | Validación JWT, RBAC, CORS, enrutamiento y OpenAPI agregado |
| `eureka-server` | `8761` | — | Registro y descubrimiento de servicios |
| `usuario-ms` | `8087` | `medizano_usuarios_db` | Login, logout, usuarios, roles y auditoría |
| `catalogo-ms` | `8081` | `medizano_catalogo_db` | Productos, medicamentos, estado y códigos de barras |
| `cliente-ms` | `8084` | `medizano_clientes_db` | Registro y consulta de clientes |
| `inventario-ms` | `8085` | `medizano_inventario_db` | Lotes, vencimientos, stock y movimientos idempotentes |
| `orden-ms` | `8082` | `medizano_ordenes_db` | Órdenes pendientes/pagadas, postpago y reintentos |
| `pago-ms` | `8083` | `medizano_pagos_db` | Mercado Pago, PayPal, webhooks y conciliación |
| `facturacion-ms` | `8086` | `medizano_facturacion_db` | Ventas, IGV, comprobantes PDF, devoluciones y reportes |
| `postgres` | `5432` | Volumen `postgres_data` | Instancia PostgreSQL 16 con bases separadas |

## Módulos visibles de la aplicación

| Ruta Angular | Función | Roles principales |
| --- | --- | --- |
| `#/dashboard` | Resumen y accesos del rol | Usuario autenticado |
| `#/billing` | POS, carrito, escáner y cobros | `CASHIER`, `ADMIN` |
| `#/inventory` | Lotes, stock y alertas | `STOCK_MONITOR`, `ADMIN` |
| `#/medicines` | Alta y edición de medicamentos | `STOCK_KEEPER`, `ADMIN` |
| `#/returns` | Devoluciones | `CUSTOMER_SUPPORT`, `ADMIN` |
| `#/reports` | Ventas, caja, IGV y stock | `ANALYST`, `MANAGER`, `ADMIN` |
| `#/purchase-history` | Historial de compras | `MANAGER`, `ADMIN` |
| `#/user-activity` | Auditoría de actividad | `ADMIN` |
| `#/login-history` | Historial de acceso | `ADMIN` |
| `#/user-management` | Usuarios, estado y contraseña | `ADMIN` |

## Reglas relevantes del Gateway

- Solo el login, la configuración pública de las pasarelas, el webhook de Mercado Pago, health y OpenAPI son públicos.
- Las rutas administrativas requieren `ADMIN`, excepto reportes, que también admiten `ANALYST` y `MANAGER`.
- Las rutas de caja y pagos se limitan a los roles autorizados.
- El endpoint que confirma una orden pagada no es accesible desde el navegador: está reservado para `pago-ms` mediante un token interno.
- Los microservicios de negocio no publican sus puertos al host; el acceso normal pasa por el Gateway.

## API principal por dominio

| Dominio | Prefijos |
| --- | --- |
| Autenticación y usuarios | `/api/auth/**`, `/api/admin/users/**`, `/api/admin/audit/**` |
| Catálogo | `/api/v1/productos/**`, `/api/pharmacist/medicines/**` |
| Inventario | `/api/v1/inventario/**`, `/api/pharmacist/batches/**` |
| Clientes | `/api/v1/clientes/**` |
| Órdenes | `/api/v1/ordenes/**` |
| Pagos | `/api/v1/pagos/**` |
| Facturación | `/api/v1/facturacion/**`, `/api/cashier/bills/**` |
| Devoluciones y reportes | `/api/cashier/returns/**`, `/api/admin/reports/**` |

En desarrollo, la documentación OpenAPI agregada está disponible en `http://localhost:8090/swagger-ui.html`.
