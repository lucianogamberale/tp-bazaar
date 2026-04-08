# ADR 002: Separación de Dominios: Identidad (IAM) vs. Perfil de Usuario

## Estado

Aprobado

## Contexto

El enunciado separa explícitamente los dominios de Usuarios y Perfil:

1. Usuarios: registro, login, recupero, sesión y seguridad.
2. Perfil: identidad visible (nombre, foto, descripción) y visualización pública/propia.

Además, la arquitectura esperada es de microservicios con responsabilidades delimitadas y bases de datos independientes.

Problema a resolver:

1. Evitar el antipatrón de "usuario único" que mezcla credenciales y datos públicos.
2. Minimizar exposición de datos sensibles en consultas de perfil.
3. Definir sincronización entre dominio de identidad (IAM) y dominio de perfil sin acoplamiento fuerte.

## Decisión

Se separa el concepto de usuario en dos bounded contexts:

1. **IAM Service** (identidad y acceso): fuente de verdad para autenticación/autorización.
2. **Profile Service** (identidad pública): fuente de verdad para datos visibles de perfil.

Asignación de datos y responsabilidades:

1. IAM persiste: `account_id`, `email`, `password_hash`, `roles`, estado de sesión.
2. Profile persiste: `user_id`, `display_name`, `bio`, `avatar_url` y campos públicos derivados.
3. Profile no guarda secretos de autenticación.
4. IAM no expone datos de perfil más allá de identificadores mínimos necesarios.

Integración entre dominios:

1. Se usa integración asíncrona por eventos de ciclo de vida de cuenta (ejemplo: alta de usuario para bootstrap de perfil).
2. Los consumidores deben ser idempotentes y tolerar duplicados/desorden de mensajes.

Nota de alcance:

1. El evento de baja de usuario y su propagación se mantienen como decisión de diseño sujeta a validación de alcance por checkpoint.

## Alternativas consideradas

1. **Modelo unificado (IAM + perfil en un solo servicio/BD)**.

- Pros: menor complejidad inicial.
- Contras: mayor riesgo de exposición de datos, acoplamiento y escalado conjunto.

2. **Servicios separados con integración síncrona directa**.

- Pros: consistencia inmediata.
- Contras: más acoplamiento en tiempo de ejecución y mayor fragilidad ante fallos de red.

3. **Servicios separados con eventos asíncronos (decisión elegida)**.

- Pros: desacoplamiento, resiliencia y mejor escalabilidad.
- Contras: consistencia eventual y necesidad de idempotencia.

## Consecuencias

- Positivas:
  - Mejor aislamiento de seguridad entre credenciales y perfil público.
  - Límites de dominio más claros para evolutividad y ownership de equipos.
  - Menor carga del dominio IAM para lecturas de catálogo/perfiles.

- Negativas o tradeoffs:
  - Mayor complejidad de integración entre servicios.
  - Consistencia eventual en creación/actualización de vistas de perfil.

- Riesgos y mitigaciones:
  - Riesgo: eventos duplicados o fuera de orden.
  - Mitigación: consumidores idempotentes + escrituras defensivas (upsert).
  - Riesgo: divergencia temporal entre IAM y Profile.
  - Mitigación: reintentos, observabilidad y reconciliación periódica.

## Impacto en artefactos

- OpenAPI afectado:
  - `docs/api/contracts/iam.openapi.yaml` (registro/login y lifecycle de cuenta).
  - `docs/api/contracts/profile.openapi.yaml` (perfil propio/público).

- Diagramas afectados:
  - Secuencia de registro con bootstrap de perfil.
  - C4 de contenedores con IAM, Profile y broker de eventos.

- Servicios y datos impactados:
  - IAM Service y su base aislada para seguridad.
  - Profile Service y su base para identidad pública.
  - Mensajería asíncrona para sincronizar ciclo de vida de cuenta.
