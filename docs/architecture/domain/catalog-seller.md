# Dominio Catalogo y Vendedor

## Alcance

Este documento define limites de responsabilidad entre Product and Inventory Service, Catalog Service y Order Service para las historias obligatorias de Catalogo y Vendedor.

## Fuente funcional

- Enunciado: epicas Catalogo y Vendedor.

## Separacion lectura vs escritura

### Product and Inventory Service (escritura)

- Crear producto (publicacion inicial).
- Editar producto (precio, descripcion, imagenes).
- Actualizar stock.
- Habilitar y deshabilitar publicacion.
- Aplicar reglas de ownership (solo el propietario gestiona su producto).

### Catalog Service (lectura)

- Home de catalogo (recientes, categorias, recomendaciones cuando existan).
- Listado paginado y busqueda por nombre/descripcion.
- Detalle de producto para compradores.
- Exclusion de productos sin stock o deshabilitados en vistas de consulta.

### Order Service (ventas del vendedor)

- Historial de ventas del vendedor.
- Detalle de venta con items del vendedor.
- Avance de estado de orden segun flujo permitido.
- Filtro por estado.

## Integracion minima por eventos

1. `ProductPublished`: Product -> Catalog.
2. `ProductUpdated`: Product -> Catalog.
3. `ProductDisabled`: Product -> Catalog.
4. `StockUpdated`: Product -> Catalog.
5. `OrderPaid` (o equivalente): Order -> Product para ajuste de stock confirmado.

## Contratos minimos (borrador)

- Escritura vendedor (Product): endpoints bajo `/seller/products`.
- Lectura catalogo (Catalog): endpoints bajo `/catalog`.
- Ventas vendedor (Order): endpoints bajo `/seller/orders`.

## Riesgos y decisiones abiertas

1. Definir consistencia temporal aceptable entre escritura en Product y lectura en Catalog.
2. Confirmar semantica de no disponibilidad (404, 410 o payload de negocio).
3. Confirmar machine state para avance de orden por vendedor.
