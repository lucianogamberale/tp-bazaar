[IMPORTANTE] no estoy seguro todavia de esto, capaz se puede guardar la imagen en la db y ya. 

# ADR 003: Gestión de Archivos Estáticos, URLs Firmadas y Optimización Móvil

**Fecha:** 7 de Abril de 2026  
**Estado:** Aprobado  
**Contexto:** Necesidad de almacenar y servir imágenes de perfil y de productos en el proyecto "Bazaar" minimizando costos, optimizando el rendimiento de la base de datos y protegiendo el ancho de banda de los usuarios móviles.

## 1. El Problema

El almacenamiento de imágenes directamente en bases de datos relacionales (PostgreSQL) degrada el rendimiento de las consultas, aumenta los costos de respaldo y satura la memoria del servidor. Además, procesar subidas de archivos pesados a través de microservicios consume recursos críticos.
Por otro lado, las cámaras de los dispositivos móviles modernos generan imágenes de entre 5MB y 10MB. Permitir la subida de estos archivos crudos saturaría el almacenamiento en la nube y haría que el catálogo de productos sea inmanejable y lento para los compradores.

## 2. Decisión Arquitectónica

Se decide desacoplar el almacenamiento de archivos utilizando un modelo de **Almacenamiento de Objetos (Object Storage)** y **URLs Pre-firmadas (Presigned URLs)**, delegando el procesamiento pesado al cliente móvil.

### 2.1. Proveedores Seleccionados (Costo $0)

- **Desarrollo Local:** Se utilizará **MinIO** desplegado mediante Docker Compose (API compatible con S3).
- **Entorno de Producción:** Se optará por **Supabase Storage** o **Cloudinary**, aprovechando sus capas gratuitas y CDN integrada.

### 2.2. Flujo de Subida de Archivos (Compresión y Presigned URLs)

Para evitar que los archivos pasen por nuestro backend y proteger el ancho de banda, el flujo estricto será:

1. **Selección y Compresión (Frontend):** El usuario selecciona una foto en la App Mobile. La aplicación (usando librerías nativas de React Native) **comprimirá la imagen localmente** reduciendo su calidad y dimensiones antes de enviarla.
2. **Solicitud de URL (Backend):** El cliente solicita permiso de subida al `Profile Service` o `Catalog Service`.
3. **Generación Segura:** El servicio genera una URL temporal firmada criptográficamente (vía S3 SDK).
4. **Subida Directa (Cliente -> Storage):** El cliente realiza un `PUT` directo del archivo comprimido hacia el Object Storage usando dicha URL.
5. **Confirmación:** El cliente notifica al backend el éxito de la subida para registrar la ruta en la base de datos de negocio.

## 3. Detalles de Implementación y Seguridad (Prevención de Abusos)

Para evitar que usuarios malintencionados abusen de las URLs pre-firmadas subiendo malware (`.exe`) o archivos gigantes (`.mp4`), el backend aplicará restricciones estrictas al momento de generar la firma:

- **Validación de Tipo (MIME Type):** La URL firmada forzará un `ContentType` específico (ej. `image/jpeg`, `image/png`, `image/webp`). Si el cliente intenta hacer el `PUT` con otro tipo de archivo, el proveedor de la nube rechazará la subida automáticamente.
- **Límite de Tamaño (Content-Length):** Se establecerá un límite estricto de tamaño en la firma (ej. máximo 2MB). Las imágenes que superen este peso serán rechazadas por el Object Storage.
- **Ventana de Tiempo:** Las URLs de subida tendrán una validez máxima de 5 minutos.
- **Sanitización:** Los nombres de los archivos se generarán en el backend utilizando UUIDs para evitar colisiones y ataques de inyección de rutas (Path Traversal).

## 4. Consecuencias y Compromisos (Trade-offs)

- **Complejidad en el Cliente:** Traslada la responsabilidad de manipular y comprimir imágenes a los desarrolladores de React Native, requiriendo el uso de librerías adicionales (ej. `react-native-image-crop-picker` o `react-native-image-resizer`).
- **Gestión de Huérfanos:** Existe la posibilidad de que un usuario suba una imagen al Storage pero la red falle antes de notificar al backend (Paso 5). Se asume que la implementación de un script periódico de limpieza (Garbage Collection) o el uso de un ciclo de vida en el bucket (borrar archivos temporales no confirmados en 24h) mitigará este riesgo.
