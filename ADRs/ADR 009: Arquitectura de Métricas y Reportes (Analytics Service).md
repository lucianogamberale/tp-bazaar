# ADR 009: Arquitectura de Métricas y Reportes (Analytics Service)

**Fecha:** 7 de Abril de 2026  
**Estado:** Propuesto / Aprobado  
**Contexto:** Épica 8 (Panel de Administración: Visualización de métricas de negocio, ventas por categoría, monto transaccionado y productos populares).

## 1. El Problema y Contexto de Negocio

El perfil de Administrador requiere acceder a un panel web (Dashboard) con reportes estadísticos y métricas de rendimiento de la plataforma "Bazaar" (ej: ingresos totales, top productos más vendidos, categorías más rentables).
Extraer esta información requiere ejecutar consultas de agregación pesadas. Si estas consultas se ejecutan directamente contra el `Order Service` (PostgreSQL transaccional), se corre el riesgo de degradar el rendimiento del motor de base de datos, afectando el flujo crítico de compras de los usuarios y violando el principio de aislamiento.

## 2. Decisión Arquitectónica: Analytics Service Dedicado

Se decide implementar un microservicio asíncrono denominado **Analytics Service** (o Reporting Service) que actúe como un pequeño _Data Warehouse_ en tiempo real, separando físicamente el tráfico de lectura analítica del tráfico transaccional.

### 2.1. Base de Datos para Analítica

- Se utilizará **MongoDB** (aprovechando el motor de _Aggregation Pipeline_) o una instancia separada de **PostgreSQL** optimizada para lecturas.
- **Justificación:** Al no requerir validaciones estrictas de negocio para insertar (solo consume datos que ya fueron validados), la base de datos puede estar indexada específicamente para filtrar por rangos de fechas (ej. "Ventas del último mes").

## 3. Flujo de Ingesta de Datos (Event-Driven)

El servicio no hace peticiones HTTP a otros servicios para armar sus reportes, sino que reacciona pasivamente a lo que ocurre en el sistema:

1. **Escucha de Eventos:** El `Analytics Service` se suscribe a eventos clave en RabbitMQ, como:
   - `OrderPaidEvent` (emitido por el Payment/Order Service).
   - `ProductCreatedEvent` (emitido por Inventory Service).
   - `UserRegistered` (emitido por IAM Service).
2. **Proyección y Pre-Cálculo:** Al recibir un `OrderPaidEvent` (ej: "Se vendió 1 termo por $50 en la categoría Hogar"), el servicio actualiza sus proyecciones internas. En lugar de guardar la orden cruda, puede incrementar contadores directamente:
   - `UPDATE reportes SET total_ventas = total_ventas + 50 WHERE fecha = 'hoy' AND categoria = 'Hogar'`
3. **Consulta Inmediata:** Cuando el Administrador entra al Panel Web, el frontend consulta al `Analytics Service`, el cual devuelve los datos ya sumados y agrupados en milisegundos, listos para graficar.

## 4. Consecuencias y Compromisos (Trade-offs)

- **Positivas:** - **Aislamiento absoluto:** El Administrador puede refrescar su panel de reportes 100 veces por segundo y no afectará en absoluto a la aplicación móvil ni al proceso de Checkout.
  - **Escalabilidad:** El servicio está preparado para procesar miles de eventos por minuto en segundo plano.
- **Negativas:**
  - **Consistencia Eventual:** Los reportes del administrador tendrán una demora (latencia) de algunos milisegundos o segundos en reflejar la última venta respecto a la realidad. Para un contexto de reportes gerenciales, este retraso es universalmente aceptado.
  - **Duplicación de Datos:** Parte de la información de órdenes y productos existirá duplicada en la base de datos de Analytics, lo cual aumenta levemente los costos de almacenamiento.
