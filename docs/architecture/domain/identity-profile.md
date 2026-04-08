# Dominio Identidad y Perfil

## Alcance

Este documento describe la separación de responsabilidades entre IAM y Profile para las historias de Registro, Login y Edición de perfil.

## Fuente funcional

- Enunciado: épicas Usuarios y Perfil.

## Límites propuestos

### IAM Service

- Registro de cuentas.
- Login y manejo de sesión.
- Credenciales y seguridad.
- Validación de password y email.
- No gestiona foto, bio ni campos de identidad pública.

### Profile Service

- Datos públicos y editables del perfil.
- Visualización y edición de perfil propio.
- Gestión de foto, nombre y descripción.
- No gestiona contraseñas, tokens ni autenticación primaria.

## Datos por servicio

- IAM DB: userId, email, passwordHash, estado de sesión.
- Profile DB: userId, displayName, bio, avatarUrl.

## Integración entre dominios

- Evento candidato: UserRegistered.
- Propósito: bootstrap de perfil al crear cuenta.

## Capacidades del dominio (iteración actual)

1. Registro y login en IAM (`POST /auth/register`, `POST /auth/login`).
2. Edición de perfil en Profile (`PATCH /profile/me`).
3. Lectura de perfil propio en Profile (`GET /profile/me`).

## Criterios provisionales acordados

1. **Update de perfil**: se prioriza `PATCH` sobre `PUT` para soportar cambios parciales y evitar reemplazo completo accidental del recurso.
2. **Foto de perfil**: no se almacena el binario en PostgreSQL; se usa Object Storage y en Profile DB solo se persisten metadatos (`avatar_url`, `avatar_key`, `updated_at`).
3. **Implementación costo-cero/low-cost**: preferir proveedor con free tier; para local, usar alternativa compatible con S3.

## Riesgos y decisiones abiertas de Perfil

1. Foto de perfil: definir URL pública o firmada y pipeline de validación.
2. Validaciones: definir reglas mínimas de nombre, bio y formato de URL.
3. Semántica de actualización: confirmar PATCH parcial como contrato estándar.

## Decisiones abiertas

1. Política de autologin en registro.
2. Requisitos exactos de password segura.
3. Estrategia de almacenamiento de imagen de perfil.
4. Semántica de expiración de sesión y refresh.
