# ADR 008: Arquitectura y Gestión de Listas de Deseos (Wishlists)

**Fecha:** 7 de Abril de 2026  
**Estado:** Propuesto / Aprobado  
**Contexto:** Épica 10 (Funcionalidad de "Favoritos" o "Guardar para después").

## 1. El Problema y Contexto de Negocio

El Criterio de Aceptación de la Épica 10 establece que el comprador debe poder marcar productos como "Favoritos" para comprarlos en el futuro.
Intentar alojar esta funcionalidad en los microservicios existentes presenta graves problemas arquitectónicos:

- **Por qué no en el `Cart Service`:** El carrito usa Redis y tiene un TTL estricto de 7 días (se auto-destruye). La Wishlist, en cambio, debe ser persistente en el tiempo (el usuario espera ver sus favoritos guardados meses después).
- **Por qué no en el `Profile Service`:** Acoplaríamos el perfil de identidad del usuario con su intención comercial, rompiendo el Principio de Responsabilidad Única.
- **Por qué no en el `Catalog Service`:** El catálogo (MongoDB) está optimizado para lecturas masivas. Las acciones de "Like / Unlike" generan una altísima tasa de escritura (es el botón más presionado en un e-commerce después del scroll), lo cual degradaría el rendimiento de las búsquedas de otros usuarios.

## 2. Decisión Arquitectónica: Wishlist Service Dedicado

Se decide implementar un microservicio ligero y dedicado exclusivamente a esta función: el **Wishlist Service**.

### 2.1. Base de Datos

- Se utilizará **PostgreSQL**.
- **Justificación:** La estructura de datos es una relación simple de _muchos-a-muchos_ (`user_id` y `product_id`). PostgreSQL manejará esta tabla asociativa de manera extremadamente eficiente, garantizando la persistencia a largo plazo.

### 2.2. Patrón de Hidratación (Igual que el Carrito)

El `Wishlist Service` **no guardará el precio ni el nombre del producto**. Almacenará únicamente las claves foráneas (IDs).
Cuando el usuario acceda a la vista "Mis Favoritos" en la App Mobile, el flujo será:

1. El Gateway consulta al `Wishlist Service` y obtiene un array de IDs: `[prod_123, prod_456]`.
2. El `Wishlist Service` hace una petición interna masiva (`GET /internal/products?ids=...`) al **Catalog Service** (MongoDB).
3. El servicio ensambla la respuesta combinando los IDs con los datos reales y actualizados (precio, foto, stock) para devolvérselos al usuario.

## 3. Estrategia de Resiliencia y Fallos

- Si el producto deja de existir o es deshabilitado por el vendedor, la hidratación desde el `Catalog Service` devolverá `null` para ese ID.
- En ese escenario, el `Wishlist Service` mostrará el ítem en la interfaz del usuario con una etiqueta de **"Producto no disponible actualmente"**, pero **no lo borrará automáticamente** de la base de datos (quizás el vendedor reponga stock la semana siguiente y el usuario quiera comprarlo).

## 4. Consecuencias y Compromisos (Trade-offs)

- **Positivas:** Mantenemos las cachés de Redis (Carrito) y las lecturas de Mongo (Catálogo) limpias y rápidas. El microservicio es tan pequeño que su consumo de CPU y memoria será mínimo.
- **Negativas:** Obliga a mantener otra base de datos transaccional más. Al usar hidratación en tiempo real, si el `Catalog Service` se cae, los usuarios no podrán ver el detalle de sus favoritos (aunque esto es aceptable, ya que si el Catálogo está caído, toda la plataforma está inutilizable de todas formas).
