# TP Bazaar

Repositorio de arquitectura y diseño para la materia Ingeniería de Software II.

## Objetivo

Construir una plataforma de e-commerce basada en microservicios, con foco en:

- Separación de responsabilidades por dominio.
- Contratos de API como fuente de verdad.
- Consistencia distribuida y resiliencia ante fallos.
- Trazabilidad entre enunciado, decisiones y entregables.

## Estructura sugerida

- `enunciado.md`: requerimientos funcionales y no funcionales de cátedra.
- `microservicios.md`: hipótesis inicial de partición de servicios.
- `ADRs/`: decisiones de arquitectura.
- `docs/`: documentación técnica viva (diagramas, contratos, gobernanza).
- `.github/AGENTS.md`: guía de trabajo con agentes de IA.

## Cómo usar este repositorio

1. Tomar `enunciado.md` como fuente canónica.
2. Tratar `microservicios.md` y ADRs existentes como hipótesis, no como verdad cerrada.
3. Mantener trazabilidad en `docs/governance/traceability-matrix.md`.
4. Registrar decisiones abiertas y cierres en `docs/governance/decision-log.md`.

## Entregables de arquitectura (alineados al enunciado)

- ADRs por decisiones relevantes.
- Contratos OpenAPI por servicio.
- Diagramas C4 (Contexto, Contenedores, Componentes).
- Diagramas de secuencia de flujos críticos.
- Plan iterativo por checkpoints.
