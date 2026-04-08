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

## Iteración 1 - Perfil (Edición de perfil)

| Épica  | Historia          | CA  | Tipo        | Capacidad del sistema                                            | Owner   | Contrato o Evento | ADR     | Estado   | Notas                                                                    |
| ------ | ----------------- | --- | ----------- | ---------------------------------------------------------------- | ------- | ----------------- | ------- | -------- | ------------------------------------------------------------------------ |
| Perfil | Edición de perfil | CA1 | Obligatoria | Persistir cambios válidos de nombre, foto y descripción          | Profile | PATCH /profile/me | ADR-002 | Definido | Reflejo inmediato en respuesta de perfil propio                          |
| Perfil | Edición de perfil | CA2 | Obligatoria | Actualizar imagen de perfil y propagarla a vistas de identidad   | Profile | PATCH /profile/me | ADR-003 | Validar  | Falta cerrar estrategia final de media (URL firmada/publica, validacion) |
| Perfil | Edición de perfil | CA3 | Obligatoria | Validar datos de perfil y rechazar persistencia en caso inválido | Profile | PATCH /profile/me | ADR-002 | Definido | Devolver errores de validación por campo                                 |

## Iteración 2 - Catálogo (CA obligatorios)

| Épica    | Historia                      | CA  | Tipo        | Capacidad del sistema                                                      | Owner   | Contrato o Evento          | ADR     | Estado  | Notas                                                                 |
| -------- | ----------------------------- | --- | ----------- | -------------------------------------------------------------------------- | ------- | -------------------------- | ------- | ------- | --------------------------------------------------------------------- |
| Catálogo | Home                          | CA1 | Obligatoria | Mostrar selección de productos recientes activos con stock                 | Catalog | GET /catalog/home          | ADR-004 | Validar | Definir criterio exacto de “reciente” y límite de elementos           |
| Catálogo | Home                          | CA2 | Obligatoria | Redirigir búsqueda desde home a listado con resultados                     | Catalog | GET /catalog/search        | ADR-004 | Validar | Confirmar si redirección es frontend y backend solo expone búsqueda   |
| Catálogo | Home                          | CA3 | Obligatoria | Navegar por categoría desde home hacia catálogo filtrado                   | Catalog | GET /catalog?category=...  | ADR-004 | Validar | Falta cerrar fuente de verdad de categorías predefinidas              |
| Catálogo | Home                          | CA4 | Obligatoria | Mostrar recomendaciones personalizadas cuando estén disponibles            | Catalog | GET /catalog/home          | ADR-010 | Abierto | Depende del dominio de recomendaciones y disponibilidad de historial   |
| Catálogo | Home                          | CA5 | Obligatoria | Permitir navegación sin login y exigir autenticación en acciones protegidas| Gateway | Authorization policy       | ADR-001 | Validar | Definir estrategia uniforme para “agregar al carrito/comprar”         |
| Catálogo | Listado y búsqueda de productos | CA1 | Obligatoria | Listar productos activos con stock de forma paginada                       | Catalog | GET /catalog/products      | ADR-004 | Definido| Incluir nombre, precio, imagen principal y vendedor                   |
| Catálogo | Listado y búsqueda de productos | CA2 | Obligatoria | Buscar por nombre o descripción y ordenar por relevancia                   | Catalog | GET /catalog/search?q=...  | ADR-004 | Validar | Definir motor de relevancia y estrategia de matching                  |
| Catálogo | Listado y búsqueda de productos | CA3 | Obligatoria | Mostrar estado vacío claro cuando no hay resultados                        | Catalog | GET /catalog/search?q=...  | ADR-004 | Definido| Sin errores técnicos expuestos al usuario                             |
| Catálogo | Listado y búsqueda de productos | CA4 | Obligatoria | Excluir productos sin stock o deshabilitados de listados y búsquedas       | Catalog | Query filters              | ADR-004 | Definido| Requiere sincronización con estado de inventario                      |
| Catálogo | Detalle de producto           | CA1 | Obligatoria | Exponer vista completa del producto activo                                 | Catalog | GET /catalog/products/{id} | ADR-004 | Definido| Debe incluir datos del vendedor e imágenes                            |
| Catálogo | Detalle de producto           | CA2 | Obligatoria | Informar falta de stock y deshabilitar agregar al carrito                  | Catalog | GET /catalog/products/{id} | ADR-004 | Definido| Mantener visibilidad del producto aun sin stock                       |
| Catálogo | Detalle de producto           | CA3 | Obligatoria | Informar no disponibilidad cuando producto está deshabilitado              | Catalog | GET /catalog/products/{id} | ADR-004 | Definido| Definir semántica de respuesta (404 vs 410 vs 200 con mensaje)        |
