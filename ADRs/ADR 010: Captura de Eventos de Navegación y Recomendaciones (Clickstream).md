# ADR 010: Captura de Eventos de Navegación y Recomendaciones (Clickstream)

## Estado

Propuesto

## Contexto

La epica Recomendaciones (optativa) exige diseñar el mecanismo de captura de navegacion y generar recomendaciones personalizadas con fallback para usuarios sin historial.

Reglas funcionales a cubrir:

- Registrar actividad de navegacion/compra como insumo de recomendacion.
- Mostrar recomendaciones personalizadas cuando exista historial.
- Mostrar alternativa (productos populares/recientes) cuando no exista historial.
- Incluir solo productos activos y con stock disponible en resultados.

La captura de visualizaciones es write-heavy y no debe afectar endpoints criticos de catalogo/checkout.

## Decisión

Se propone separar Tracking/Recommendations en un servicio dedicado con dos capacidades:

1. Ingesta de eventos de vista de producto mediante endpoint liviano y no bloqueante.
2. Construccion de listas de recomendacion por usuario usando historial reciente y reglas simples de ranking.

Lineamientos de implementacion propuestos:

- Almacenamiento de historial reciente en estructura de alta escritura (Redis) con limite por usuario.
- Endpoint de tracking que responde rapido (`202 Accepted`) y evita bloquear UX.
- Hidratacion de IDs recomendados contra Catalog para devolver datos actuales y filtrar no disponibles.
- Fallback deterministico para usuario sin historial (populares/mas vendidos recientes).

## Alternativas consideradas

1. Registrar clickstream en Catalog/Order directamente.
2. Servicio dedicado de tracking + recomendaciones con storage optimizado a escritura.
3. Persistencia completa de largo plazo desde el inicio.
4. Persistencia acotada (reciente) con posible evolucion futura a data lake.

## Consecuencias

- Positivas:
	- Reduce impacto de escrituras de navegacion sobre servicios transaccionales.
	- Permite evolucionar el motor de recomendacion sin afectar catalogo/checkout.
	- Mejora capacidad de personalizacion incremental en frontend.
- Negativas o tradeoffs:
	- Mayor complejidad operativa por nuevo servicio y pipeline de hidratacion.
	- Si storage es efimero, limita analitica historica de largo plazo.
	- Dependencia de Catalog para materializar recomendaciones consumibles en UI.
- Riesgos y mitigaciones:
	- Riesgo: perdida de eventos de tracking por caidas transitorias.
		- Mitigacion: tolerancia funcional (degradacion aceptable) y reintento best-effort en cliente/edge.
	- Riesgo: recomendaciones con productos no disponibles.
		- Mitigacion: filtro final por estado/stock al hidratar desde Catalog.
	- Riesgo: sesgo por historial corto.
		- Mitigacion: combinar señales de popularidad global como fallback.

## Impacto en artefactos

- OpenAPI afectado:
	- `POST /tracking/views`
	- `GET /recommendations/me` (o equivalente)
	- Contrato interno/batch con Catalog para hidratar IDs.
- Diagramas afectados:
	- Secuencia de captura de visualizacion no bloqueante.
	- Secuencia de generacion y entrega de recomendaciones.
- Servicios y datos impactados:
	- Tracking/Recommendation Service (ingesta, ranking, almacenamiento reciente).
	- Catalog Service (hidratacion y filtros de disponibilidad).
	- Frontend mobile/web (consumo de recomendaciones y fallback).
