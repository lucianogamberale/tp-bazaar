# AGENTS Workflow Guide

Este archivo define buenas practicas para trabajar con agentes de IA en el proyecto.

## Principios

1. Fuente canónica: `enunciado.md`.
2. Hipótesis de diseño: `microservicios.md` y ADRs existentes.
3. Toda afirmación arquitectónica debe trazar a criterio de aceptación, requisito no funcional o ADR aprobado.
4. El equipo es responsable final de todas las decisiones.

## Modo de trabajo recomendado

### 1) Plan

Usar para:

- Descubrimiento de dominio.
- Mapeo de épicas a capacidades.
- Detección de ambigüedades y riesgos.
- Definición de prioridades por checkpoint.

Salidas esperadas:

- Matriz de trazabilidad.
- Mapa de límites de servicios.
- Registro de decisiones abiertas.

### 2) Explore (subagente)

Usar para:

- Relevamiento rápido de fuentes en el repo.
- Búsquedas de consistencia entre documentos.
- Preparar insumos de análisis sin editar archivos.

Buenas practicas:

- Pedir resultados estructurados por secciones.
- Exigir referencias a archivo y línea cuando sea posible.

### 3) Ejecución

Usar para:

- Crear o actualizar documentos versionables.
- Construir contratos OpenAPI y diagramas.
- Alinear estructura documental con entregables.

## Cadencia sugerida por iteración

1. Discovery: relevar requisitos de una épica.
2. Síntesis: proponer capacidades, owners y contratos.
3. Cierre: registrar decisiones y pendientes.
4. Publicación: actualizar docs para revisión de equipo.

## Plantilla de iteración

- Dominio:
- Requisitos fuente:
- Capacidades identificadas:
- Asignación provisional de servicios:
- Decisiones cerradas:
- Decisiones abiertas:
- Riesgos:
- Próxima acción:

## Reglas de calidad

1. No aceptar defaults sin justificar alternativas.
2. Separar hechos del enunciado vs hipótesis del equipo.
3. Mantener consistencia entre ADR, OpenAPI y diagramas.
4. Revisar semanalmente trazabilidad y deuda de diseño.
