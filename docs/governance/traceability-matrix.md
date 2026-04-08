# Matriz de Trazabilidad

Tabla base para mapear requerimientos del enunciado hacia capacidades, owners y artefactos técnicos.

## Campos recomendados

- Épica
- Historia
- Criterio de aceptación
- Tipo (Obligatoria/Optativa)
- Capacidad del sistema
- Servicio owner (provisional o final)
- Contrato/API o Evento relacionado
- ADR relacionado
- Estado (Definido/Validar/Abierto)
- Notas

## Plantilla

| Épica    | Historia | CA  | Tipo        | Capacidad    | Owner | Contrato o Evento   | ADR     | Estado  | Notas                         |
| -------- | -------- | --- | ----------- | ------------ | ----- | ------------------- | ------- | ------- | ----------------------------- |
| Usuarios | Registro | CA1 | Obligatoria | RegisterUser | IAM   | POST /auth/register | ADR-001 | Abierto | Definir política de autologin |

## Iteración 1 - Usuarios (Registro)

| Épica    | Historia | CA  | Tipo        | Capacidad del sistema                          | Owner | Contrato o Evento   | ADR     | Estado   | Notas                                                                |
| -------- | -------- | --- | ----------- | ---------------------------------------------- | ----- | ------------------- | ------- | -------- | -------------------------------------------------------------------- |
| Usuarios | Registro | CA1 | Obligatoria | Registrar cuenta con datos válidos             | IAM   | POST /auth/register | ADR-001 | Abierto  | Pendiente cerrar política de autologin y formato final de respuesta  |
| Usuarios | Registro | CA2 | Obligatoria | Validar unicidad de email y rechazar duplicado | IAM   | POST /auth/register | ADR-001 | Definido | Error de conflicto por email ya existente                            |
| Usuarios | Registro | CA3 | Obligatoria | Validar formato de email antes de persistir    | IAM   | POST /auth/register | ADR-001 | Definido | No debe crear cuenta con formato inválido                            |
| Usuarios | Registro | CA4 | Obligatoria | Validar política mínima de contraseña segura   | IAM   | POST /auth/register | ADR-001 | Validar  | Falta acordar reglas exactas (largo, complejidad, mensajes de error) |

## Iteración 1 - Usuarios (Login)

| Épica    | Historia | CA  | Tipo        | Capacidad del sistema                               | Owner         | Contrato o Evento | ADR     | Estado   | Notas                                                               |
| -------- | -------- | --- | ----------- | --------------------------------------------------- | ------------- | ----------------- | ------- | -------- | ------------------------------------------------------------------- |
| Usuarios | Login    | CA1 | Obligatoria | Autenticar credenciales y abrir sesión              | IAM           | POST /auth/login  | ADR-001 | Definido | Definir formato exacto de sesión (tokens/cookies) por cliente       |
| Usuarios | Login    | CA2 | Obligatoria | Responder error genérico ante contraseña incorrecta | IAM           | POST /auth/login  | ADR-001 | Definido | No revelar si falló email o password                                |
| Usuarios | Login    | CA3 | Obligatoria | Responder error genérico ante email no registrado   | IAM           | POST /auth/login  | ADR-001 | Definido | Debe ser indistinguible de CA2 para evitar enumeración de cuentas   |
| Usuarios | Login    | CA4 | Obligatoria | Detectar sesión expirada y forzar reautenticación   | IAM + Gateway | Authorization JWT | ADR-001 | Validar  | Falta definir mecanismo para preservar acción pendiente en frontend |
