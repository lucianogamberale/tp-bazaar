# ADR 002: Separación de Dominios: Identidad (IAM) vs. Perfil de Usuario

**Fecha:** 6 de Abril de 2026  
**Estado:** Aprobado  
**Contexto:** Gestión del ciclo de vida de usuarios en la plataforma "Bazaar".

## 1. El Problema: El Antipatrón "God User" y Privacidad

Tradicionalmente, la información de un usuario se almacena en una única tabla o servicio. En una arquitectura de microservicios, esto genera:

- **Acoplamiento Excesivo:** El servicio de seguridad (IAM) se ve obligado a escalar innecesariamente cuando el catálogo consulta fotos de perfil.
- **Riesgo de Seguridad:** Exposición de hashes de contraseñas en consultas de presentación pública.
- **Inconsistencia en la Baja:** Cuando un usuario pide borrar su cuenta, si los datos están dispersos y no hay un flujo claro, quedan "datos fantasma" en el sistema.

## 2. Decisión: División de Responsabilidades y Eventos de Ciclo de Vida

Se divide el concepto de "Usuario" en dos microservicios independientes con bases de datos aisladas, comunicados mediante eventos asíncronos.

### 2.1. IAM Service (Identity & Access Management)

- **Misión:** Única fuente de verdad para seguridad. Responde a: _"¿Quién eres y es válida tu sesión?"_.
- **Datos:** UUID (`account_id`), Email, Password Hash, Roles.
- **Acción de Salida:** Publica eventos `UserRegistered` y `UserDeleted`.

### 2.2. Profile Service (Servicio de Perfiles)

- **Misión:** Gestión de identidad comercial y pública. Responde a: _"¿Cómo te ven los demás?"_.
- **Datos:** Nombre, Apellido, Bio, URL de Avatar, Reputación.
- **Acción de Entrada:** Escucha los eventos del IAM para crear o destruir perfiles locales.

## 3. Gestión de la Baja (Derecho al Olvido)

Para garantizar la integridad y la privacidad del usuario, se implementa el siguiente flujo de borrado:

1. **Gatillo:** El Administrador (vía Web) o el Usuario (vía Mobile) solicita la baja.
2. **IAM:** Marca al usuario como eliminado en su DB y revoca todos los tokens activos. Inmediatamente publica el evento `UserDeleted` en RabbitMQ.
3. **Profile Service:** Al recibir `UserDeleted`, elimina físicamente la información personal (Nombre, Email, Bio) para cumplir con la privacidad.
4. **Otros Servicios (Órdenes/Catálogo):** - El **Catálogo** oculta los productos del vendedor.
   - El **Order Service** _anonimiza_ la orden (ej: cambia "Juan Pérez" por "Usuario Eliminado") pero **no borra la orden**, ya que debe persistir por razones contables y legales.

## 4. Estrategia de Resiliencia (Consistency)

Debido a que los eventos pueden llegar desordenados o duplicados, aplicamos:

- **Escrituras Defensivas (UPSERT):** Si el `Profile Service` recibe una edición de perfil antes que el evento de creación, usará `INSERT ... ON CONFLICT UPDATE` para no fallar.
- **Idempotencia:** El consumidor de `UserDeleted` debe ser capaz de procesar el mensaje varias veces sin error, asegurando que si el perfil ya no existe, simplemente ignore la petición (`ACK`).

## 5. Beneficios Obtenidos

1. **Aislamiento de Seguridad:** El `Profile Service` puede estar en una zona de red más accesible, mientras que el `IAM` permanece protegido.
2. **Cumplimiento Legal:** El flujo de `UserDeleted` garantiza que no queden datos sensibles tras la baja de un usuario.
3. **Performance:** Las búsquedas de vendedores en el catálogo no tocan la base de datos de seguridad.
