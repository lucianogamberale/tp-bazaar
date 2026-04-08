# ADR 007: Arquitectura y Gestión de Cupones de Descuento

**Fecha:** 7 de Abril de 2026  
**Estado:** Propuesto / Aprobado  
**Contexto:** Épicas 5 y 6 (Creación, gestión y aplicación de cupones de descuento en el Checkout).

## 1. El Problema y Contexto de Negocio

El enunciado exige que los vendedores puedan crear cupones de descuento personalizados y que los compradores puedan aplicarlos durante el Checkout para reducir el monto total a pagar.
Estas operaciones involucran lógicas complejas y volátiles: validación de códigos únicos, verificación de fechas de caducidad, control de límite de usos y cálculo matemático de descuentos (porcentajes o montos fijos).

Si delegamos esta responsabilidad al `Order Service`, este servicio perdería su foco (que es orquestar transacciones y pagos) y se acoplaría fuertemente a reglas de marketing, dificultando su mantenimiento y aumentando el riesgo de fallos durante el pago.

## 2. Decisión Arquitectónica: Microservicio Dedicado

Se decide crear un nuevo microservicio llamado **Discount Service** (o Coupon Service) encargado exclusivamente de la gestión del ciclo de vida de los cupones.

### 2.1. Responsabilidades del Discount Service

- **Gestión (Escritura):** Permitir a los vendedores crear (`POST`), listar (`GET`) y desactivar (`PATCH`) sus propios cupones.
- **Validación:** Garantizar la unicidad del código del cupón a nivel de base de datos para evitar colisiones.
- **Cálculo (Lectura/Cálculo):** Exponer un endpoint interno (`POST /internal/coupons/validate`) que reciba un código de cupón y el monto del carrito, y devuelva el descuento a aplicar (o un error si está vencido/agotado).

### 2.2. Base de Datos

- Se utilizará **PostgreSQL**.
- **Justificación:** Los cupones representan dinero y tienen reglas de límite de uso (ej. "Válido solo para los primeros 100 compradores"). Esto requiere garantías transaccionales (ACID) estrictas para evitar que, en un pico de concurrencia, 150 personas usen un cupón que estaba limitado a 100.

## 3. Flujo de Aplicación en el Checkout

Para garantizar que el cupón no sea alterado maliciosamente por el cliente y para proteger el `Order Service`, el flujo será el siguiente:

1. **Simulación en el Carrito:** Cuando el comprador ingresa el código en la pantalla de Checkout, el Frontend consulta al `Discount Service` (a través del Gateway) para mostrar el descuento en la UI.
2. **Checkout Real (Consistencia):** Al presionar "Pagar", el Frontend envía el payload de la orden al `Order Service` incluyendo el string `coupon_code`.
3. **Validación Síncrona:** Antes de pedirle el dinero al Gateway de Pagos, el `Order Service` realiza una llamada síncrona interna (HTTP/gRPC) al `Discount Service` consultando: _"¿Es válido este código y cuánto debo descontar del total?"_.
4. **Deducción de Uso:** Una vez que el pago se aprueba (`PAID`), el `Order Service` emitirá un evento asíncrono (`OrderPaidEvent`) por RabbitMQ. El `Discount Service` escuchará este evento y restará "1" al límite de usos del cupón utilizado.

## 4. Consecuencias y Compromisos (Trade-offs)

- **Positivas:** El `Order Service` se mantiene simple y enfocado. Si el `Discount Service` colapsa, el sistema puede estar configurado para simplemente no aceptar cupones temporalmente, pero el e-commerce puede seguir vendiendo (alta disponibilidad).
- **Negativas:** Se introduce un nuevo componente a la topología de red y se suma un "salto" de red (Network Hop) síncrono durante el flujo crítico del Checkout.
