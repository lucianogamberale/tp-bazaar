# Backlog de Descubrimiento del Dominio

Este backlog desagrega el descubrimiento por épicas en tareas pequeñas y revisables.

## Convenciones

- Estado: `todo`, `doing`, `done`, `blocked`.
- Evidencia: referencia a CA del enunciado y archivo actualizado.
- Tamaño de tarea: maximo 30 minutos por item.

## Iteración 1 - Usuarios y Perfil

### Bloque A: Registro y Login

| ID     | Tarea                                                      | Estado | Evidencia esperada                    |
| ------ | ---------------------------------------------------------- | ------ | ------------------------------------- |
| DD-001 | Extraer CA de Registro (1 a 4) textual                     | done   | Fila creada en matriz de trazabilidad |
| DD-002 | Mapear capacidades para Registro                           | done   | Capacidad por CA en trazabilidad      |
| DD-003 | Proponer owner provisional para cada capacidad de Registro | done   | Owner en trazabilidad                 |
| DD-004 | Extraer CA de Login (1 a 4) textual                        | todo   | Fila creada en trazabilidad           |
| DD-005 | Mapear capacidades para Login                              | todo   | Capacidad por CA en trazabilidad      |
| DD-006 | Registrar decisiones abiertas de Registro/Login            | todo   | Entrada en decision-log               |

### Bloque B: Perfil

| ID     | Tarea                                                      | Estado | Evidencia esperada                     |
| ------ | ---------------------------------------------------------- | ------ | -------------------------------------- |
| DD-007 | Extraer CA de Edicion de perfil (1 a 3)                    | todo   | Fila creada en trazabilidad            |
| DD-008 | Mapear capacidades para Edicion de perfil                  | todo   | Capacidad por CA en trazabilidad       |
| DD-009 | Definir limites IAM vs Profile para esos CA                | todo   | Documento identity-profile actualizado |
| DD-010 | Registrar riesgos tecnicos (foto, validaciones, sincronia) | todo   | Entrada en decision-log                |

## Iteración 2 - Catalogo y Vendedor

| ID     | Tarea                                               | Estado | Evidencia esperada                |
| ------ | --------------------------------------------------- | ------ | --------------------------------- |
| DD-011 | Extraer CA obligatorios de Catalogo                 | todo   | Trazabilidad actualizada          |
| DD-012 | Extraer CA obligatorios de Vendedor                 | todo   | Trazabilidad actualizada          |
| DD-013 | Definir capacidades de lectura vs escritura         | todo   | Documento de dominio actualizado  |
| DD-014 | Identificar eventos minimos entre Product y Catalog | todo   | Decision-log y diagrama secuencia |

## Iteración 3 - Carrito y Checkout/Ordenes

| ID     | Tarea                                                   | Estado | Evidencia esperada              |
| ------ | ------------------------------------------------------- | ------ | ------------------------------- |
| DD-015 | Extraer CA obligatorios de Carrito                      | todo   | Trazabilidad actualizada        |
| DD-016 | Extraer CA obligatorios de Checkout y Ordenes           | todo   | Trazabilidad actualizada        |
| DD-017 | Definir invariantes de consistencia del flujo de compra | todo   | Decision-log + ADR candidato    |
| DD-018 | Definir modelo de estados de orden                      | todo   | Documento de dominio y diagrama |

## Iteración 4 - Dominios transversales

| ID     | Tarea                                                             | Estado | Evidencia esperada           |
| ------ | ----------------------------------------------------------------- | ------ | ---------------------------- |
| DD-019 | Extraer CA de Administracion y Metricas                           | todo   | Trazabilidad actualizada     |
| DD-020 | Extraer CA de Notificaciones y Recomendaciones                    | todo   | Trazabilidad actualizada     |
| DD-021 | Definir integraciones por eventos para analitica y notificaciones | todo   | Diagrama y decision-log      |
| DD-022 | Priorizar optativas por impacto y riesgo                          | todo   | Checkpoints-plan actualizado |

## Criterios de avance

1. No marcar `done` sin evidencia en archivo versionado.
2. Cada iteración debe cerrar al menos una decision abierta de prioridad alta.
3. Si una tarea queda `blocked`, crear accion de desbloqueo con responsable y fecha.
