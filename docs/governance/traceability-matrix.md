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
