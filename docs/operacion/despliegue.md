# Despliegue y configuración

## Requisitos

- Docker Engine con Docker Compose v2.
- Puertos libres para el ambiente elegido.
- Un archivo `.env` local basado en `.env.example`.
- Para producción: dominio apuntando al VPS y puertos TCP `80` y `443` accesibles.

## Variables obligatorias

Nunca se deben registrar credenciales reales en Git. El `.env` debe contener valores fuertes para:

```dotenv
POSTGRES_PASSWORD=...
JWT_SECRET=...
MEDIZANO_DEFAULT_PASSWORD=...
INTERNAL_SERVICE_TOKEN=...
GRAFANA_ADMIN_PASSWORD=...
```

Pasarelas, según las que se habiliten:

```dotenv
PAYPAL_BASE_URL=https://api-m.sandbox.paypal.com
PAYPAL_CLIENT_ID=...
PAYPAL_CLIENT_SECRET=...
PAYPAL_CURRENCY=USD
PAYPAL_EXCHANGE_RATE=3.75

MERCADOPAGO_ACCESS_TOKEN=...
MERCADOPAGO_PUBLIC_KEY=...
MERCADOPAGO_WEBHOOK_SECRET=...
MERCADOPAGO_CURRENCY=PEN
```

`JWT_SECRET` y `INTERNAL_SERVICE_TOKEN` deben ser aleatorios y tener al menos 32 bytes. El frontend recibe únicamente datos públicos de las pasarelas; los secretos permanecen en `pago-ms`.

## Ejecución local

Desde la raíz de `MediZano_microservicios`:

```bash
cp .env.example .env
# Editar .env con valores locales seguros
docker compose up -d --build
docker compose ps
```

Accesos del ambiente local:

- Aplicación: `http://localhost:4200`
- API Gateway: `http://localhost:8090`
- Swagger: `http://localhost:8090/swagger-ui.html`
- Eureka: `http://localhost:8761`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`

## Producción con HTTPS

El override `docker-compose.prod.yml` elimina la publicación directa del frontend, Gateway y Eureka. Caddy queda como única entrada pública y reenvía a Nginx.

```bash
export MEDIZANO_DOMAIN=medizano.sbs
docker compose \
  -f docker-compose.yml \
  -f docker-compose.prod.yml \
  up -d --build
```

Para incluir observabilidad en producción:

```bash
docker compose \
  -f docker-compose.yml \
  -f docker-compose.prod.yml \
  --profile observability \
  up -d --build
```

## Configuración segura de pagos

Los scripts solicitan los secretos con entrada oculta, actualizan el `.env`, recrean solo `pago-ms` y esperan que quede saludable:

```bash
./deploy/configure-paypal-sandbox.sh
./deploy/configure-mercadopago.sh
./deploy/configure-mercadopago-webhook.sh
```

Para Mercado Pago, la URL de notificación debe apuntar a:

```text
https://<dominio>/api/v1/pagos/mercadopago/webhook
```

## Datos persistentes y respaldo

- PostgreSQL usa el volumen `postgres_data`.
- Caddy usa `caddy_data` y `caddy_config`.
- Prometheus, Loki y Grafana tienen volúmenes propios.
- `deploy/backup.sh` ejecuta `pg_dumpall`, comprime el resultado y conserva copias según la política configurada.
- `deploy/medizano-backup.cron` sirve como plantilla para programar el respaldo.

Antes de actualizar contenedores en producción, genere y valide un respaldo recuperable.
