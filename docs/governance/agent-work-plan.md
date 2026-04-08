# Plan de Trabajo con Agentes

## Objetivo

Guiar la colaboración entre equipo y agentes de IA para descubrir dominio, cerrar decisiones de arquitectura y producir entregables defendibles ante cátedra.

## Objetivos de calidad

1. Trazabilidad completa desde épicas y CA a capacidades y servicios.
2. Coherencia entre ADRs, OpenAPI y diagramas.
3. Evidencia de iteración y revisión de decisiones.

## Estado final esperado (target)

Al finalizar el camino de aprendizaje, el repositorio debe tener como minimo:

1. Microservicios con responsabilidades y limites claramente definidos por dominio.
2. Endpoints documentados por servicio en OpenAPI (requests, responses, errores).
3. Diagramas C4 (Contexto, Contenedores, Componentes) consistentes con la descomposicion real.
4. Diagramas de secuencia de los flujos criticos (registro, checkout, etc.).
5. ADRs por decisiones relevantes, con alternativas descartadas y consecuencias.
6. Matriz de trazabilidad completa: CA -> capacidad -> owner -> contrato/evento -> ADR.

## Flujo recomendado

### Fase A - Descubrimiento de dominio (Plan + Explore)

1. Relevar épicas y CA del enunciado.
2. Identificar capacidades (comandos, consultas, eventos).
3. Detectar ambigüedades y riesgos.

Salida:

- Actualización de `traceability-matrix.md`.
- Lista priorizada en `decision-log.md`.

### Fase B - Diseño por dominio (Plan)

1. Definir límites de servicio por dominio.
2. Proponer contratos mínimos y eventos.
3. Evaluar alternativas y tradeoffs.

Salida:

- Documento de dominio en `docs/architecture/domain/`.
- ADR nuevo o actualización de ADR existente.

### Fase C - Formalización de entregables (Ejecución)

1. Crear o ajustar OpenAPI en `docs/api/contracts/`.
2. Actualizar diagramas C4 y de secuencia.
3. Registrar estado y deuda de diseño.

Salida:

- Paquete de entrega revisable por equipo y profesor.

## Plan desagregado en micro-tareas

### Iteración tipo (60-90 min)

1. Seleccionar una única épica objetivo.
2. Leer solo esa sección del enunciado y listar historias/CA textuales.
3. Convertir cada CA en una capacidad técnica verificable.
4. Proponer owner provisional (servicio) por capacidad.
5. Marcar incertidumbres y supuestos por cada capacidad.
6. Definir artefactos mínimos a actualizar (trazabilidad, decisiones, dominio, contrato o diagrama).
7. Cerrar la iteración con semáforo:
   - Verde: lista para diseño detallado.
   - Amarillo: faltan decisiones.
   - Rojo: faltan datos esenciales.

### Checklist de sesión

1. ¿Separamos claramente hechos del enunciado de hipótesis del equipo?
2. ¿Cada CA quedó mapeado a una capacidad?
3. ¿Cada capacidad tiene owner provisional?
4. ¿Quedaron decisiones abiertas registradas con fecha y responsable?
5. ¿Se actualizó la matriz de trazabilidad?

### Marco de decision iterativa (caso por caso)

Para cada duda de arquitectura, usar siempre esta secuencia:

1. Problema: que CA/requisito exige resolver esta decision.
2. Opciones: 2 o 3 alternativas concretas y comparables.
3. Tradeoffs: impacto en complejidad, costo, resiliencia, mantenibilidad y time-to-market.
4. Eleccion provisional: opcion recomendada para el checkpoint actual.
5. Criterio de validacion: como sabremos rapido si la decision fue correcta.
6. Registro: guardar la decision en `decision-log.md` y, si aplica, en ADR.

## Secuencia sugerida para descubrir dominio

1. Usuarios + Perfil (base de identidad).
2. Catálogo + Vendedor (fuente de productos).
3. Carrito + Checkout y Ordenes (flujo transaccional).
4. Reviews + Recomendaciones (calidad de contenido y personalizacion).
5. Administración + Métricas + Notificaciones (capacidades transversales).

## Buenas practicas de mercado para trabajo con agentes

1. Timebox estricto por iteración: evitar sesiones largas sin cierre documental.
2. Una decisión por vez: no mezclar varios dominios en una misma discusión.
3. Evidencia primero: citar CA o requisito no funcional antes de proponer arquitectura.
4. Documentación viva: cada hipótesis debe poder confirmarse o descartarse en la siguiente iteración.
5. Contratos tempranos: escribir OpenAPI preliminar antes de implementar para detectar lagunas.
6. Arquitectura trazable: toda decisión importante debe terminar en ADR con alternativas.
7. Criterio de salida explícito: no avanzar de fase sin definir qué se considera "done".
8. Vertical slices primero: cerrar un flujo extremo a extremo antes de abrir muchos frentes.
9. ADRs livianos y frecuentes: registrar decisiones cuando se toman, no al final.
10. Contrato antes que codigo: alinear backend y clientes desde OpenAPI.
11. Resiliencia by design: explicitar idempotencia, retries y compensaciones en flujos distribuidos.
12. Seguridad por defecto: nunca mezclar datos de identidad sensible con perfil publico.
13. Archivos multimedia fuera de DB transaccional: usar Object Storage (preferentemente free tier/low-cost) y persistir solo referencias en PostgreSQL.

## Como guiar al agente en cada iteracion

Para obtener mejores resultados, al iniciar cada sesion indicar:

1. Epica objetivo y CA especificos.
2. Alcance exacto de la iteracion (que SI y que NO).
3. Artefacto esperado de salida (trazabilidad, ADR, OpenAPI, diagrama).
4. Nivel de profundidad (borrador, intermedio, listo para presentar).
5. Restricciones del checkpoint (tiempo, tecnologias, decisiones ya tomadas).

## Buenas practicas para prompts

1. Pedir siempre formato de salida estructurado.
2. Exigir supuestos explícitos.
3. Separar hechos del enunciado de hipótesis.
4. Solicitar riesgos y decisiones pendientes.

## Definición de Done por iteración

1. Hay al menos un dominio con trazabilidad completa.
2. Hay decisiones cerradas y abiertas claramente registradas.
3. OpenAPI y diagramas del dominio están alineados.
4. El equipo puede explicar y defender las decisiones.
