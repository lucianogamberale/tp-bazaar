# Docs

Documentación técnica viva del proyecto.

## Carpetas

- `architecture/`: C4 y modelado de dominios.
- `diagrams/sequence/`: flujos de negocio críticos en Mermaid.
- `api/contracts/`: contratos OpenAPI por servicio.
- `governance/`: trazabilidad, plan por checkpoints y decisiones abiertas.

## Convención de actualización

1. Cada cambio relevante de arquitectura debe impactar en:
   - ADR correspondiente.
   - Contrato OpenAPI afectado.
   - Diagrama relacionado (si cambia el flujo).
2. Registrar estado de madurez y supuestos en governance.
