# ADR 005: Estrategia de Almacenamiento y Validación del Carrito de Compras

## Estado

Aprobado

## Contexto

El dominio de carrito maneja estado efimero de alta frecuencia de cambios y abandono. Debe cumplir CAs obligatorios de la epica Carrito sin romper consistencia en Checkout:

- Persistir carrito por usuario entre sesiones y dispositivos autenticados.
- Evitar precios stale y disponibilidad desactualizada al mostrar o confirmar compra.
- Bloquear checkout cuando haya items invalidos (sin stock o deshabilitados).
- Mantener ownership claro: Cart valida precondiciones de carrito; la validacion final de concurrencia/stock para compra pertenece al flujo de Checkout/Orden.

Ademas, existe una decision de infraestructura pendiente sobre el engine de almacenamiento del carrito (Redis vs PostgreSQL), pero no bloquea las reglas de negocio ya definidas.

## Decisión

Se adopta la siguiente estrategia funcional para Carrito:

1. Solo usuarios autenticados pueden operar carrito. No se implementa carrito anonimo local/cookie.
2. El carrito persiste referencias (`user_id`, `product_id`, `quantity`) y no persiste precio ni estado del producto.
3. En lectura de carrito, el servicio hidrata contra Catalog para exponer precio y disponibilidad vigentes.
4. Al agregar o modificar cantidades, Cart valida contra disponibilidad actual y rechaza superar stock.
5. Cuando un item queda no disponible, se marca en la respuesta y se bloquea checkout hasta que el usuario lo resuelva.
6. La validacion transaccional final de concurrencia en compra no se resuelve en Cart; se delega a Checkout/Order.

## Alternativas consideradas

1. Persistir snapshot de precio/estado en el carrito.
2. Persistir solo referencias y recalcular/hidratar en lectura.
3. Infraestructura Redis para estado efimero con TTL.
4. Infraestructura PostgreSQL con expiracion gestionada por proceso de limpieza.

## Consecuencias

- Positivas:
  - Se reduce riesgo de cobrar precios desactualizados al no usar snapshots de precio en carrito.
  - Se alinea el comportamiento con CAs de bloqueo por disponibilidad y persistencia por usuario autenticado.
  - Se separan responsabilidades entre Cart y Checkout/Order, evitando logica transaccional duplicada.
- Negativas o tradeoffs:
  - La hidratacion agrega dependencia sincrona entre Cart y Catalog en lecturas.
  - Mayor sensibilidad a latencia/fallas de Catalog durante consulta de carrito.
- Riesgos y mitigaciones:
  - Riesgo: diferencia temporal entre vista de carrito y momento de pago.
    - Mitigacion: revalidacion obligatoria en Checkout/Order antes de confirmar orden.
  - Riesgo: crecimiento de carritos abandonados.
    - Mitigacion: politica de expiracion (TTL en Redis o job de limpieza en PostgreSQL).
  - Riesgo: decision tardia de infraestructura.
    - Mitigacion: mantener contrato y modelo logico agnosticos al engine para diferir eleccion sin reescribir APIs.

## Impacto en artefactos

- OpenAPI afectado:
  - `POST /cart/items`
  - `PATCH /cart/items/{itemId}`
  - `DELETE /cart/items/{itemId}`
  - `GET /cart`
- Diagramas afectados:
  - Secuencia de agregar item y gestion de carrito con validacion de stock/hidratacion.
  - Secuencia de checkout mostrando revalidacion final fuera de Cart.
- Servicios y datos impactados:
  - Cart Service (modelo de item referencial + validaciones de disponibilidad).
  - Catalog Service (fuente de verdad para precio/estado/stock en lectura de carrito).
  - Order/Checkout (resolucion de concurrencia y confirmacion final de compra).
