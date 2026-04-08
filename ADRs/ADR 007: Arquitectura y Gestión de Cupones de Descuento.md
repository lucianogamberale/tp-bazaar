# ADR 007: Arquitectura y Gestión de Cupones de Descuento

## Estado

Aprobado

## Contexto

El enunciado define dos capacidades relacionadas:

- Vendedor: crear y gestionar cupones propios.
- Comprador: aplicar cupon en checkout con reglas de validacion de negocio.

Reglas funcionales obligatorias relevantes:

- Codigo unico global en plataforma.
- Un solo cupon por orden.
- Cupon invalido/vencido no interrumpe el flujo de checkout.
- El descuento aplicado no puede superar el total de la orden.

La logica de cupones es de dominio promocional y evoluciona con mayor frecuencia que el flujo transaccional de orden/pago.

## Decisión

Se separa la gestion de cupones en un servicio dedicado (`Discount` o `Coupon`) con base PostgreSQL y responsabilidades acotadas:

1. Gestion de cupones de vendedor: crear, listar y desactivar cupones propios.
2. Validacion de codigo unico global mediante restriccion a nivel de datos.
3. Evaluacion de validez de cupon (activo, vigente, reglas de uso) y calculo de descuento.
4. Integracion con checkout en dos momentos:
	 - Previsualizacion: validacion para mostrar total estimado en UI.
	 - Confirmacion de checkout: recalculo server-side antes de iniciar pago.
5. Aplicacion de regla "un cupon por orden" y cap de descuento para no superar subtotal/total aplicable.
6. Deduccion/consumo de uso de cupon en confirmacion de orden pagada, con idempotencia para evitar decrementos dobles.

`Order Service` conserva el rol de orquestar checkout/pago sin absorber reglas promocionales.

## Alternativas consideradas

1. Modelar cupones dentro de `Order Service`.
2. Servicio dedicado de cupones con validacion sincronica desde checkout.
3. Consumo de uso en el momento de aplicar cupon en UI.
4. Consumo de uso al confirmar pago/orden.

## Consecuencias

- Positivas:
	- Se desacopla el dominio promocional del dominio transaccional de orden/pago.
	- Se protege la consistencia de reglas de cupon con validacion centralizada.
	- Se facilita evolucion de campañas sin tocar el core de checkout.
- Negativas o tradeoffs:
	- Se agrega una dependencia sincronica extra en el camino critico del checkout.
	- Incrementa la complejidad de integracion e idempotencia entre servicios.
- Riesgos y mitigaciones:
	- Riesgo: desalineacion entre descuento previsualizado y descuento final al pagar.
		- Mitigacion: recalculo obligatorio en backend durante confirmacion de checkout.
	- Riesgo: sobreconsumo por reintentos/callbacks duplicados.
		- Mitigacion: operacion de consumo idempotente por `order_id`.
	- Riesgo: indisponibilidad temporal de servicio de cupones.
		- Mitigacion: degradar funcionalidad de cupones manteniendo checkout sin descuento cuando la politica de negocio lo permita.

## Impacto en artefactos

- OpenAPI afectado:
	- Endpoints de vendedor para cupones (crear/listar/desactivar).
	- Endpoint de validacion interna de cupon para checkout.
	- `POST /checkout` (campo `coupon_code` y total final calculado server-side).
- Diagramas afectados:
	- Secuencia de aplicacion de cupon en checkout (previsualizacion + confirmacion).
	- Secuencia de consumo de cupon tras pago confirmado.
- Servicios y datos impactados:
	- Discount/Coupon Service (reglas, persistencia, consumo).
	- Order Service (invocacion de validacion y aplicacion final de descuento).
	- Payment/Checkout flow (monto final consistente previo a cobro).
