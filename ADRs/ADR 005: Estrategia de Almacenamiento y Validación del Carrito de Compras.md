# ADR 005: Estrategia de Almacenamiento y Validación del Carrito de Compras

**Fecha:** 5 de Abril de 2026  
**Estado:** Propuesto (En debate tecnológico)  
**Contexto:** Dominio del Carrito de Compras ("Bazaar"), responsable de mantener el estado temporal de los productos seleccionados por el usuario antes de proceder al Checkout.

## 1. El Problema y Contexto de Negocio

El carrito de compras maneja datos altamente volátiles (efímeros). Los usuarios agregan, modifican y eliminan productos constantemente, y muchos carritos son abandonados.
Además, el sistema debe protegerse de inconsistencias de datos: no podemos permitir que un usuario pague el precio "viejo" de un producto que fue modificado mientras estaba en su carrito, ni permitir la compra de un artículo que fue deshabilitado por el vendedor o que se quedó sin stock.

## 2. Decisiones Arquitectónicas Base (Aprobadas)

### 2.1. Bloqueo a Usuarios Anónimos

Alineado con el Criterio de Aceptación 5 de la Épica Catálogo, **no se implementarán carritos locales ni basados en cookies para usuarios no logueados**. El Gateway rechazará cualquier petición al `Cart Service` que no contenga un token JWT válido del `IAM Service`.

### 2.2. Patrón de "Hidratación" (Single Source of Truth para Precios)

Para evitar discrepancias financieras, la base de datos del carrito **solo almacenará referencias**: el `user_id`, el `product_id` y la `quantity` (cantidad).
**No se guardarán precios, nombres ni estados en el carrito.** Cuando el usuario consulte su carrito, el `Cart Service` hará una consulta síncrona al `Catalog Service` (MongoDB) para "hidratar" la respuesta con los precios reales, el stock actual y el estado (activo/deshabilitado) en ese exacto milisegundo.

## 3. Opciones de Infraestructura (Para debate del equipo)

Se debe definir el motor de base de datos para el `Cart Service`. Se proponen dos alternativas viables:

### Opción A: Redis (Base de datos Clave-Valor en memoria) - _Recomendada_

- **Cómo funciona:** Se guarda un objeto JSON o Hash por usuario (ej. clave `cart:user_123`).
- **Pros:** Velocidad de lectura/escritura sub-milisegundo. Soporta limpieza automática configurando un TTL (Time-To-Live), por ejemplo, que los carritos se borren solos a los 7 días de inactividad.
- **Cons:** Introduce una nueva tecnología al stack del equipo que requiere configuración inicial.

### Opción B: PostgreSQL (Base de datos Relacional)

- **Cómo funciona:** Tablas clásicas `carts` y `cart_items` ligadas por Foreign Keys.
- **Pros:** El equipo ya está familiarizado con la tecnología y simplifica el despliegue local.
- **Cons:** La escritura/lectura constante genera desgaste en el motor. Requiere programar un _CronJob_ (tarea programada en el backend) que corra diariamente ejecutando un `DELETE FROM carts WHERE updated_at < NOW() - INTERVAL '7 days'` para evitar acumulación de basura.

## 4. Resolución de Casos Borde y Criterios de Aceptación (CA)

El diseño propuesto resuelve los siguientes escenarios exigidos por la cátedra:

1. **Persistencia Multi-dispositivo (CA 4 - Agregar producto):** Al estar vinculado al `user_id` en el backend, el usuario conserva su carrito independientemente de si entra desde la app móvil o la web.
2. **Productos Deshabilitados/Sin Stock (CA 4 - Gestión):** Gracias a la "Hidratación", si un ítem en el carrito es deshabilitado por el vendedor, el sistema lo detecta al leer del Catálogo y bloquea el botón de Checkout, notificando visualmente al usuario.
3. **Límite de Stock al Agregar (CA 3 - Agregar producto):** Antes de insertar o sumar cantidades en el carrito, se consulta el stock disponible en el Catálogo. Si la suma supera el límite físico, la API devuelve un error de validación (400 Bad Request).
4. **Concurrencia Extrema en el Pago (CA 4 - Checkout):** El carrito asume que la validación final no le pertenece. Delega la responsabilidad de los bloqueos transaccionales por concurrencia ("dos usuarios comprando la última unidad al mismo tiempo") al `Order Service` durante el Checkout.
