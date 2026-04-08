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
