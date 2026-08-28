# Brief Técnico del Proyecto Sello

Este documento es el punto donde el equipo declara, por escrito, qué sistema distribuido desarrollará durante el ciclo.

MediZano se adapta al dominio de comercio electrónico, manteniendo el flujo de negocio principal **catálogo → orden → pago** e integrando un servicio externo real.

---

## 1. Datos del equipo

- **Nombre del equipo:** MediZano
- **Sección:** —
- **Repositorio (URL):** https://github.com/rbn-69-cod/MediZano.git
- **Topics del repositorio configurados (sí/no):** No — pendiente configurar `grupo-<numero>-medizano`

Integrantes:

| Integrante | Rol o énfasis previsto |
| --- | --- |
| Aldo Calla Ticona | Backend y frontend |
| Igarlos Ruben Mamani Quispe | Seguridad e integración de pagos |
| Kengui P. Calsin Mamani | Inventario y gestión de clientes |

---

## 2. Dominio del proyecto

- **Nombre del proyecto:** MediZano

- **Problema o necesidad que resuelve (2-4 líneas):**  
  MediZano busca facilitar la venta en línea de productos farmacéuticos y de botica, permitiendo que los clientes consulten productos disponibles y realicen pedidos desde una plataforma web. Además, busca integrar las ventas con el control de inventario para mantener actualizado el stock de los productos.

- **Dominio de negocio:**  
  Comercio electrónico orientado a productos farmacéuticos y de botica.

  **Flujo de extremo a extremo:**  
  catálogo → orden → pago → confirmación → actualización de inventario.

- **Usuarios / actores principales:**
  - `CLIENTE`
  - `ADMIN`
  - `ALMACEN`

- **Servicio externo real que integra el proyecto:**  
  **Mercado Pago**, utilizado como pasarela externa para procesar los pagos de las órdenes realizadas mediante MediZano.

- **¿Continúa un proyecto de un ciclo anterior, o es un dominio nuevo?**  
  Sí. MediZano continúa un proyecto desarrollado anteriormente en el curso de **Lenguaje de Programación II**.

---

## 3. Microservicios previstos y alcance esperado

Cada integrante tendrá dos microservicios. De los dos, al menos uno será transaccional.

| Integrante | Microservicio transaccional | Microservicio no transaccional |
| --- | --- | --- |
| Aldo Calla Ticona | `orden-ms` | `catalogo-ms` |
| Igarlos Ruben Mamani Quispe | `pago-ms` | `auth-ms` |
| Kengui P. Calsin Mamani | `inventario-ms` | `cliente-ms` |

---

### Microservicio: `orden-ms` (integrante: Aldo Calla Ticona · tipo: transaccional)

- **Descripción breve (2-3 líneas):**  
  Gestiona las órdenes de compra realizadas por los clientes. Registra la cabecera de la orden y los productos incluidos en cada pedido, calcula el total y mantiene el estado de la compra.

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
  Sí. Se comunicará de forma síncrona mediante REST/Feign con `catalogo-ms` para consultar la información de los productos y con `pago-ms` para gestionar el proceso de pago.

- **¿Qué rutas quedan protegidas y con qué rol(es)?**
  - Crear una orden: `CLIENTE`
  - Consultar órdenes propias: `CLIENTE`
  - Consultar todas las órdenes: `ADMIN`

- **Lista inicial de requisitos:**
  1. El sistema debe permitir crear una orden con uno o más productos.
  2. El sistema debe calcular el total de la orden a partir de sus detalles.
  3. El sistema debe permitir consultar el estado de una orden.

---

### Microservicio: `catalogo-ms` (integrante: Aldo Calla Ticona · tipo: no transaccional)

- **Descripción breve (2-3 líneas):**  
  Gestiona el catálogo de productos disponibles en MediZano. Proporciona la información necesaria para que los clientes puedan consultar los productos ofrecidos en la plataforma.

- **Entidad principal:**  
  `Producto` / `Categoria`

- **Datos iniciales previstos:**
  - `Producto`: id, nombre, precio.
  - `Categoria`: id, nombre, descripcion.

- **Endpoints iniciales previstos:**
  - `GET /api/v1/productos`
  - `GET /api/v1/productos/{id}`
  - `POST /api/v1/productos`

- **¿Se comunica con otro microservicio del equipo? ¿Cómo?**  
  Sí. `orden-ms` consultará `catalogo-ms` mediante REST/Feign para obtener la información de los productos incluidos en una orden.

- **¿Qué rutas quedan protegidas y con qué rol(es)?**
  - Consultar catálogo: público.
  - Consultar producto: público.
  - Registrar y modificar productos: `ADMIN`.

- **Lista inicial de requisitos:**
  1. El sistema debe permitir consultar los productos disponibles.
  2. El sistema debe permitir organizar los productos por categorías.
  3. El sistema debe permitir al administrador registrar y modificar productos.

---

### Microservicio: `pago-ms` (integrante: Igarlos Ruben Mamani Quispe · tipo: transaccional)

- **Descripción breve (2-3 líneas):**  
  Gestiona los pagos correspondientes a las órdenes de MediZano e integra el sistema con la API externa de Mercado Pago. Registra el resultado de cada operación y permite conocer el estado del pago.

- **Entidad principal o cabecera-detalle:**  
  `Pago` / `DetallePago`

- **Datos iniciales previstos:**
  - `Pago`: id, ordenId, monto, estado.
  - `DetallePago`: id, pagoId, referenciaExterna.

- **Endpoints iniciales previstos:**
  - `POST /api/v1/pagos`
  - `GET /api/v1/pagos/{id}`
  - `GET /api/v1/pagos/orden/{ordenId}`

- **¿Se comunica con otro microservicio del equipo? ¿Cómo?**  
  Sí. Se comunicará con `orden-ms` mediante REST/Feign y consumirá la API externa real de Mercado Pago.

- **¿Qué rutas quedan protegidas y con qué rol(es)?**
  - Generar pago: `CLIENTE`
  - Consultar pago propio: `CLIENTE`
  - Consultar todos los pagos: `ADMIN`

- **Lista inicial de requisitos:**
  1. El sistema debe permitir generar un pago asociado a una orden.
  2. El sistema debe integrar el proceso de pago con Mercado Pago.
  3. El sistema debe registrar el estado del pago como pendiente, aprobado o rechazado.

---

### Microservicio: `auth-ms` (integrante: Igarlos Ruben Mamani Quispe · tipo: no transaccional)

- **Descripción breve (2-3 líneas):**  
  Gestiona la autenticación y autorización de los usuarios de MediZano. Permite controlar el acceso a las funcionalidades del sistema mediante JWT y roles.

- **Entidad principal:**  
  `Usuario` / `Rol`

- **Datos iniciales previstos:**
  - `Usuario`: id, username, password.
  - `Rol`: id, nombre, descripcion.

- **Endpoints iniciales previstos:**
  - `POST /api/v1/auth/login`
  - `POST /api/v1/auth/register`

- **¿Se comunica con otro microservicio del equipo? ¿Cómo?**  
  Todos los microservicios utilizarán el token JWT para validar la identidad del usuario y sus permisos.

- **¿Qué rutas quedan protegidas y con qué rol(es)?**
  - Login: público.
  - Registro de cliente: público.
  - Funciones administrativas: `ADMIN`.
  - Todos los servicios validarán JWT en sus rutas protegidas.

- **Lista inicial de requisitos:**
  1. El sistema debe permitir autenticar usuarios mediante credenciales.
  2. El sistema debe generar un token JWT después de una autenticación válida.
  3. El sistema debe restringir funcionalidades según el rol del usuario.

---

### Microservicio: `inventario-ms` (integrante: Kengui P. Calsin Mamani · tipo: transaccional)

- **Descripción breve (2-3 líneas):**  
  Gestiona las existencias de los productos de MediZano mediante movimientos de entrada y salida. Permite mantener actualizado el stock después de las operaciones realizadas en el sistema.

- **Entidad principal o cabecera-detalle:**  
  `MovimientoInventario` / `DetalleMovimiento`

- **Datos iniciales previstos:**
  - `MovimientoInventario`: id, fecha, tipoMovimiento.
  - `DetalleMovimiento`: productoId, cantidad, lote.

- **Endpoints iniciales previstos:**
  - `POST /api/v1/inventario/movimientos`
  - `GET /api/v1/inventario/stock/{productoId}`
  - `GET /api/v1/inventario/movimientos`

- **¿Se comunica con otro microservicio del equipo? ¿Cómo?**  
  Sí. Se comunicará con `orden-ms` y `catalogo-ms`. Inicialmente la comunicación será mediante REST/Feign.

- **¿Qué rutas quedan protegidas y con qué rol(es)?**
  - Consultar disponibilidad: `CLIENTE`, `ADMIN`, `ALMACEN`
  - Registrar movimientos: `ALMACEN`, `ADMIN`
  - Consultar movimientos: `ALMACEN`, `ADMIN`

- **Lista inicial de requisitos:**
  1. El sistema debe registrar las entradas y salidas de productos.
  2. El sistema debe impedir una salida cuando la cantidad solicitada sea superior al stock disponible.
  3. El sistema debe permitir consultar el stock disponible de cada producto.

---

### Microservicio: `cliente-ms` (integrante: Kengui P. Calsin Mamani · tipo: no transaccional)

- **Descripción breve (2-3 líneas):**  
  Gestiona la información de los clientes registrados en MediZano. Mantiene los datos necesarios para identificar al comprador y relacionarlo con sus órdenes.

- **Entidad principal:**  
  `Cliente`

- **Datos iniciales previstos:**
  - `Cliente`: id, nombres, correo.

- **Endpoints iniciales previstos:**
  - `POST /api/v1/clientes`
  - `GET /api/v1/clientes/{id}`
  - `PUT /api/v1/clientes/{id}`

- **¿Se comunica con otro microservicio del equipo? ¿Cómo?**  
  Sí. `orden-ms` utilizará la identificación del cliente para asociar una orden con el comprador. La comunicación será mediante REST/Feign.

- **¿Qué rutas quedan protegidas y con qué rol(es)?**
  - Consultar perfil propio: `CLIENTE`
  - Modificar perfil propio: `CLIENTE`
  - Consultar clientes: `ADMIN`

- **Lista inicial de requisitos:**
  1. El sistema debe permitir registrar la información de un cliente.
  2. El sistema debe permitir al cliente consultar sus datos.
  3. El sistema debe permitir al cliente actualizar sus datos personales.

---

- **Qué SÍ cubre este proyecto en conjunto:**  
  MediZano cubrirá la consulta de un catálogo de productos, gestión de clientes, generación de órdenes de compra, procesamiento de pagos mediante Mercado Pago, autenticación y autorización mediante JWT y control del inventario asociado a las compras.

- **Qué NO cubre — fuera de alcance, explícito:**
  - Gestión de entregas mediante una flota propia.
  - Seguimiento GPS de repartidores.
  - Integración con proveedores farmacéuticos externos.
  - Venta internacional y conversión de monedas.
  - Aplicación móvil nativa.

---

## 4. Aprobación

- **Docente:**
- **Fecha:**