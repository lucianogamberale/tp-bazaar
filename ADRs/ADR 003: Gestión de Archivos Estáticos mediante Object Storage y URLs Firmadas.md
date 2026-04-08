# ADR 003: Gestión de Archivos Estáticos, URLs Firmadas y Optimización Móvil

## Estado

Aprobado

## Contexto

El enunciado exige soporte de imágenes para perfil y productos con restricciones explícitas:

1. Formatos permitidos: JPEG, PNG o WebP.
2. Tamaños máximos: 5 MB para perfil y 10 MB por imagen de producto.

Problema a resolver:

1. Evitar sobrecargar bases transaccionales con binarios grandes.
2. Mantener performance y costo razonable en almacenamiento/transferencia.
3. Permitir subida segura de archivos sin exponer infraestructura interna.

## Decisión

Se adopta almacenamiento de media en **Object Storage** y persistencia en bases de negocio solo de metadatos/referencias (`url`, `key`, `mime`, `size`, `owner`).

Se define el siguiente flujo de carga:

1. Cliente solicita autorización de carga a backend.
2. Backend valida intención de carga y genera URL prefirmada temporal.
3. Cliente sube el archivo directamente al storage.
4. Cliente confirma la carga para registrar referencia en servicio de dominio.

Controles de seguridad mínimos:

1. Validación de MIME permitido (`image/jpeg`, `image/png`, `image/webp`).
2. Validación de tamaño según tipo de imagen (perfil o producto).
3. URLs prefirmadas con expiración corta.
4. Generación de claves de objeto seguras (UUID/ULID) para evitar colisiones.

Implementación de costo:

1. Desarrollo local: solución S3-compatible local.
2. Producción: proveedor con free tier o bajo costo, documentado en ADR de despliegue.

## Alternativas consideradas

1. **Guardar binarios en PostgreSQL (BLOB/bytea)**.

- Pros: arquitectura más simple en etapa temprana.
- Contras: crecimiento de DB, backups más costosos, peor performance para consultas transaccionales.

2. **Pasar todos los uploads por backend y luego subir a storage**.

- Pros: control centralizado del archivo.
- Contras: mayor consumo de CPU/red en servicios de negocio y peor escalabilidad.

3. **Object Storage + URL prefirmada (decisión elegida)**.

- Pros: desacopla media de servicios core, escala mejor y reduce carga del backend.
- Contras: mayor complejidad operativa y gestión de archivos huérfanos.

## Consecuencias

- Positivas:
  - Menor presión sobre bases transaccionales.
  - Mejor escalabilidad para catálogo y perfil.
  - Menor costo relativo por almacenamiento de objetos.

- Negativas o tradeoffs:
  - Mayor complejidad en flujo de subida y confirmación.
  - Necesidad de gobernar lifecycle de objetos (huérfanos, reemplazos, borrados).

- Riesgos y mitigaciones:
  - Riesgo: archivos huérfanos por cortes entre subida y confirmación.
  - Mitigación: jobs de reconciliación/lifecycle rules por prefijo temporal.
  - Riesgo: abuso de URLs prefirmadas.
  - Mitigación: TTL corto, validación de MIME/tamaño y rate limiting en emisión.

## Impacto en artefactos

- OpenAPI afectado:
  - Endpoints para solicitar URL prefirmada y confirmar carga.
  - Endpoints de perfil/producto que persisten y exponen `avatar_url`/`image_url`.

- Diagramas afectados:
  - Secuencias de carga de imagen (cliente -> backend -> storage -> confirmación).
  - C4 de contenedores con componente de Object Storage.

- Servicios y datos impactados:
  - Profile Service y Product&Inventory para metadatos de imágenes.
  - Infraestructura de storage para objetos binarios.
