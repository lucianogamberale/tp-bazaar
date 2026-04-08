# ADR 004: Aplicación del Patrón CQRS para la Gestión y Búsqueda de Productos

**Fecha:** 5 de Abril de 2026  
**Estado:** Propuesto / Aprobado  
**Contexto:** Dominio de Productos, Inventario y Catálogo e n la plataforma "Bazaar".

## 1. El Problema: Asimetría de Cargas y Modelos de Datos

En la plataforma Bazaar, el comportamiento sobre los productos presenta dos perfiles diametralmente opuestos:

- **Escritura (Vendedores):** Baja frecuencia. Requiere validaciones estrictas de negocio (precios positivos, stock exacto) y garantías ACID para no perder información financiera.
- **Lectura (Compradores):** Altísima frecuencia. Requiere búsquedas difusas rápidas, filtros combinados dinámicos y agregación de datos (reputación del vendedor, imágenes, categorías) en una sola vista.

Si utilizamos un único microservicio con una base de datos relacional para ambas tareas, las lecturas masivas del catálogo penalizarán el rendimiento de las transacciones críticas de inventario, y nos veremos obligados a ejecutar consultas complejas (`JOINs`) para renderizar la página de inicio, afectando la experiencia de usuario.

## 2. Decisión Arquitectónica: Segregación CQRS

Se decide implementar el patrón **CQRS (Command Query Responsibility Segregation)** apoyado en un modelo de consistencia eventual. Separaremos el dominio en dos microservicios con tecnologías de persistencia distintas:

### A. Product & Inventory Service (Comandos / Escritura)

- **Base de Datos:** PostgreSQL (Relacional).
- **Responsabilidad:** Es la "Fuente de Verdad" absoluta. Gestiona la creación, edición, deshabilitación y el stock físico de los productos.
- **Ventaja:** Garantiza la integridad transaccional mediante bloqueos de fila (`SELECT ... FOR UPDATE`) críticos para evitar sobreventas en el _Checkout_.

### B. Catalog Service (Consultas / Lectura)

- **Base de Datos:** MongoDB (NoSQL Documental).
- **Responsabilidad:** Es la "Vidriera". Resuelve las búsquedas, filtros y ordenamientos de la _Home_ y la página de detalle del producto.
- **Ventaja:** Cumple con el requerimiento técnico de la cátedra de usar NoSQL, permitiendo almacenar cada producto como un documento pre-ensamblado (desnormalizado), optimizando los tiempos de respuesta al máximo.

### C. Sincronización (El Pegamento)

Ambos servicios se comunicarán de forma asíncrona mediante un _Message Broker_ (RabbitMQ / Kafka). Cuando el `Product Service` confirma una escritura, emite un evento (ej. `ProductCreated` o `ProductDisabled`). El `Catalog Service` consume este evento y actualiza su documento en MongoDB.

## 3. Manejo de Casos Borde y Consistencia Eventual

El equipo reconoce que la comunicación asíncrona introduce una **ventana de consistencia eventual** (latencia de red entre que se actualiza PostgreSQL y se actualiza MongoDB).

**Escenario de Riesgo (El Producto Fantasma):**

1. Un vendedor "Deshabilita" un producto (o su stock llega a cero).
2. Se actualiza PostgreSQL instantáneamente.
3. El evento se demora en la cola de mensajes 2 segundos por alta concurrencia.
4. En ese lapso de 2 segundos, un comprador busca en el Catálogo (MongoDB). El producto aún figura como "Activo".
5. El comprador intenta agregarlo al carrito o comprarlo.

**Mitigación Estratégica (Validación Transaccional):**
Adoptamos la máxima arquitectónica: _"El catálogo puede mentir, pero el Checkout no puede mentir"_.
Para mitigar este caso borde, el `Order Service` (encargado del pago) **nunca confiará en los datos provenientes del Catálogo o del Carrito al momento de cobrar**. Al iniciar el Checkout, realizará una validación síncrona obligatoria contra el `Product & Inventory Service` (PostgreSQL).
Si el producto fue deshabilitado hace milisegundos, PostgreSQL bloqueará la operación, el Checkout será abortado y se notificará al usuario, cumpliendo así con los Criterios de Aceptación del negocio de manera segura y controlada.

## 4. Conclusión

Esta separación nos permite escalar el tráfico de búsquedas de manera independiente sin poner en riesgo la base de datos transaccional, justifica académicamente el uso de NoSQL (MongoDB) en el entorno ideal (búsquedas documentales) y establece fronteras claras de responsabilidad en el equipo de desarrollo.
