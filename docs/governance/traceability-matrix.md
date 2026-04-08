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

| Épica    | Historia                        | CA  | Tipo        | Capacidad del sistema                                                       | Owner   | Contrato o Evento          | ADR     | Estado   | Notas                                                                |
| -------- | ------------------------------- | --- | ----------- | --------------------------------------------------------------------------- | ------- | -------------------------- | ------- | -------- | -------------------------------------------------------------------- |
| Catálogo | Home                            | CA1 | Obligatoria | Mostrar selección de productos recientes activos con stock                  | Catalog | GET /catalog/home          | ADR-004 | Validar  | Definir criterio exacto de “reciente” y límite de elementos          |
| Catálogo | Home                            | CA2 | Obligatoria | Redirigir búsqueda desde home a listado con resultados                      | Catalog | GET /catalog/search        | ADR-004 | Validar  | Confirmar si redirección es frontend y backend solo expone búsqueda  |
| Catálogo | Home                            | CA3 | Obligatoria | Navegar por categoría desde home hacia catálogo filtrado                    | Catalog | GET /catalog?category=...  | ADR-004 | Validar  | Falta cerrar fuente de verdad de categorías predefinidas             |
| Catálogo | Home                            | CA4 | Obligatoria | Mostrar recomendaciones personalizadas cuando estén disponibles             | Catalog | GET /catalog/home          | ADR-010 | Abierto  | Depende del dominio de recomendaciones y disponibilidad de historial |
| Catálogo | Home                            | CA5 | Obligatoria | Permitir navegación sin login y exigir autenticación en acciones protegidas | Gateway | Authorization policy       | ADR-001 | Validar  | Definir estrategia uniforme para “agregar al carrito/comprar”        |
| Catálogo | Listado y búsqueda de productos | CA1 | Obligatoria | Listar productos activos con stock de forma paginada                        | Catalog | GET /catalog/products      | ADR-004 | Definido | Incluir nombre, precio, imagen principal y vendedor                  |
| Catálogo | Listado y búsqueda de productos | CA2 | Obligatoria | Buscar por nombre o descripción y ordenar por relevancia                    | Catalog | GET /catalog/search?q=...  | ADR-004 | Validar  | Definir motor de relevancia y estrategia de matching                 |
| Catálogo | Listado y búsqueda de productos | CA3 | Obligatoria | Mostrar estado vacío claro cuando no hay resultados                         | Catalog | GET /catalog/search?q=...  | ADR-004 | Definido | Sin errores técnicos expuestos al usuario                            |
| Catálogo | Listado y búsqueda de productos | CA4 | Obligatoria | Excluir productos sin stock o deshabilitados de listados y búsquedas        | Catalog | Query filters              | ADR-004 | Definido | Requiere sincronización con estado de inventario                     |
| Catálogo | Detalle de producto             | CA1 | Obligatoria | Exponer vista completa del producto activo                                  | Catalog | GET /catalog/products/{id} | ADR-004 | Definido | Debe incluir datos del vendedor e imágenes                           |
| Catálogo | Detalle de producto             | CA2 | Obligatoria | Informar falta de stock y deshabilitar agregar al carrito                   | Catalog | GET /catalog/products/{id} | ADR-004 | Definido | Mantener visibilidad del producto aun sin stock                      |
| Catálogo | Detalle de producto             | CA3 | Obligatoria | Informar no disponibilidad cuando producto está deshabilitado               | Catalog | GET /catalog/products/{id} | ADR-004 | Definido | Definir semántica de respuesta (404 vs 410 vs 200 con mensaje)       |

## Iteración 2 - Vendedor (CA obligatorios)

| Épica    | Historia                         | CA  | Tipo        | Capacidad del sistema                                                   | Owner               | Contrato o Evento                  | ADR     | Estado   | Notas                                                                |
| -------- | -------------------------------- | --- | ----------- | ----------------------------------------------------------------------- | ------------------- | ---------------------------------- | ------- | -------- | -------------------------------------------------------------------- |
| Vendedor | Publicar producto                | CA1 | Obligatoria | Publicar producto con campos obligatorios e imagen principal            | Product & Inventory | POST /seller/products              | ADR-004 | Definido | Primera imagen define portada en catálogo                            |
| Vendedor | Publicar producto                | CA2 | Obligatoria | Validar campos obligatorios y rechazar publicación incompleta           | Product & Inventory | POST /seller/products              | ADR-004 | Definido | Retornar errores por campo faltante                                  |
| Vendedor | Publicar producto                | CA3 | Obligatoria | Validar precio > 0 antes de crear producto                              | Product & Inventory | POST /seller/products              | ADR-004 | Definido | Regla de negocio de pricing mínimo                                   |
| Vendedor | Publicar producto                | CA4 | Obligatoria | Validar stock inicial >= 0 antes de crear producto                      | Product & Inventory | POST /seller/products              | ADR-004 | Definido | Stock negativo prohibido                                             |
| Vendedor | Gestión de stock y publicaciones | CA1 | Obligatoria | Editar datos de producto propio y reflejar cambios en catálogo          | Product & Inventory | PATCH /seller/products/{id}        | ADR-004 | Validar  | Definir latencia esperada de propagación a vistas de catálogo        |
| Vendedor | Gestión de stock y publicaciones | CA2 | Obligatoria | Actualizar stock manualmente con validación de no-negatividad           | Product & Inventory | PATCH /seller/products/{id}/stock  | ADR-004 | Definido | Rechazar actualizaciones negativas                                   |
| Vendedor | Gestión de stock y publicaciones | CA3 | Obligatoria | Deshabilitar/habilitar publicación según visibilidad y stock            | Product & Inventory | PATCH /seller/products/{id}/status | ADR-004 | Definido | Al habilitar, reingresa solo si stock > 0                            |
| Vendedor | Gestión de stock y publicaciones | CA4 | Obligatoria | Marcar producto sin stock automáticamente al llegar a cero por venta    | Product & Inventory | StockUpdated event                 | ADR-004 | Validar  | Depende de integración con flujo de órdenes                          |
| Vendedor | Gestión de stock y publicaciones | CA5 | Obligatoria | Autorizar modificación solo al propietario del producto                 | Product & Inventory | Authorization policy               | ADR-001 | Definido | Validar ownership en capa de aplicación                              |
| Vendedor | Gestión de stock y publicaciones | CA6 | Obligatoria | Gestionar imágenes (alta, baja, orden) garantizando al menos una imagen | Product & Inventory | PATCH /seller/products/{id}/images | ADR-003 | Validar  | Requiere decisión final de estrategia de storage y orden de imágenes |
| Vendedor | Historial de ventas              | CA1 | Obligatoria | Listar ventas del vendedor ordenadas por fecha descendente              | Order               | GET /seller/orders                 | ADR-006 | Definido | Solo órdenes con al menos un ítem del vendedor                       |
| Vendedor | Historial de ventas              | CA2 | Obligatoria | Mostrar detalle de venta con ítems propios y datos de envío/comprador   | Order               | GET /seller/orders/{id}            | ADR-006 | Validar  | Definir nivel de detalle de datos del comprador por privacidad       |
| Vendedor | Historial de ventas              | CA3 | Obligatoria | Permitir avance de estado de orden según flujo permitido                | Order               | PATCH /seller/orders/{id}/status   | ADR-006 | Validar  | Cerrar máquina de estados canónica                                   |
| Vendedor | Historial de ventas              | CA4 | Obligatoria | Filtrar historial por estado de orden                                   | Order               | GET /seller/orders?status=...      | ADR-006 | Definido | Mantener paginación y orden por fecha                                |

## Iteración 3 - Carrito (CA obligatorios)

| Épica   | Historia                    | CA  | Tipo        | Capacidad del sistema                                               | Owner | Contrato o Evento              | ADR     | Estado   | Notas                                                                       |
| ------- | --------------------------- | --- | ----------- | ------------------------------------------------------------------- | ----- | ------------------------------ | ------- | -------- | --------------------------------------------------------------------------- |
| Carrito | Agregar producto al carrito | CA1 | Obligatoria | Agregar item al carrito y recalcular total                          | Cart  | POST /cart/items               | ADR-005 | Definido | Requiere stock disponible al momento de agregar                             |
| Carrito | Agregar producto al carrito | CA2 | Obligatoria | Incrementar cantidad si el item ya existe respetando stock          | Cart  | POST /cart/items               | ADR-005 | Definido | Upsert de item por productId                                                |
| Carrito | Agregar producto al carrito | CA3 | Obligatoria | Limitar cantidad por stock disponible y devolver máximo permitido   | Cart  | POST /cart/items               | ADR-005 | Definido | Mensaje de error orientado a negocio                                        |
| Carrito | Agregar producto al carrito | CA4 | Obligatoria | Persistir carrito entre sesiones del usuario                        | Cart  | GET /cart                      | ADR-005 | Validar  | Definir política exacta de expiración/TTL                                   |
| Carrito | Gestión del carrito         | CA1 | Obligatoria | Mostrar carrito con detalle por item y total general                | Cart  | GET /cart                      | ADR-005 | Definido | Puede requerir hidratación de datos de producto                             |
| Carrito | Gestión del carrito         | CA2 | Obligatoria | Modificar cantidad de item dentro de límites de stock               | Cart  | PATCH /cart/items/{itemId}     | ADR-005 | Definido | Recalcular totales inmediatamente                                           |
| Carrito | Gestión del carrito         | CA3 | Obligatoria | Eliminar item del carrito y recalcular total                        | Cart  | DELETE /cart/items/{itemId}    | ADR-005 | Definido | Operación idempotente recomendada                                           |
| Carrito | Gestión del carrito         | CA4 | Obligatoria | Señalizar items no disponibles y bloquear checkout hasta resolución | Cart  | GET /cart + availability flags | ADR-005 | Validar  | Definir fuente de verdad para disponibilidad (Catalog vs Product/Inventory) |

## Iteración 3 - Checkout y Órdenes (CA obligatorios)

| Épica              | Historia                      | CA  | Tipo        | Capacidad del sistema                                                                   | Owner                            | Contrato o Evento                   | ADR     | Estado   | Notas                                                          |
| ------------------ | ----------------------------- | --- | ----------- | --------------------------------------------------------------------------------------- | -------------------------------- | ----------------------------------- | ------- | -------- | -------------------------------------------------------------- |
| Checkout y Órdenes | Checkout e inicio de pago     | CA1 | Obligatoria | Confirmar checkout con pago aprobado, confirmar orden, descontar stock y vaciar carrito | Order + Payment + Product + Cart | POST /checkout + payment callback   | ADR-006 | Validar  | Definir secuencia exacta de transacciones y compensaciones     |
| Checkout y Órdenes | Checkout e inicio de pago     | CA2 | Obligatoria | Marcar pago rechazado sin modificar stock y conservar carrito                           | Order + Payment + Cart           | POST /checkout                      | ADR-006 | Definido | Orden en estado pago rechazado                                 |
| Checkout y Órdenes | Checkout e inicio de pago     | CA3 | Obligatoria | Bloquear checkout por stock insuficiente antes de intentar pago                         | Order + Product                  | POST /checkout                      | ADR-006 | Definido | Debe retornar items conflictivos                               |
| Checkout y Órdenes | Checkout e inicio de pago     | CA4 | Obligatoria | Resolver concurrencia del último item sin doble compra ni doble cobro                   | Order + Product + Payment        | POST /checkout                      | ADR-006 | Validar  | Requiere control transaccional y reserva/confirmación de stock |
| Checkout y Órdenes | Checkout e inicio de pago     | CA5 | Obligatoria | Garantizar idempotencia para evitar doble cobro en reintentos                           | Order + Payment                  | Idempotency-Key                     | ADR-006 | Validar  | Definir llave idempotente y ventana temporal                   |
| Checkout y Órdenes | Checkout e inicio de pago     | CA6 | Obligatoria | Exigir dirección válida para confirmar orden                                            | Order                            | POST /checkout                      | ADR-006 | Definido | Validación obligatoria de address                              |
| Checkout y Órdenes | Checkout e inicio de pago     | CA7 | Obligatoria | Persistir dirección de entrega en la orden confirmada                                   | Order                            | Order aggregate                     | ADR-006 | Definido | Debe quedar visible para vendedor                              |
| Checkout y Órdenes | Estado y seguimiento de orden | CA1 | Obligatoria | Mostrar estado actual, historial de transiciones e items de orden                       | Order                            | GET /orders/{id}                    | ADR-006 | Definido | Incluir timestamps de transición                               |
| Checkout y Órdenes | Estado y seguimiento de orden | CA2 | Obligatoria | Mantener consistencia del estado de orden entre sesiones                                | Order                            | GET /orders/{id}                    | ADR-006 | Definido | Fuente única de verdad en Order                                |
| Checkout y Órdenes | Estado y seguimiento de orden | CA3 | Obligatoria | Reflejar actualización de estado hecha por vendedor                                     | Order                            | PATCH /seller/orders/{id}/status    | ADR-006 | Definido | Propagación visible al comprador                               |
| Checkout y Órdenes | Estado y seguimiento de orden | CA4 | Obligatoria | Rechazar transiciones inválidas fuera del flujo definido                                | Order                            | PATCH /seller/orders/{id}/status    | ADR-006 | Definido | Máquina de estados con validaciones                            |
| Checkout y Órdenes | Estado y seguimiento de orden | CA5 | Obligatoria | Permitir código de seguimiento opcional al pasar a enviada                              | Order                            | PATCH /seller/orders/{id}/status    | ADR-006 | Validar  | Definir formato de trackingCode                                |
| Checkout y Órdenes | Estado y seguimiento de orden | CA6 | Obligatoria | Mostrar tracking code al comprador cuando exista                                        | Order                            | GET /orders/{id}                    | ADR-006 | Definido | Campo opcional en detalle de orden                             |
| Checkout y Órdenes | Estado y seguimiento de orden | CA7 | Obligatoria | Permitir confirmación de entrega por comprador y transición a entregada                 | Order                            | PATCH /orders/{id}/confirm-delivery | ADR-006 | Validar  | Confirmar precondición estado enviada                          |
| Checkout y Órdenes | Historial de compras          | CA1 | Obligatoria | Listar órdenes del comprador por fecha descendente con estado total y fecha             | Order                            | GET /orders                         | ADR-006 | Definido | Soportar paginación y orden                                    |
| Checkout y Órdenes | Historial de compras          | CA2 | Obligatoria | Mostrar detalle completo de orden desde historial                                       | Order                            | GET /orders/{id}                    | ADR-006 | Definido | Reutiliza endpoint de detalle                                  |
| Checkout y Órdenes | Historial de compras          | CA3 | Obligatoria | Filtrar historial por estado                                                            | Order                            | GET /orders?status=...              | ADR-006 | Definido | Filtro combinable con paginación                               |
