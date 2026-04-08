# ADR 011: Invariantes de Consistencia e Idempotencia en Checkout

## Estado

Propuesto

## Contexto

El flujo de checkout involucra varios servicios (Order, Payment, Product/Inventory, Cart) y el enunciado exige:

1. Evitar doble cobro ante reintentos o fallos de red.
2. Evitar inconsistencias entre pago, stock y estado de orden.
3. Resolver concurrencia del ultimo item sin cobros incorrectos.

## Decisión

Se adoptan los siguientes invariantes de consistencia para el checkout:

1. No puede existir orden confirmada sin pago aprobado.
2. No puede haber cobro exitoso sin una orden asociada e identificable.
3. No puede descontarse stock de forma definitiva hasta confirmar pago.
4. El checkout debe ser idempotente por clave de idempotencia (checkout intent).
5. Si el pago falla o expira, el carrito se conserva y el stock no se descuenta.
6. Solo una transacción puede ganar la compra del ultimo item disponible.

## Estrategia técnica (provisional)

1. Crear orden en estado `pendiente de pago` con `checkout_intent_id` unico.
2. Ejecutar pago con `Idempotency-Key = checkout_intent_id`.
3. Confirmar orden y descontar stock solo cuando el pago sea aprobado.
4. Si el pago es rechazado, marcar `pago rechazado` y conservar carrito.
5. Garantizar atomicidad de transicion de estados en Order Service.

## Alternativas consideradas

1. Flujo sin idempotencia explicita en pago.

- Pros: menor complejidad inicial.
- Contras: alto riesgo de doble cobro.

2. Descontar stock antes de pago y compensar luego.

- Pros: simplifica validacion de disponibilidad al cobrar.
- Contras: mayor complejidad de compensaciones y riesgo de inconsistencia.

3. Pago primero y orden despues.

- Pros: reduce estados intermedios en orden.
- Contras: puede existir cobro sin orden si falla persistencia.

## Consecuencias

1. Requiere diseño explícito de idempotencia entre cliente, Order y Payment.
2. Requiere definir manejo de timeouts y reintentos.
3. Reduce riesgos de doble cobro y conflictos de stock.
4. Obliga a documentar maquina de estados de orden y transiciones validas.

## Impacto en artefactos

- Trazabilidad: Checkout y Ordenes (CA1, CA2, CA4, CA5).
- Contratos: endpoint de checkout con soporte de idempotency key.
- Diagramas: secuencia de checkout con ramas de aprobado/rechazado/timeout.
- Decision log: cierre de invariantes de consistencia en Iteracion 3.
