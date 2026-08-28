# Brief Técnico del Proyecto Sello

Este documento declara el sistema distribuido que el equipo MediZano desarrollará durante el ciclo.
El proyecto adapta MediZano al dominio de comercio electrónico, manteniendo el flujo principal
catálogo → orden → pago e integrando un servicio externo real.

## 1. Datos del equipo

- **Nombre del equipo:** MediZano
- **Sección:** —
- **Repositorio (URL):** https://github.com/rbn-69-cod/MediZano.git
- **Topics del repositorio configurados (sí/no):** No — pendiente configurar `grupo-<numero>-medizano`

**Integrantes:**

| Integrante | Rol o énfasis previsto |
| --- | --- |
| Aldo Calla Ticona | Backend y frontend |
| Igarlos Ruben Mamani Quispe | Seguridad e integración de pagos |
| Kengui P. Calsin Mamani | Inventario y gestión de clientes |

## 2. Dominio del proyecto

- **Nombre del proyecto:** MediZano

- **Problema o necesidad que resuelve (2-4 líneas):**  
  MediZano busca facilitar la venta en línea de productos farmacéuticos y de botica, permitiendo que los clientes consulten productos disponibles y realicen pedidos desde una plataforma web. Además, integra las ventas con el control de inventario para mantener actualizado el stock.

- **Dominio de negocio:**  
  Comercio electrónico orientado a productos farmacéuticos y de botica.

  **Flujo:** catálogo → orden → pago → confirmación → actualización de inventario.

- **Usuarios / actores principales:**  
  `CLIENTE`, `ADMIN`, `ALMACEN`.

- **Servicio externo real que integra el proyecto:**  
  Mercado Pago, utilizado como pasarela externa para procesar los pagos de las órdenes.

- **¿Continúa un proyecto de un ciclo anterior, o es un dominio nuevo?**  
  Sí. Continúa el proyecto MediZano desarrollado en el curso de Lenguaje de Programación II.

## 3. Microservicios previstos y alcance esperado

| Integrante | Microservicio transaccional | Microservicio no transaccional |
| --- | --- | --- |
| Aldo Calla Ticona | `orden-ms` | `catalogo-ms` |
| Igarlos Ruben Mamani Quispe | `pago-ms` | `auth-ms` |
| Kengui P. Calsin Mamani | `inventario-ms` | `cliente-ms` |

### Microservicio: `orden-ms` (integrante: Aldo Calla Ticona · tipo: transaccional)

- **Descripción breve:**  
  Gestiona las órdenes de compra realizadas por los clientes. Registra la cabecera y los productos incluidos en cada pedido, calcula los importes correspondientes y mantiene el estado de la orden.

- **Entidad principal o cabecera-detalle:**  
  `Orden` / `DetalleOrden`

- **Datos iniciales previstos:**
    - `Orden`: id, clienteId, total, estado.
    - `DetalleOrden`: productoId, cantidad, precioUnitario.

- **Endpoints iniciales previstos:**
    - `POST /api/v1/ordenes`
    - `GET /api/v1/ordenes/{id}`
    - `GET /api/v1/ordenes/cliente/{clienteId}`

- **¿Se comunica con otro microservicio del equipo? ¿Cómo?**  
  Sí. Se comunicará de forma síncrona mediante REST/Feign con `catalogo-ms` y `pago-ms`.

- **¿Qué rutas quedan protegidas y con qué rol(es)?**
    - Crear orden: `CLIENTE`
    - Consultar órdenes propias: `CLIENTE`
    - Consultar todas las órdenes: `ADMIN`

- **Lista inicial de requisitos:**
    1. El sistema debe permitir crear una orden con uno o más productos.
    2. El sistema debe calcular el total de la orden.
    3. El sistema debe permitir consultar el estado de una orden.