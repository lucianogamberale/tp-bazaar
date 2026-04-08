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
| D-004 | Estrategia de foto de perfil      | CA2 Perfil      | A: URL publica, B: URL firmada       | Seguridad + costo  | Equipo Backend | Alta      | Abierta | 2026-04-11     |
| D-005 | Validaciones de perfil            | CA1/CA3 Perfil  | A: mínimas, B: estrictas             | UX + calidad datos | Equipo Backend | Media     | Abierta | 2026-04-12     |
| D-006 | Semántica de update de perfil     | CA1 Perfil      | A: PATCH parcial, B: PUT total       | Evolución API      | Equipo Backend | Media     | Abierta | 2026-04-12     |

## Notas de discusión (provisionales)

1. **D-006 (PATCH vs PUT)**: recomendación provisional = `PATCH` para edición de perfil por cambios parciales frecuentes y menor riesgo de sobreescritura.
2. **D-004 (foto de perfil)**: recomendación provisional = almacenar binario fuera de PostgreSQL (Object Storage) y guardar solo `avatar_url`/`avatar_key` en Profile DB.
3. **Costo**: priorizar alternativa gratis o de bajo costo (free tier). Para desarrollo local, usar storage S3-compatible self-hosted.
4. **Cierre pendiente**: confirmar política de URL pública vs URL firmada según alcance de seguridad y complejidad del checkpoint.

## Iteración 2 - Catálogo y Vendedor

| ID    | Decisión                                       | Contexto           | Opciones                                              | Impacto                             | Responsable    | Prioridad | Estado      | Fecha objetivo |
| ----- | ---------------------------------------------- | ------------------ | ----------------------------------------------------- | ----------------------------------- | -------------- | --------- | ----------- | -------------- |
| D-007 | Ownership escritura vs lectura en productos    | DD-013             | A: Un solo servicio, B: Product escribe + Catalog lee | Escalabilidad + claridad de dominio | Equipo Backend | Alta      | En análisis | 2026-04-13     |
| D-008 | Integración Product -> Catalog                 | DD-014             | A: Sync HTTP, B: Async eventos                        | Consistencia + resiliencia          | Equipo Backend | Alta      | En análisis | 2026-04-13     |
| D-009 | Semántica de producto no disponible en detalle | Catalog CA detalle | A: 404, B: 410, C: 200 con estado de negocio          | UX + contratos                      | Equipo Backend | Media     | Abierta     | 2026-04-14     |

## Iteración 3 - Carrito y Checkout

| ID    | Decisión                                    | Contexto             | Opciones                                                 | Impacto                             | Responsable    | Prioridad | Estado      | Fecha objetivo |
| ----- | ------------------------------------------- | -------------------- | -------------------------------------------------------- | ----------------------------------- | -------------- | --------- | ----------- | -------------- |
| D-010 | Invariantes de consistencia del checkout    | DD-017               | A: invariantes explícitos, B: manejo ad-hoc por servicio | Consistencia distribuida            | Equipo Backend | Alta      | En análisis | 2026-04-15     |
| D-011 | Momento de descuento de stock               | Checkout CA1/CA2/CA4 | A: antes de pago, B: al confirmar pago                   | Riesgo de sobreventa y compensación | Equipo Backend | Alta      | En análisis | 2026-04-15     |
| D-012 | Estrategia de idempotencia en checkout/pago | Checkout CA5         | A: key por intento, B: sin key explícita                 | Riesgo de doble cobro               | Equipo Backend | Alta      | En análisis | 2026-04-15     |
| D-013 | Política de concurrencia para último item   | Checkout CA4         | A: lock pesimista, B: validación optimista + retry       | Equidad + performance               | Equipo Backend | Media     | Abierta     | 2026-04-16     |
