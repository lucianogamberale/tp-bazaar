# ADR 008: Arquitectura y Gestión de Listas de Deseos (Wishlists)

## Estado

Propuesto

## Contexto

La epica Wishlist es optativa, pero requiere cubrir reglas claras:

- Agregar y quitar productos para usuario autenticado.
- Mostrar indicador en catalogo de si el producto ya esta en wishlist.
- Mantener items sin stock o deshabilitados en la lista, marcados como no disponibles.
- Mostrar estado vacio cuando no haya favoritos.

Wishlist tiene semantica distinta a Carrito: no es transaccional de compra inmediata y no debe expirar agresivamente por TTL corto.

## Decisión

Se propone un servicio dedicado de Wishlist con almacenamiento relacional y modelo referencial:

1. Persistir relacion `user_id` + `product_id` (clave unica compuesta) sin duplicados.
2. Exponer operaciones de agregar/quitar/listar wishlist del usuario autenticado.
3. No persistir snapshot de nombre/precio/stock; hidratar contra Catalog al listar.
4. Mantener productos no disponibles en wishlist, devolviendo flag de disponibilidad para UI.
5. Requerir autenticacion para mutaciones y para lectura de wishlist propia.

## Alternativas consideradas

1. Guardar wishlist dentro de Profile Service.
2. Guardar wishlist en Cart Service reutilizando Redis.
3. Servicio dedicado Wishlist con persistencia propia.

## Consecuencias

- Positivas:
	- Desacopla favoritos del carrito y evita mezclar datos efimeros con datos persistentes de interes.
	- Facilita cumplir CAs de visualizacion (estado de wishlist en catalogo y lista de favoritos).
	- Mantiene ownership claro de dominio sin contaminar Profile o Catalog.
- Negativas o tradeoffs:
	- Agrega un servicio y base de datos adicionales para una epica optativa.
	- La hidratacion agrega dependencia sincronica con Catalog para render de lista.
- Riesgos y mitigaciones:
	- Riesgo: inconsistencia temporal de datos de producto al listar favoritos.
		- Mitigacion: hidratar en lectura y priorizar Catalog como fuente de verdad.
	- Riesgo: degradacion por llamadas masivas de hidratacion.
		- Mitigacion: endpoint batch interno en Catalog + paginacion en wishlist.

## Impacto en artefactos

- OpenAPI afectado:
	- `POST /wishlist/items`
	- `DELETE /wishlist/items/{productId}`
	- `GET /wishlist`
	- Endpoint para estado de wishlist en vistas de catalogo (segun diseño final).
- Diagramas afectados:
	- Secuencia agregar/quitar favorito.
	- Secuencia listar wishlist con hidratacion desde Catalog.
- Servicios y datos impactados:
	- Wishlist Service (tabla asociativa usuario-producto).
	- Catalog Service (hidratacion y disponibilidad).
	- Gateway/Auth (enforcement de autenticacion para acciones protegidas).
