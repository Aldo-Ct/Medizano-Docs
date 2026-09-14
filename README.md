# Medizano-Docs

Documentación técnica y operativa de [MediZano Botica](https://github.com/rbn-69-cod/MediZano_microservicios), publicada con MkDocs Material.

## Accesos en línea

Los siguientes enlaces fueron verificados desde Internet el **11 de septiembre de 2026**:

| Recurso | Enlace | Estado |
| --- | --- | --- |
| Aplicación web | [https://medizano.sbs/](https://medizano.sbs/) | Público mediante HTTPS |
| Swagger UI | [https://medizano.sbs/swagger-ui.html](https://medizano.sbs/swagger-ui.html) | Público mediante HTTPS |
| Salud del API Gateway | [https://medizano.sbs/actuator/health](https://medizano.sbs/actuator/health) | Público; debe responder `{"status":"UP"}` |
| Documentación técnica | [https://aldo-ct.github.io/Medizano-Docs/](https://aldo-ct.github.io/Medizano-Docs/) | Público mediante GitHub Pages |

### Observabilidad

Grafana, Prometheus y Loki están configurados en el perfil Docker `observability`, pero **no están publicados actualmente en Internet**. Esta separación evita exponer métricas, registros y datos operativos sin autenticación adicional.

| Servicio | Acceso actual | Uso |
| --- | --- | --- |
| Grafana | `http://localhost:3000` desde el VPS o mediante un túnel SSH | Dashboards, métricas y consulta de logs |
| Prometheus | `http://localhost:9090` desde el VPS o la red autorizada | Métricas y consultas PromQL |
| Loki | `http://loki:3100` dentro de la red Docker | Fuente de logs; se consulta desde **Grafana → Explore** |

> No se deben usar `https://medizano.sbs/grafana/`, `/prometheus/` o `/loki/` como enlaces de observabilidad: esas rutas no están configuradas en el proxy y actualmente cargan el frontend. Para habilitar acceso remoto se recomienda publicar solamente Grafana, protegido con HTTPS y autenticación; Prometheus y Loki deben permanecer en la red privada.

La secuencia completa de inicio, verificación, diagnóstico y recuperación está en la [guía de levantamiento](https://aldo-ct.github.io/Medizano-Docs/operacion/levantamiento/).

## Vista local

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Abra `http://127.0.0.1:8000`.

## Compilar el sitio

```bash
mkdocs build --strict
```

Los diagramas están escritos en Mermaid dentro de los archivos Markdown para que puedan mantenerse junto con la arquitectura.
