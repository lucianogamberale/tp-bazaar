# ADR 001: Arquitectura de Seguridad, Identidad y Gestión de Sesiones

## Estado

Aprobado

## Contexto

El ecosistema de "Bazaar" requiere una arquitectura de microservicios que soporte múltiples clientes con naturalezas tecnológicas distintas: una aplicación móvil en React Native (orientada a Compradores y Vendedores) y un panel de administración Web (SPA en React).

El enunciado exige además:

1. API Gateway como punto único de entrada con validación de tokens y rate limiting.
2. Tokens de sesión con expiración y soporte de refresh.
3. Seguridad en endpoints sensibles (login/recupero) y control de acceso por roles.

Debemos garantizar:

1. Los microservicios internos no se saturen validando sesiones contra la base de datos en cada petición.
2. La aplicación móvil almacene las credenciales de forma segura.
3. El panel web esté protegido contra vectores de ataque comunes en navegadores, específicamente **XSS (Cross-Site Scripting)** y **CSRF (Cross-Site Request Forgery)**.
4. Exista un control estricto de roles (RBAC) y un mecanismo seguro para el cierre de sesión (Logout).

## Decisión

Se adopta una arquitectura de **Autenticación Sin Estado (Stateless)** basada en el estándar **JWT (JSON Web Tokens)**, centralizando la emisión de credenciales en un **IAM Service (Identity and Access Management)** y delegando la validación perimetral a un **API Gateway**.

Se define el siguiente esquema operativo:

1. **Access Token (JWT)** de corta duración con `user_id` y `roles`.
2. **Refresh Token** de mayor duración para renovar sesión sin reingresar credenciales.
3. **Validación perimetral en Gateway**: firma JWT, expiración y reglas globales de acceso/rate limiting.
4. **Autorización de negocio en microservicios**: ownership y reglas específicas por recurso.
5. **Logout**: invalidación de refresh token + limpieza del almacenamiento del cliente.

Detalles relevantes de decisión:

- **Access Token (JWT):** duración corta (ejemplo de referencia: 15 minutos).
- **Refresh Token:** duración mayor (ejemplo de referencia: 7 días).
- **Mobile:** tokens en almacenamiento seguro del dispositivo.
- **Web:** tokens en cookies seguras (`HttpOnly`, `Secure`, `SameSite`).

## Alternativas consideradas

1. **Sesiones stateful en base de datos por request**.

- Pros: revocación inmediata y simple.
- Contras: alto costo de I/O y menor escalabilidad en arquitectura distribuida.

2. **JWT sin refresh token**.

- Pros: implementación más simple.
- Contras: peor UX por relogin frecuente y mayor fricción en clientes móviles.

3. **Validación completa de autorización solo en gateway**.

- Pros: punto único de control.
- Contras: acoplamiento excesivo del gateway a reglas de negocio y pérdida de defensa en profundidad.

## Consecuencias

- Positivas:
  - Escalabilidad al evitar validación de sesión contra DB en cada request.
  - Mejor UX con refresh token y sesiones renovables.
  - Alineación con requerimiento de API Gateway y rate limiting.
  - Separación clara entre autorización perimetral y autorización de negocio.

- Negativas o tradeoffs:
  - Revocación de access token no inmediata por naturaleza stateless.
  - Mayor complejidad en frontend/mobile para manejar refresh y expiraciones.
  - Operación adicional de seguridad para rotación de claves y manejo de cookies en web.

- Riesgos y mitigaciones:
  - Riesgo: uso indebido de token robado durante ventana de expiración.
  - Mitigación: access token corto + refresh invalidable + rotación de claves.
  - Riesgo: abuso en endpoints de autenticación.
  - Mitigación: rate limiting por IP y por cuenta en login/recupero.

## Impacto en artefactos

- OpenAPI afectado:
  - `docs/api/contracts/iam.openapi.yaml` (`/auth/register`, `/auth/login`, futuro `/auth/refresh`, `/auth/logout`).
  - Contratos del gateway para políticas de autenticación/autorización.

- Diagramas afectados:
  - C4 de contenedores (`docs/architecture/c4/containers.mmd`) con gateway e IAM.
  - Secuencias de registro/login y refresh/logout.

- Servicios y datos impactados:
  - IAM Service: emisión, refresh y revocación de tokens.
  - API Gateway: validación de JWT y rate limiting.
  - Clientes Web/Mobile: estrategia de almacenamiento y renovación de sesión.
