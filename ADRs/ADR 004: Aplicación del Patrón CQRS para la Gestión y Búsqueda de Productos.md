# ADR 004: Aplicación del Patrón CQRS para la Gestión y Búsqueda de Productos

## Estado

Aprobado

## Contexto

El enunciado define una asimetría clara entre:

1. Operaciones de escritura sobre productos/stock (vendedor).
2. Operaciones de lectura masiva para catálogo (comprador).

Además, se requiere arquitectura de microservicios con responsabilidades delimitadas y uso de al menos una tecnología SQL y una NoSQL.

Problema a resolver:

1. Evitar que lecturas masivas degraden operaciones transaccionales de inventario.
2. Mantener búsquedas y filtros rápidos en catálogo.
3. Proteger la consistencia en checkout ante eventual consistency de vistas de lectura.

## Decisión

Se adopta segregación **CQRS** para el dominio de productos:

1. **Product & Inventory Service (write model)** con base SQL para comandos críticos.
2. **Catalog Service (read model)** con base documental para consultas de alta frecuencia.
3. Sincronización asíncrona por eventos de ciclo de vida de producto/stock.

Responsabilidades resultantes:

1. Product & Inventory: crear/editar/deshabilitar producto y gestionar stock como fuente de verdad.
2. Catalog: home, listado, búsqueda, filtros y detalle orientados a lectura.
3. Order/Checkout: validación final de disponibilidad contra write model antes de confirmar compra.

## Alternativas consideradas

1. **Modelo único (comandos y consultas en el mismo servicio/BD)**.

- Pros: menor complejidad inicial.
- Contras: contención de recursos y escalado acoplado.

2. **Separación lógica sin proyección asíncrona (lectura directa al write model)**.

- Pros: consistencia inmediata.
- Contras: consultas más costosas para búsqueda/filtros y menor elasticidad en lectura.

3. **CQRS con proyección asíncrona (decisión elegida)**.

- Pros: mejor performance de lectura y desacople de escalado.
- Contras: consistencia eventual y mayor complejidad operativa.

## Consecuencias

- Positivas:
  - Escalado independiente de lectura y escritura.
  - Mejor rendimiento de catálogo en home/búsqueda.
  - Protección del write model para operaciones críticas de stock.

- Negativas o tradeoffs:
  - Ventana de inconsistencia temporal entre write y read models.
  - Mayor necesidad de observabilidad y reintentos en proyección.

- Riesgos y mitigaciones:
  - Riesgo: producto desactualizado en catálogo durante propagación.
  - Mitigación: checkout valida contra Product&Inventory antes de cobrar.
  - Riesgo: eventos duplicados/fuera de orden.
  - Mitigación: consumidores idempotentes + upsert en read model.

## Impacto en artefactos

- OpenAPI afectado:
  - `docs/api/contracts/catalog.openapi.yaml` (consultas de catálogo).
  - Contratos de Product&Inventory para escritura de productos/stock.

- Diagramas afectados:
  - Secuencia de publicación y sincronización Product -> Catalog.
  - C4 de contenedores con separación de write/read models.

- Servicios y datos impactados:
  - Product&Inventory (SQL transaccional).
  - Catalog (NoSQL documental para lectura).
  - Broker/eventos para proyecciones de catálogo.
