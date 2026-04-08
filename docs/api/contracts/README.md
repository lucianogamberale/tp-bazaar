# Contratos API

Este directorio centraliza los contratos OpenAPI por servicio.

## Convenciones

1. Un archivo por servicio.
2. Versionar cambios significativos.
3. Mantener ejemplos para casos exitosos y de error.
4. Mantener trazabilidad con ADR y criterios de aceptación.

## Checklist de calidad para endpoints

1. Cada endpoint debe mapear a al menos un CA del enunciado.
2. Definir codigos de exito y error de forma explicita.
3. Incluir ejemplos de request y response para casos principales.
4. Separar errores de validacion, negocio y autorizacion.
5. Documentar invariantes relevantes (idempotencia, orden de estados, etc.).
6. Versionar cambios breaking y registrar impacto en ADR/decision-log.

## Archivos sugeridos

- `iam.openapi.yaml`
- `profile.openapi.yaml`
- `catalog.openapi.yaml`
- `order.openapi.yaml`
