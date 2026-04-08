# Plan de Trabajo con Agentes

## Objetivo

Guiar la colaboración entre equipo y agentes de IA para descubrir dominio, cerrar decisiones de arquitectura y producir entregables defendibles ante cátedra.

## Objetivos de calidad

1. Trazabilidad completa desde épicas y CA a capacidades y servicios.
2. Coherencia entre ADRs, OpenAPI y diagramas.
3. Evidencia de iteración y revisión de decisiones.

## Flujo recomendado

### Fase A - Descubrimiento de dominio (Plan + Explore)

1. Relevar épicas y CA del enunciado.
2. Identificar capacidades (comandos, consultas, eventos).
3. Detectar ambigüedades y riesgos.

Salida:

- Actualización de `traceability-matrix.md`.
- Lista priorizada en `decision-log.md`.

### Fase B - Diseño por dominio (Plan)

1. Definir límites de servicio por dominio.
2. Proponer contratos mínimos y eventos.
3. Evaluar alternativas y tradeoffs.

Salida:

- Documento de dominio en `docs/architecture/domain/`.
- ADR nuevo o actualización de ADR existente.

### Fase C - Formalización de entregables (Ejecución)

1. Crear o ajustar OpenAPI en `docs/api/contracts/`.
2. Actualizar diagramas C4 y de secuencia.
3. Registrar estado y deuda de diseño.

Salida:

- Paquete de entrega revisable por equipo y profesor.

## Buenas practicas para prompts

1. Pedir siempre formato de salida estructurado.
2. Exigir supuestos explícitos.
3. Separar hechos del enunciado de hipótesis.
4. Solicitar riesgos y decisiones pendientes.

## Definición de Done por iteración

1. Hay al menos un dominio con trazabilidad completa.
2. Hay decisiones cerradas y abiertas claramente registradas.
3. OpenAPI y diagramas del dominio están alineados.
4. El equipo puede explicar y defender las decisiones.
