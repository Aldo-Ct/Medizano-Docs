# Cómo funciona la aplicación

## Diagrama general

Este diagrama representa el despliegue de producción y las relaciones que existen en el código actual.

```mermaid
flowchart TB
    Staff([Personal de botica])
    Customer([Cliente con celular])
    Internet((Internet))

    Staff -->|HTTPS| Internet
    Internet --> Caddy[Caddy<br/>TLS, HSTS y proxy inverso]
    Caddy --> Front[Angular 20 + Nginx<br/>SPA y proxy /api]
    Front -->|REST /api + JWT| Gateway[API Gateway :8090<br/>Autenticación, RBAC y rutas]

    subgraph Docker[Red privada Docker: medizano-net]
        Eureka[Eureka :8761<br/>Descubrimiento]
        User[usuario-ms :8087<br/>JWT, usuarios y auditoría]
        Catalog[catalogo-ms :8081<br/>Medicamentos]
        Client[cliente-ms :8084<br/>Clientes]
        Inventory[inventario-ms :8085<br/>Lotes y stock]
        Order[orden-ms :8082<br/>Órdenes]
        Payment[pago-ms :8083<br/>Pagos]
        Billing[facturacion-ms :8086<br/>Comprobantes, devoluciones y reportes]
        DB[(PostgreSQL 16<br/>una base por dominio)]

        Gateway --> User
        Gateway --> Catalog
        Gateway --> Client
        Gateway --> Inventory
        Gateway --> Order
        Gateway --> Payment
        Gateway --> Billing

        Gateway -. registro .-> Eureka
        User -. registro .-> Eureka
        Catalog -. registro .-> Eureka
        Client -. registro .-> Eureka
        Inventory -. registro .-> Eureka
        Order -. registro .-> Eureka
        Payment -. registro .-> Eureka
        Billing -. registro .-> Eureka

        User --> DB
        Catalog --> DB
        Client --> DB
        Inventory --> DB
        Order --> DB
        Payment --> DB
        Billing --> DB

        Order -->|consulta| Catalog
        Order -->|descuenta stock| Inventory
        Order -->|genera comprobante| Billing
        Payment -->|confirma con token interno| Order
        Billing -->|consulta productos y lotes| Catalog
        Billing -->|stock y reposición| Inventory
    end

    Payment <-->|Orders API v2| PayPal[PayPal Sandbox]
    Payment <-->|Checkout Pro y API de pagos| MP[Mercado Pago]
    Customer -->|escanea QR o abre enlace| MP

    subgraph Obs[Observabilidad opcional]
        Prom[Prometheus]
        Loki[Loki + Alloy]
        Grafana[Grafana]
        Prom --> Grafana
        Loki --> Grafana
    end

    Docker -. métricas y logs .-> Obs
```

## Camino de una solicitud

1. El navegador carga la aplicación Angular desde Nginx.
2. Caddy termina TLS en producción; Nginx sirve la SPA y reenvía `/api` al API Gateway.
3. Después del login, Angular adjunta el JWT a las solicitudes protegidas.
4. El Gateway valida firma, vigencia y rol, añade la identidad en cabeceras internas y resuelve el servicio mediante Eureka.
5. Cada microservicio aplica su regla de negocio y guarda únicamente los datos de su dominio en PostgreSQL.
6. Las operaciones entre servicios usan REST/OpenFeign. Las confirmaciones críticas entre `pago-ms`, `orden-ms`, `inventario-ms` y `facturacion-ms` incluyen controles de idempotencia y reintentos programados.

## Separación de responsabilidades

```mermaid
flowchart LR
    UI[Interfaz Angular] -->|intención del usuario| API[API Gateway]
    API -->|identidad y autorización| Domain[Microservicio de dominio]
    Domain -->|estado propio| DB[(PostgreSQL)]
    Domain -->|solo cuando corresponde| External[Proveedor externo]
    External -->|estado oficial| Domain
    Domain -->|resultado verificado| UI
```

La interfaz nunca determina por sí sola que un pago fue exitoso. `pago-ms` consulta a la pasarela y valida orden, monto y moneda antes de confirmar la venta.

## Desarrollo y producción

| Capa | Desarrollo con `docker-compose.yml` | Producción con override |
| --- | --- | --- |
| Entrada | Frontend `:4200`, Gateway `:8090` | Caddy `:80/:443` |
| TLS | No incluido | Certificado automático de Caddy |
| Servicios internos | Red `medizano-net` | Red `medizano-net` |
| Observabilidad | Puertos locales para Prometheus y Grafana | Perfil opcional `observability` |
| Persistencia | Volumen Docker de PostgreSQL | Volúmenes persistentes y respaldo programado |

El módulo `config-server` forma parte del reactor Maven, pero el despliegue actual obtiene su configuración de los archivos `application.yml` y de variables de entorno; no se levanta como servicio en Docker Compose.
