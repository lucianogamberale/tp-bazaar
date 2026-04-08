    ### 🛡️ 1. IAM Service (Identity & Access Management)

    - **Responsabilidad:** Es el guardia de seguridad del sistema. Se encarga exclusivamente del inicio de sesión, registro, emisión y validación de tokens (JWT / Refresh Tokens) y de orquestar la "Baja de Usuario" (emitiendo el evento de borrado).
    - **Base de Datos:** PostgreSQL (Aislada, solo almacena credenciales, emails y hashes).

    ### 👤 2. Profile Service (Servicio de Perfiles)

    - **Responsabilidad:** Maneja la "identidad pública" de los usuarios (Compradores y Vendedores). Almacena el nombre, la biografía y la URL de la foto de perfil. Es el encargado de aplicar el "Derecho al Olvido", anonimizando los datos públicos si el IAM le avisa que una cuenta fue eliminada.
    - **Base de Datos:** PostgreSQL.

    ### 📦 3. Product & Inventory Service (Comandos - CQRS)

    - **Responsabilidad:** Es la fuente de verdad absoluta para el negocio. Aquí los vendedores crean, editan o deshabilitan productos. Es el único servicio autorizado para sumar, reservar o deducir stock de manera transaccional y síncrona.
    - **Base de Datos:** PostgreSQL (Garantías ACID).

    ### 🔍 4. Catalog Service (Consultas - CQRS)

    - **Responsabilidad:** Optimizado para la lectura extrema y búsquedas de los compradores. Consume eventos del inventario para armar vistas rápidas. Es dueño del sub-dominio de Reseñas, calculando la reputación de los productos y vendedores de forma asíncrona. Además, es el servicio que "hidrata" (provee datos reales) al Carrito, la Wishlist y las Recomendaciones.
    - **Base de Datos:** MongoDB (NoSQL Documental, ideal para búsquedas difusas sin JOINs).

    ### 🛒 5. Cart Service (Carrito de Compras)

    - **Responsabilidad:** Mantiene el estado efímero de los productos a comprar. Solo guarda referencias (`user_id`, `product_id`, `cantidad`) y delega la responsabilidad de obtener los precios reales al Catalog Service. Se vacía automáticamente al expirar su TTL de 7 días o si la compra se inicia con éxito.
    - **Base de Datos:** Redis (En memoria, rapidísima y con auto-borrado).

    ### 🧾 6. Order Service (Servicio de Órdenes)

    - **Responsabilidad:** Es el "Director de Orquesta" del Checkout. Crea la orden, hace la captura inmutable del precio (Snapshot), pide reservar stock, valida cupones, maneja la máquina de estados (`PENDING_PAYMENT`, `PAID`, `CANCELLED`) e implementa el patrón Outbox para la emisión segura de eventos.
    - **Base de Datos:** PostgreSQL.

    ### 💳 7. Payment Service (Servicio de Pagos)

    - **Responsabilidad:** Capa Anticorrupción (Adapter). Nuestro sistema interno no habla con pasarelas externas; habla con este servicio. Transforma nuestras órdenes en intenciones de pago, se comunica con el Mock Gateway externo, recibe Webhooks asíncronos y asegura la idempotencia.
    - **Base de Datos:** PostgreSQL (o Redis) para logs transaccionales y bloqueos de idempotencia.

    ### 🔔 8. Notification Service (Servicio de Notificaciones)

    - **Responsabilidad:** Escucha pasivamente a RabbitMQ (`OrderPaidEvent`, `OrderShippedEvent`), formatea el texto y se comunica con el SDK de Firebase (FCM) para enviar notificaciones Push a la app móvil.
    - **Base de Datos:** Ninguna (Stateless).

    ### 🎟️ 9. Discount Service (Servicio de Cupones) _[Nuevo - ADR 007]_

    - **Responsabilidad:** Gestiona el ciclo de vida de los cupones de descuento (creación, límites de uso, fechas de vencimiento). Recibe consultas síncronas internas del `Order Service` durante el Checkout para calcular montos a descontar, y escucha eventos asíncronos para restar los usos de los cupones aplicados.
    - **Base de Datos:** PostgreSQL (Garantías ACID para no regalar dinero por fallos de concurrencia).

    ### ❤️ 10. Wishlist Service (Lista de Deseos) _[Nuevo - ADR 008]_

    - **Responsabilidad:** Gestiona los productos marcados como "Favoritos". A diferencia del carrito, su almacenamiento es permanente en el tiempo. Guarda relaciones `user_id` y `product_id`, e "hidrata" la vista consultando los detalles al `Catalog Service`.
    - **Base de Datos:** PostgreSQL (Tabla asociativa simple de muchos a muchos).

    ### 📊 11. Analytics Service (Métricas y Reportes) _[Nuevo - ADR 009]_

    - **Responsabilidad:** Actúa como un pequeño Data Warehouse en tiempo real para el Panel de Administración. Escucha eventos de ventas, productos y usuarios, y pre-calcula totales (ej. ventas por categoría) para que el Admin los consulte sin ralentizar la base de datos de compras.
    - **Base de Datos:** MongoDB (o PostgreSQL optimizado para OLAP / lectura de agregaciones).

    ### 👣 12. Tracking & Recommendation Service (Historial) _[Nuevo - ADR 010]_

    - **Responsabilidad:** Recibe el tráfico masivo de "clics" y visualizaciones de los usuarios (Clickstream) bajo un patrón _fire-and-forget_. Almacena el historial cronológico reciente y renderiza la sección "Vistos recientemente".
    - **Base de Datos:** Redis (Usando _Sorted Sets_ para ordenar por timestamp y TTL para descartar datos muy antiguos y no saturar memoria).

    ---

    ### 🧱 Componentes de Infraestructura Clave

    - **API Gateway:** Enruta el tráfico, valida firmas JWT y bloquea accesos no autorizados por roles (RBAC) antes de que toquen la red interna.
    - **RabbitMQ:** El bus de eventos (Message Broker) que permite que los servicios hablen entre sí de forma asíncrona y tolerante a fallos.
    - **Object Storage (MinIO / Supabase / Cloudinary):** El lugar físico para las fotos (perfiles/productos), subidas directamente desde Mobile vía URLs pre-firmadas.
