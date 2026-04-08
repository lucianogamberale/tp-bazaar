# ADR 001: Arquitectura de Seguridad, Identidad y Gestión de Sesiones

**Fecha:** 5 de Abril de 2026
**Proyecto:** 2026C1 - Bazaar
**Estado:** Aprobado
**Autores:** Equipo de Arquitectura Backend

## 1. Contexto y Problema

El ecosistema de "Bazaar" requiere una arquitectura de microservicios que soporte múltiples clientes con naturalezas tecnológicas distintas: una aplicación móvil en React Native (orientada a Compradores y Vendedores) y un panel de administración Web (SPA en React).

Debemos garantizar que:

1. Los microservicios internos no se saturen validando sesiones contra la base de datos en cada petición.
2. La aplicación móvil almacene las credenciales de forma segura.
3. El panel web esté protegido contra vectores de ataque comunes en navegadores, específicamente **XSS (Cross-Site Scripting)** y **CSRF (Cross-Site Request Forgery)**.
4. Exista un control estricto de roles (RBAC) y un mecanismo seguro para el cierre de sesión (Logout).

## 2. Decisiones Arquitectónicas

Se adopta una arquitectura de **Autenticación Sin Estado (Stateless)** basada en el estándar **JWT (JSON Web Tokens)**, centralizando la emisión de credenciales en un **IAM Service (Identity and Access Management)** y delegando la validación perimetral a un **API Gateway**.

### 2.1. Ciclo de Vida de los Tokens

Se implementará un sistema de doble token para balancear seguridad y experiencia de usuario (UX):

- **Access Token (JWT):** Tiempo de vida corto (ej. 15 minutos). Contiene el _payload_ de identidad (ID de usuario, roles). Será utilizado para autorizar las peticiones a los microservicios.
- **Refresh Token (Opaco/JWT de larga duración):** Tiempo de vida largo (ej. 7 días). Se utiliza exclusivamente para solicitar un nuevo Access Token cuando este expira, sin pedirle al usuario que vuelva a ingresar su contraseña.

### 2.2. Validación de Identidad y Autorización de Roles (RBAC)

Para evitar que tráfico malicioso alcance nuestra red interna, se establece la siguiente frontera de responsabilidades:

- **Responsabilidad del API Gateway (Validación de grano grueso):** El Gateway interceptará toda petición entrante. Verificará criptográficamente la firma del JWT (sin ir a la base de datos) usando la clave pública del IAM Service. Además, validará el acceso a rutas globales; por ejemplo, si la ruta empieza con `/api/admin/*`, el Gateway bloqueará automáticamente la petición con un `403 Forbidden` si el JWT no contiene el rol `ADMIN`.
- **Responsabilidad de los Microservicios (Validación de grano fino):** Los microservicios asumen que si la petición llegó, la identidad es real. Ellos se encargarán de la autorización de negocio. Por ejemplo, el `Order Service` verificará que el `user_id` del token coincida con el `buyer_id` de la orden antes de devolver el historial de compras.

### 2.3. Almacenamiento Seguro por Plataforma

- **Aplicación Mobile (React Native):** Al no ser vulnerables a CSRF ni XSS tradicional, ambos tokens se almacenarán en el almacenamiento seguro nativo del dispositivo (ej. `SecureStore` en Expo / `Keychain`). El cliente inyectará el Access Token en el header `Authorization: Bearer <token>`.
- **Panel Administrativo (Web React):** Para evitar el robo de tokens vía XSS, el Frontend no tendrá acceso directo al token mediante JavaScript. El API Gateway (o el IAM) enviará los tokens utilizando cookies seguras:
  - `HttpOnly`: Invisible para JavaScript.
  - `Secure`: Transmisión exclusiva sobre HTTPS.
  - `SameSite=Strict`: El navegador solo adjuntará la cookie si la petición se origina exactamente en el mismo dominio (mitiga CSRF).

### 2.4. Mecanismo de Cierre de Sesión (Logout)

Dado que los JWT son "stateless" y no pueden borrarse mágicamente de la red una vez emitidos, el flujo de Logout será el siguiente:

1. El cliente (App o Web) hace una petición `POST /auth/logout`.
2. El **IAM Service** recibe la petición y elimina/invalida el **Refresh Token** de su base de datos o caché, impidiendo futuras renovaciones.
3. El cliente borra los tokens de su almacenamiento local (SecureStore o borrado de Cookies).
4. El Access Token remanente vivirá como máximo 15 minutos (hasta su expiración natural), minimizando la ventana de exposición.

## 3. Consecuencias y Compromisos (Trade-offs)

- **Complejidad del Cliente:** Los desarrolladores frontend y mobile deberán implementar interceptores robustos en Axios/Fetch para manejar las transiciones de expiración (`401 Unauthorized`) y ejecutar el refresco ("Silent Refresh") de manera transparente para el usuario.
- **Revocación Diferida:** Al usar JWT sin estado, la revocación absoluta no es inmediata. En caso de un compromiso crítico (ej. baneo de un usuario o robo de cuenta detectado), el IAM Service deberá emitir un evento para agregar ese token a una "Blacklist" temporal en Redis alojada en el API Gateway, añadiendo una ligera penalización de latencia en peticiones de alta seguridad.
