# Verificación y soporte

## Comprobaciones rápidas

```bash
docker compose ps
docker compose logs --tail 100 api-gateway
docker compose logs --tail 100 pago-ms
docker compose logs --tail 100 orden-ms
```

Todos los servicios principales incorporan healthchecks. Un contenedor debe aparecer como `healthy` antes de validar un flujo completo.

## Pruebas del código

Backend completo:

```bash
cd medizano-microservices
mvn test
```

Frontend:

```bash
cd frontend
npm ci
npm run lint
npm run build
```

La suite backend incluye pruebas de catálogo, facturación, inventario, órdenes, Mercado Pago, PayPal y autenticación.

## Verificación de producción

El script de comprobación inspecciona contenedores, HTTPS, cabeceras, CORS, login, rutas protegidas, configuración pública de pagos, firewall, servicios del sistema y el respaldo más reciente:

```bash
./deploy/verify-production.sh
```

Debe ejecutarse desde el servidor donde corre Docker Compose y con las variables que solicita el script.

## Lista de prueba para un pago

1. Confirmar que `pago-ms`, `orden-ms`, `inventario-ms` y `facturacion-ms` estén saludables.
2. Iniciar sesión con un rol de caja.
3. Añadir un medicamento con lote vigente y stock suficiente.
4. Generar la operación con la pasarela elegida.
5. Pagar únicamente con una cuenta compradora del ambiente de prueba.
6. Esperar la confirmación automática; no cambiar el estado manualmente.
7. Comprobar que la orden esté `PAGADA`.
8. Verificar un único descuento de stock.
9. Verificar el comprobante y su PDF.
10. Repetir la consulta de estado y confirmar que no se duplique nada.

## Problemas frecuentes

### El pago queda esperando

- Revisar bloqueadores de ventanas emergentes y protección estricta del navegador.
- Confirmar que el comprador terminó la aprobación en la pasarela.
- Consultar logs de `pago-ms` y buscar el identificador de la orden/preferencia.
- En Mercado Pago, revisar que el webhook use HTTPS y el secreto correcto.
- En Sandbox, no mezclar vendedor y comprador ni credenciales de ambientes distintos.

### El pago está aprobado pero la orden no termina

El pago se conserva como aprobado y `pago-ms` reintenta su entrega a `orden-ms`. Luego `orden-ms` reintenta stock y comprobante. Revisar:

```bash
docker compose logs --since 10m pago-ms orden-ms inventario-ms facturacion-ms
```

No se debe volver a cobrar al cliente mientras la reconciliación esté pendiente.

### El sitio muestra FortiGuard o certificado inválido

Esto pertenece a la red o al certificado, no a la contraseña de MediZano. Verificar DNS, certificado TLS y clasificación del dominio. Probar desde otra conexión ayuda a separar un bloqueo de red de un fallo del servidor.

## Observabilidad

- Prometheus recopila `/actuator/prometheus`.
- Grafana Alloy recolecta logs de Docker y los envía a Loki.
- Grafana consulta métricas y logs desde un único panel.

Consultas útiles de LogQL:

```logql
{service="pago-ms"} |= "ERROR"
{service="orden-ms"} |= "pendiente"
{service="inventario-ms"} |= "Venta"
```

Nunca copie tokens, secretos, contraseñas ni respuestas completas de autenticación dentro de tickets o capturas públicas.
