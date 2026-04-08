# ADR 010: Captura de Eventos de Navegación y Recomendaciones (Clickstream)

**Fecha:** 7 de Abril de 2026  
**Estado:** Propuesto / Aprobado  
**Contexto:** Épica 12 (Mecanismo de captura del historial de navegación de los usuarios y motor de recomendaciones basado en visualizaciones recientes).

## 1. El Problema y Contexto de Negocio

El sistema requiere registrar cada vez que un usuario visualiza el detalle de un producto para poder construir su "Historial de Navegación" y ofrecer la sección "Vistos recientemente".
La proporción de lectura vs. escritura en la navegación es extrema: un usuario puede ver 50 productos antes de comprar 1 (o ninguno). Registrar estas 50 visualizaciones genera una avalancha de escrituras (_Write-Heavy Workload_). Alojarlas en el `Catalog Service` o `Order Service` degradaría el rendimiento de las operaciones críticas de los compradores y vendedores reales.

## 2. Decisión Arquitectónica: Tracking & Recommendation Service

Se decide extraer esta funcionalidad a un microservicio completamente independiente y no bloqueante llamado **Tracking & Recommendation Service**.

### 2.1. Base de Datos en Memoria

- Se utilizará **Redis** (estructuras de datos: _Sorted Sets_ o _Lists_).
- **Justificación:** Redis puede soportar decenas de miles de escrituras por segundo en memoria. Al almacenar el historial de navegación en un _Sorted Set_ por `user_id`, los productos se ordenan automáticamente por la fecha exacta en la que fueron vistos (`timestamp` como _score_). Además, se le puede aplicar un límite de tamaño (ej. solo recordar los últimos 20 productos) y un `TTL` (caducidad a los 30 días), automatizando la limpieza de datos inútiles.

## 3. Patrón de Ingesta "Fire-and-Forget"

Para garantizar que la captura del historial nunca ralentice la experiencia del usuario en la aplicación móvil o web, el contrato de comunicación será el siguiente:

1. **Captura en Segundo Plano:** Cuando el usuario entra a la pantalla de detalle de un producto, la aplicación móvil envía una petición HTTP `POST /tracking/views` (con el `product_id`) al API Gateway.
2. **Respuesta Inmediata:** El API Gateway o el `Tracking Service` responde instantáneamente con un `202 Accepted`, sin esperar a que el dato se guarde físicamente.
3. **Tolerancia a Fallos:** Si el `Tracking Service` está caído, la petición simplemente falla silenciosamente en el cliente. La experiencia de usuario no se interrumpe (el usuario puede seguir viendo el producto y comprarlo). Perder un clic en el historial es un riesgo aceptable frente a perder una venta.

## 4. Hidratación de Recomendaciones (Lectura)

El flujo para renderizar la pantalla "Vistos recientemente" compartirá el mismo patrón arquitectónico de la Wishlist y el Carrito:

1. El cliente pide sus recomendaciones al `Tracking Service`.
2. El servicio recupera desde Redis únicamente el array de IDs: `[prod_99, prod_12]`.
3. El servicio realiza una consulta interna masiva al **Catalog Service** para hidratar esos IDs con la información en tiempo real (foto, precio actual y disponibilidad).
4. Si un producto ya no existe, el `Tracking Service` lo descarta del array final y lo purga de Redis.

## 5. Consecuencias y Compromisos (Trade-offs)

- **Positivas:** La base de datos del Catálogo queda protegida del tráfico basura de clics. La ingesta es ultrarrápida al operar 100% en memoria RAM (Redis).
- **Negativas:** La analítica a largo plazo no es posible con este diseño (los datos en Redis expiran). Si en el futuro el negocio requiere entrenar modelos de Inteligencia Artificial complejos con el historial de meses enteros, se deberá añadir un mecanismo para volcar (hacer un _dump_) la memoria de Redis hacia un almacenamiento frío (ej. AWS S3 o un Data Lake).
