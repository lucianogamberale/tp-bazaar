# ADR 009: Arquitectura de Métricas y Reportes (Analytics Service)

## Estado

Propuesto

## Contexto

La epica Metricas exige exponer indicadores para administracion sin impactar negativamente los flujos transaccionales de compra.

Requerimientos base del enunciado:

- Usuarios registrados (totales y por periodo).
- Ordenes por estado y evolucion temporal.
- Monto transaccionado por periodo.
- Ranking de productos mas vendidos.
- Periodos predefinidos (7, 30 y 90 dias).

Adicionalmente existen capacidades optativas de desglose por categoria y exportacion CSV.

## Decisión

Se propone un servicio dedicado de Analytics/Reporting, alimentado de forma asincrona por eventos de dominio, con almacenamiento orientado a consulta agregada:

1. Separar lectura analitica del store transaccional de Order y otros servicios core.
2. Consumir eventos relevantes (`order_paid`, `user_registered`, cambios de catalogo necesarios para joins/reportes).
3. Mantener proyecciones/materializaciones por periodo y dimension requerida (estado, producto, categoria).
4. Exponer endpoints de consulta para panel admin con filtros de periodo y paginacion cuando aplique.
5. Tratar metricas por categoria y exportacion como extensiones optativas sin bloquear alcance obligatorio.

## Alternativas consideradas

1. Consultas analiticas directas sobre base transaccional de Order.
2. Servicio dedicado de analytics con proyecciones asincronas.
3. Batch nocturno para reportes diferidos.

## Consecuencias

- Positivas:
   - Aisla carga de reporting del flujo critico de checkout.
   - Mejora tiempos de respuesta del panel admin con datos preagregados.
   - Permite evolucionar nuevas metricas sin acoplarse a modelos transaccionales.
- Negativas o tradeoffs:
   - Introduce consistencia eventual entre venta real y metrica visible.
   - Duplica parte de los datos en almacenamiento analitico.
   - Incrementa complejidad operativa (consumidores, reproceso y versionado de eventos).
- Riesgos y mitigaciones:
   - Riesgo: eventos duplicados o fuera de orden.
      - Mitigacion: consumidores idempotentes y llaves de deduplicacion.
   - Riesgo: desfasaje temporal en dashboard.
      - Mitigacion: SLA explicito de frescura de datos y timestamp de ultima actualizacion.
   - Riesgo: reprocesos costosos ante cambios de modelo.
      - Mitigacion: versionado de eventos y estrategias de backfill por ventana.

## Impacto en artefactos

- OpenAPI afectado:
   - Endpoints admin de metricas por periodo.
   - Endpoints optativos de metricas por categoria y exportacion.
- Diagramas afectados:
   - Flujo de eventos hacia proyecciones analiticas.
   - Diagrama de consulta del dashboard administrativo.
- Servicios y datos impactados:
   - Analytics Service (store analitico y proyecciones).
   - Order/IAM/Catalog (fuentes de eventos para agregaciones).
   - Backoffice Admin (consumo de endpoints de metricas).
