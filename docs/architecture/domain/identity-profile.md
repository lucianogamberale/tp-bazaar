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

### Profile Service

- Datos públicos y editables del perfil.
- Visualización y edición de perfil propio.
- Gestión de foto, nombre y descripción.

## Datos por servicio

- IAM DB: userId, email, passwordHash, estado de sesión.
- Profile DB: userId, displayName, bio, avatarUrl.

## Integración entre dominios

- Evento candidato: UserRegistered.
- Propósito: bootstrap de perfil al crear cuenta.

## Decisiones abiertas

1. Política de autologin en registro.
2. Requisitos exactos de password segura.
3. Estrategia de almacenamiento de imagen de perfil.
4. Semántica de expiración de sesión y refresh.
