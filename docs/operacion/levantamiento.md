# Guía de levantamiento

Esta guía permite iniciar MediZano desde terminal, comprobar todos los componentes y recuperar el entorno cuando un servicio no responde. Los comandos se ejecutan desde la raíz de `MediZano_microservicios`.

## 1. Requisitos

- Docker Engine o Docker Desktop con Docker Compose v2.
- Archivo `.env` creado a partir de `.env.example`.
- Contraseñas y tokens reales almacenados solo en `.env`.
- Puertos `4200`, `8090`, `8761`, `3000` y `9090` libres para el entorno local.

En macOS puede iniciar Docker Desktop desde terminal:

```bash
open -a Docker
docker info
docker compose version
```

`docker info` debe terminar correctamente antes de continuar.

## 2. Validar la configuración

```bash
test -f .env || cp .env.example .env
docker compose --profile observability config --quiet
```

Si se acaba de crear `.env`, edítelo y sustituya todos los valores de ejemplo. No copie su contenido en capturas, tickets o repositorios.

## 3. Primer levantamiento

El primer inicio construye las imágenes y habilita Grafana, Prometheus, Loki y Alloy:

```bash
docker compose --profile observability up -d --build
```

Para inicios posteriores, cuando no cambió el código, basta con:

```bash
docker compose --profile observability up -d
```

## 4. Comprobar los contenedores

```bash
docker compose --profile observability ps
```

Los componentes con healthcheck deben aparecer como `healthy`. El arranque de Spring Boot puede tardar entre 30 y 90 segundos dependiendo del equipo.

Para observar la transición en tiempo real:

```bash
watch -n 3 'docker compose --profile observability ps'
```

En macOS, si `watch` no está instalado:

```bash
while true; do clear; docker compose --profile observability ps; sleep 3; done
```

Detenga el bucle con `Ctrl+C`.

## 5. Verificación funcional

```bash
curl -fsS http://localhost:4200/ >/dev/null && echo "Frontend OK"
curl -fsS http://localhost:8090/actuator/health
curl -fsSL http://localhost:8090/swagger-ui.html >/dev/null && echo "Swagger OK"
curl -fsS http://localhost:8761/ >/dev/null && echo "Eureka OK"
curl -fsS http://localhost:3000/api/health
curl -fsS http://localhost:9090/-/ready
docker exec medizano-loki wget -qO- http://localhost:3100/ready
```

Resultados esperados:

- Gateway: `{"status":"UP"}`.
- Grafana: base de datos `ok`.
- Prometheus: `Prometheus Server is Ready.`.
- Loki: `ready`.

## 6. Verificar los targets de Prometheus

Con `jq` instalado:

```bash
curl -fsS http://localhost:9090/api/v1/targets \
  | jq -r '.data.activeTargets[] | [.labels.job, .health, .lastError] | @tsv'
```

Todos estos jobs deben quedar `up`:

- `api-gateway`
- `catalogo-ms`
- `cliente-ms`
- `facturacion-ms`
- `inventario-ms`
- `orden-ms`
- `pago-ms`
- `usuario-ms`
- `prometheus`

Para mostrar solo fallos:

```bash
curl -fsS http://localhost:9090/api/v1/targets \
  | jq -r '.data.activeTargets[] | select(.health != "up") | [.labels.job, .lastError] | @tsv'
```

Una salida vacía significa que no hay targets caídos.

## 7. Accesos

### Desde el mismo equipo

| Recurso | URL |
| --- | --- |
| Aplicación | [http://localhost:4200](http://localhost:4200) |
| API Gateway | [http://localhost:8090](http://localhost:8090) |
| Swagger | [http://localhost:8090/swagger-ui.html](http://localhost:8090/swagger-ui.html) |
| Eureka | [http://localhost:8761](http://localhost:8761) |
| Grafana | [http://localhost:3000](http://localhost:3000) |
| Prometheus | [http://localhost:9090](http://localhost:9090) |

Loki no ofrece un panel para usuarios. Sus logs se consultan en **Grafana → Explore** usando el datasource Loki.

### Publicados en Internet

| Recurso | URL |
| --- | --- |
| Aplicación | [https://medizano.sbs/](https://medizano.sbs/) |
| Swagger | [https://medizano.sbs/swagger-ui.html](https://medizano.sbs/swagger-ui.html) |
| Estado del Gateway | [https://medizano.sbs/actuator/health](https://medizano.sbs/actuator/health) |
| Documentación | [https://aldo-ct.github.io/Medizano-Docs/](https://aldo-ct.github.io/Medizano-Docs/) |

Grafana, Prometheus y Loki no están publicados directamente. Para administrarlos a distancia sin abrir sus puertos, utilice un túnel SSH hacia el VPS:

```bash
ssh \
  -L 3000:127.0.0.1:3000 \
  -L 9090:127.0.0.1:9090 \
  usuario@servidor
```

Mientras la sesión permanezca abierta, Grafana estará disponible en `http://localhost:3000` y Prometheus en `http://localhost:9090`.

!!! warning "Seguridad"
    No publique Prometheus ni Loki sin una capa de autenticación y control de acceso. Si se necesita un panel accesible desde cualquier lugar, exponga únicamente Grafana mediante HTTPS y autenticación fuerte.

## 8. Logs y diagnóstico

Logs generales recientes:

```bash
docker compose --profile observability logs --since 10m
```

Servicios principales:

```bash
docker compose logs --since 10m api-gateway usuario-ms pago-ms orden-ms
```

Observabilidad:

```bash
docker compose --profile observability logs --since 10m prometheus loki grafana-alloy grafana
```

Reiniciar solamente un componente:

```bash
docker compose restart nombre-del-servicio
```

Reconstruir solamente un microservicio modificado:

```bash
docker compose build nombre-del-servicio
docker compose up -d --no-deps nombre-del-servicio
```

## 9. Producción

En el VPS, con DNS y `.env` ya configurados:

```bash
docker compose \
  -f docker-compose.yml \
  -f docker-compose.prod.yml \
  --profile observability \
  up -d --build
```

Después:

```bash
docker compose \
  -f docker-compose.yml \
  -f docker-compose.prod.yml \
  --profile observability \
  ps

./deploy/verify-production.sh
```

## 10. Detener el entorno

Detener conservando datos:

```bash
docker compose --profile observability down
```

No agregue `-v` salvo que quiera eliminar deliberadamente las bases, métricas, logs y configuración persistente.
