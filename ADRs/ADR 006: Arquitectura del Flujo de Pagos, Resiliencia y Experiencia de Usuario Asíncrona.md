# ADR 006: Arquitectura del Flujo de Pagos, Resiliencia y Experiencia de Usuario Asíncrona

## Estado

Aprobado

## Contexto

Checkout y Ordenes concentran requisitos criticos de consistencia: evitar doble cobro, evitar sobreventa, mantener historial de compras/ventas inmutable y soportar asincronia de pago sin degradar experiencia de usuario.

En este dominio conviven:

- Operaciones sincronas de inicio de checkout.
- Confirmaciones asincronas de pago desde gateway.
- Actualizaciones de estado de orden por vendedor/comprador.
- Casos de error y reintento (timeouts, callbacks duplicados, fallas de broker).

Ademas, la maquina de estados debe alinearse al modelo canónico definido para el checkpoint (`pendiente_de_pago`, `confirmada`, `en_preparacion`, `enviada`, `entregada` y estados de excepcion).

## Decisión

Se adoptan las siguientes decisiones de arquitectura para Checkout/Order/Payment:

1. Snapshot de datos de producto al crear la orden (nombre, precio de venta, imagen) para preservar inmutabilidad historica.
2. Flujo asincrono de confirmacion de pago con orden inicial en `pendiente_de_pago`.
3. Idempotencia obligatoria en operaciones sensibles de pago/reembolso para evitar ejecuciones duplicadas.
4. Expiracion de ordenes en `pendiente_de_pago` por timeout operativo, con transicion controlada a estado de excepcion y compensacion de stock cuando aplique.
5. Historial de transiciones de estado con timestamp para auditabilidad.
6. UX asincrona con polling de estado de orden; notificaciones push quedan como extension no bloqueante del checkpoint.

La implementacion concreta de patrones de resiliencia (por ejemplo, outbox relay) se considera recomendada y se aplicara segun complejidad/etapa de entrega, sin alterar contratos funcionales del checkpoint.

## Alternativas consideradas

1. Orden referenciando solo `product_id` sin snapshot historico.
2. Snapshot de producto en orden para congelar evidencia transaccional.
3. Confirmacion de pago totalmente sincrona en request de checkout.
4. Confirmacion asincrona con estados intermedios e idempotencia.
5. Publicacion directa a broker sin outbox.
6. Publicacion confiable con outbox/relay.

## Consecuencias

- Positivas:
	- Se protege integridad historica de compras/ventas aunque cambie catalogo.
	- Se reduce riesgo de doble cobro con llaves de idempotencia.
	- Se soporta asincronia real de pasarela de pago sin romper flujo de negocio.
	- Se mejora auditabilidad con historial explicito de estados.
- Negativas o tradeoffs:
	- Mayor complejidad operativa por estados intermedios y compensaciones.
	- Requiere coordinacion entre Order, Payment y Product/Inventory.
	- Incremento de almacenamiento por snapshots y log de transiciones.
- Riesgos y mitigaciones:
	- Riesgo: callbacks duplicados o tardios del proveedor de pagos.
		- Mitigacion: idempotencia por `order_id`/clave de negocio y validacion de transiciones.
	- Riesgo: reservas de stock no liberadas por timeout.
		- Mitigacion: job de reconciliacion con barrido periodico de `pendiente_de_pago` expiradas.
	- Riesgo: perdida de eventos ante caida de broker.
		- Mitigacion: patron outbox/relay en etapas de endurecimiento de resiliencia.

## Impacto en artefactos

- OpenAPI afectado:
	- `POST /checkout`
	- `GET /orders`
	- `GET /orders/{id}`
	- `PATCH /seller/orders/{id}/status`
	- `PATCH /orders/{id}/confirm-delivery`
- Diagramas afectados:
	- Secuencia de checkout con confirmacion asincrona de pago.
	- Maquina de estados de orden y validacion de transiciones.
- Servicios y datos impactados:
	- Order Service (aggregate de orden, snapshots, historial de estado).
	- Payment Service (idempotencia, callbacks, resiliencia de publicacion).
	- Product/Inventory Service (descuento/liberacion de stock por transiciones de orden).
