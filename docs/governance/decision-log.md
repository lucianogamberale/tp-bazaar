# Registro de Decisiones Abiertas

## Convención

- Estado: Abierta, En análisis, Cerrada.
- Prioridad: Alta, Media, Baja.
- Fecha objetivo: día comprometido para cierre.

## Plantilla

| ID    | Decisión                          | Contexto        | Opciones                             | Impacto            | Responsable    | Prioridad | Estado  | Fecha objetivo |
| ----- | --------------------------------- | --------------- | ------------------------------------ | ------------------ | -------------- | --------- | ------- | -------------- |
| D-001 | Política de autologin en registro | CA1 de Registro | A: tokens en 201, B: login posterior | Contratos IAM + UX | Equipo Backend | Alta      | Abierta | 2026-04-10     |
| D-002 | Estrategia de error en login      | CA2/CA3 Login   | A: 401 genérico, B: 400 genérico     | Seguridad + UX     | Equipo Backend | Alta      | Abierta | 2026-04-10     |
| D-003 | Manejo de sesión expirada         | CA4 Login       | A: refresh + retry, B: relogin       | UX + seguridad     | Equipo Backend | Alta      | Abierta | 2026-04-11     |
| D-004 | Estrategia de foto de perfil      | CA2 Perfil      | A: URL publica, B: URL firmada        | Seguridad + costo  | Equipo Backend | Alta      | Abierta | 2026-04-11     |
| D-005 | Validaciones de perfil            | CA1/CA3 Perfil  | A: mínimas, B: estrictas              | UX + calidad datos | Equipo Backend | Media     | Abierta | 2026-04-12     |
| D-006 | Semántica de update de perfil     | CA1 Perfil      | A: PATCH parcial, B: PUT total        | Evolución API      | Equipo Backend | Media     | Abierta | 2026-04-12     |
