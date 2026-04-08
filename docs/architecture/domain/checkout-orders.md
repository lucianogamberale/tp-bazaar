# Dominio Checkout y Ordenes

## Alcance

Este documento define el modelo de estados de orden y sus transiciones validas para los flujos obligatorios de Checkout y Ordenes.

## Fuente funcional

- Enunciado: epica Checkout y Ordenes (historias obligatorias).

## Modelo de estados de orden

Estados principales del flujo:

1. `pendiente_de_pago`
2. `confirmada`
3. `en_preparacion`
4. `enviada`
5. `entregada`

Estados de excepcion (deben existir en modelo de datos):

1. `pago_rechazado`
2. `cancelada`
3. `reembolso_en_proceso`
4. `reembolso_procesado`

## Transiciones permitidas (obligatorias)

1. `pendiente_de_pago -> confirmada` (pago aprobado)
2. `pendiente_de_pago -> pago_rechazado` (pago rechazado)
3. `confirmada -> en_preparacion` (avance vendedor)
4. `en_preparacion -> enviada` (avance vendedor, tracking opcional)
5. `enviada -> entregada` (confirmacion comprador)

## Reglas de negocio asociadas

1. No se permiten transiciones hacia atras.
2. Una transicion invalida debe devolver error de negocio claro.
3. El historial de transiciones debe guardarse con timestamp.
4. Al pasar a `enviada`, el tracking code es opcional.
5. El comprador solo puede confirmar entrega desde `enviada`.

## Estados de excepcion en este checkpoint

Aunque no se implementen funcionalidades optativas completas, el modelo debe contemplar:

- `cancelada`
- `reembolso_en_proceso`
- `reembolso_procesado`

En este checkpoint quedan deshabilitadas en UI las transiciones optativas no implementadas.

## Riesgos y decisiones abiertas

1. Definir representacion tecnica de la maquina de estados (tabla de transiciones o motor dedicado).
2. Definir nivel de auditoria para historial de estados.
3. Definir politica de reintentos para callbacks de pago.
