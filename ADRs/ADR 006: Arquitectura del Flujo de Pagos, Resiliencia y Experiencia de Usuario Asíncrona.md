# ADR 006: Gestión Integral del Ciclo de Vida de Órdenes, Pagos y Resiliencia Sistémica

**Fecha:** 6 de Abril de 2026  
**Estado:** Aprobado  
**Contexto:** El dominio de Órdenes y Pagos es el núcleo crítico de "Bazaar". Requiere una arquitectura que garantice la consistencia de datos, la inmutabilidad financiera, la gestión eficiente del stock y la recuperación automática ante fallos de red o de infraestructura interna.

## 1. El Problema: Consistencia y Resiliencia en Sistemas Distribuidos

Un flujo de compras mal diseñado puede derivar en:

1. **Pérdida de Integridad Histórica:** Cambios en el catálogo que alteran facturas pasadas.
2. **Stock Secuestrado:** Usuarios que inician un pago pero no lo completan, bloqueando el producto para otros compradores.
3. **Mensajes Fantasmas:** Pagos confirmados que no impactan en el sistema porque el Broker de Mensajería (RabbitMQ) estaba caído.
4. **Doble Reembolso:** Errores de red que provocan que se devuelva el dinero dos veces ante una cancelación.

## 2. Decisión: Inmutabilidad mediante Snapshots

Para blindar el historial de "Mis Compras" y "Mis Ventas":

- El `Order Service` no solo guardará el `product_id`. Al momento de crear la orden, realizará un **Snapshot** (captura) de los datos del producto (Nombre, Precio de venta, Imagen).
- Estos datos se guardan de forma redundante en la base de datos PostgreSQL de Órdenes.
- **Resultado:** Si un vendedor cambia el precio o borra el producto al día siguiente, la orden del comprador permanece intacta y veraz.

## 3. Decisión: Gestión de Reservas Huérfanas (TTL de Stock)

Para evitar que el stock quede bloqueado por pagos abandonados:

- Toda orden en estado `PENDING_PAYMENT` tendrá un **TTL (Time-To-Live) de 15 minutos**.
- El **CronJob** de reconciliación cancelará automáticamente cualquier orden que supere este tiempo sin una confirmación de pago del Gateway.
- Al cancelar, se emitirá un evento para **liberar el stock** en el `Inventory Service`, permitiendo que otros usuarios compren el producto.

## 4. Decisión: Resiliencia ante Fallos Internos (Transactional Outbox)

Para garantizar que nunca se pierda una notificación de pago exitoso aunque RabbitMQ falle:

- Se implementará el patrón **Transactional Outbox** en el `Payment Service`.
- En una misma transacción de base de datos, se guardará el estado del pago y el mensaje del evento en una tabla de "Salida" (`outbox_events`).
- Un proceso independiente (Relay/Worker) leerá esta tabla y reintentará el envío a RabbitMQ hasta que se confirme la recepción. Esto garantiza **entrega al menos una vez (At-least-once delivery)**.

## 5. Decisión: Idempotencia en Cancelaciones y Reembolsos

- Toda solicitud de reembolso (Refund) enviada desde el `Order Service` al `Payment Service` (y de allí al Gateway) incluirá el `order_id` como **Clave de Idempotencia**.
- Si el sistema intenta reembolsar dos veces por un error de reintento, el Gateway detectará la clave duplicada y rechazará la segunda operación, evitando pérdidas financieras.

## 6. Flujo de Experiencia de Usuario (UX)

- **Polling en Frontend:** El cliente (React/Mobile) consultará el estado de la orden cada 2-3 segundos tras el pago para dar feedback inmediato al usuario.
- **Notificaciones Push (Firebase FCM):** En caso de que el usuario cierre la aplicación, el `Notification Service` enviará una alerta push al detectar el cambio de estado a `PAID` o `CANCELLED` en la cola de mensajes.

## 7. Máquina de Estados de la Orden

1. `PENDING_PAYMENT`: Stock reservado temporalmente.
2. `PAID`: Pago confirmado y stock deducido definitivamente.
3. `SHIPPED`: Vendedor despachó el producto.
4. `DELIVERED`: Paquete recibido (habilita **Reseñas**).
5. `CANCELLED`: Pago fallido, expiración de tiempo (TTL) o cancelación manual. Gatilla reembolso si correspondiera.

## 8. Consecuencias

- **Positivas:** El sistema es financieramente auditable, resistente a particiones de red y garantiza una alta disponibilidad del stock real.
- **Negativas:** Mayor complejidad en el desarrollo (implementación de Outbox y Workers) y ligero incremento en el uso de almacenamiento por los Snapshots de productos.
