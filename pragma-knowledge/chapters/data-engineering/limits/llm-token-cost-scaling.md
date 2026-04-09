# Límite: Escalamiento de Costos de Tokens del LLM en Operaciones a Gran Escala

## Contexto de Identificación

Identificado en la operación del Agente de IA para ingesta Bronze dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma) al escalar el uso del agente a múltiples fuentes de datos y múltiples clientes simultáneamente. El problema se manifiesta cuando el consumo acumulado de tokens del LLM (Large Language Model — Modelo de Lenguaje Grande) en un periodo determinado erosiona el margen de rentabilidad del proyecto, calculado mediante el modelo COCOMO III (Constructive Cost Model).

La capa Nexo del SOPP rastrea el consumo de tokens por cliente como parte de su observabilidad total, lo que permite identificar este límite de forma cuantitativa.

Escenarios donde se presenta:

- Clientes con alto volumen de fuentes nuevas (> 20 fuentes/mes) que requieren ejecuciones frecuentes del agente.
- Fuentes de datos complejas (> 100 columnas, JSON — JavaScript Object Notation — anidado) que demandan múltiples iteraciones del patrón Reflection (Reflexión) para generar código correcto, multiplicando el consumo de tokens por ejecución.
- Uso de modelos grandes (ej. GPT-4, Claude) para todas las ejecuciones sin discriminar por complejidad de la fuente.
- Ejecuciones fallidas que se reintentan automáticamente, consumiendo tokens adicionales sin generar salida útil.
- Iteraciones del plan Pre-Act (planificación multietapa) donde el refinamiento continuo (Revise Plan) tras cada observación acumula contexto y tokens con cada ciclo.

## La Restricción

Cada invocación del agente de IA consume tokens facturables: tokens de entrada (el prompt con la muestra de datos, estándares, instrucciones y contexto acumulado del plan Pre-Act) y tokens de salida (el código PySpark generado, el contrato YAML — YAML Ain't Markup Language, el DDL — Data Definition Language, y la documentación). El costo es proporcional al número de tokens procesados y al modelo utilizado.

Estimaciones de consumo por ejecución:

- **Ejecución simple (< 30 columnas, CSV — Comma-Separated Values — estándar):** ~20.000 tokens totales (entrada + salida). Sin Reflection.
- **Ejecución media (30-100 columnas, formatos mixtos):** ~40.000-60.000 tokens. 1-2 ciclos de Reflection.
- **Ejecución compleja (> 100 columnas, JSON anidado):** ~80.000-120.000 tokens. 3+ ciclos de Reflection. Ver [[limits/llm-context-window-limit.md]].

El costo por token varía según el modelo (precios de referencia, sujetos a cambio):

- **SLMs (Small Language Models) con cuantización:** ~$0.10-0.30 USD por millón de tokens.
- **Modelos medianos (Claude Sonnet, GPT-4o-mini):** ~$0.50-3.00 USD por millón de tokens.
- **Modelos grandes (Claude Opus, GPT-4):** ~$10.00-30.00 USD por millón de tokens.

Para un cliente con 20 fuentes nuevas al mes, la proyección mensual es:

- Con modelo grande: 20 fuentes × 60.000 tokens × $15/M tokens = **$18.00 USD/mes** (ejecución media).
- Con modelo grande + Reflection intensivo: 20 fuentes × 120.000 tokens × $15/M tokens = **$36.00 USD/mes**.
- A escala de 10 clientes: **$180-360 USD/mes** solo en tokens de inferencia del agente.

Estos costos parecen manejables individualmente pero se acumulan y, combinados con los costos de AWS Glue (ver [[limits/glue-job-memory-timeout.md]]), pueden representar entre el 5% y el 15% del costo total del proyecto, reduciendo el margen de rentabilidad por debajo del objetivo del 60%.

## Evidencia e Impacto

Si se ignora este límite:

- **Erosión del margen:** El informe técnico del SOPP establece un margen de rentabilidad objetivo > 60% para proyectos desde cero. Si el costo de tokens no se controla, puede reducir el margen al 40-50%, haciendo el proyecto financieramente menos atractivo.
- **Costos ocultos por reintentos:** Las ejecuciones fallidas que se reintentan automáticamente consumen tokens sin generar valor. Un agente que falla 3 veces antes de tener éxito cuadruplica el costo de esa ejecución.
- **Desalineación de pricing con el cliente:** Si el costo de tokens no se incluye en la estructura de precios del servicio, Pragma absorbe el costo como overhead, reduciendo la rentabilidad a medida que el cliente escala su uso del agente.
- **Incentivo perverso a evitar Reflection:** Si el equipo intenta reducir costos limitando los ciclos de Reflection, la calidad del código generado puede degradarse, generando más Toil (trabajo manual) en la revisión humana — exactamente lo que el agente debe eliminar.

## Señales de Alerta

- El costo mensual de tokens por cliente supera el 10% del costo total del proyecto registrado en el JTBD (Job To Be Done) Tracker.
- El número promedio de ciclos de Reflection por ejecución supera 3, indicando que el modelo está generando código de baja calidad que requiere múltiples correcciones.
- El consumo de tokens de una ejecución individual supera los 100.000 tokens, indicando que la fuente es demasiado compleja para una sola invocación.
- La tasa de ejecuciones fallidas supera el 20%, indicando un problema sistemático que multiplica el consumo de tokens sin generar valor.
- La capa Nexo del SOPP reporta un incremento mes a mes > 30% en el consumo total de tokens sin un incremento proporcional en el número de fuentes procesadas.

## Workarounds y Mitigaciones

- **Selección de modelo por complejidad:** Implementar una lógica de routing en el paso de Planificación Inicial (Pre-Act) del agente que seleccione el modelo según la complejidad de la fuente: SLMs con cuantización de 4 bits para fuentes simples (< 30 columnas, formatos estándar), modelos medianos para fuentes de complejidad media, y modelos grandes reservados exclusivamente para fuentes complejas o con JSON anidado. [[references/pre-act-pattern-guide.md]]
- **Caché de esquemas y patrones:** Si el agente ya generó un pipeline exitoso para una fuente con estructura similar, reutilizar el esquema y el código como template en lugar de invocar al LLM desde cero. Almacenar estos templates en la base de conocimiento del chapter.
- **Límite de ciclos de Reflection:** Configurar un máximo de 3 ciclos de Reflection por ejecución. Si después de 3 ciclos el código aún tiene errores, escalar al Data Engineer para intervención manual en lugar de consumir tokens adicionales. [[references/reflection-pattern-guide.md]]
- **Compresión de la muestra de datos:** Enviar un resumen estadístico en lugar de la muestra completa para reducir tokens de entrada. Ver [[limits/llm-context-window-limit.md]] para técnicas de compresión.
- **Presupuesto de tokens por cliente:** Establecer un presupuesto mensual de tokens por cliente como parte del contrato de servicio. Registrar el consumo en la capa Nexo del SOPP y alertar cuando se alcance el 80% del presupuesto.
- **Monitoreo y reporte de costos:** Incluir el costo de tokens como métrica obligatoria en el JTBD Tracker de cada ejecución. Revisar mensualmente la relación costo de tokens vs. horas-hombre ahorradas para validar que el ROI (Return on Investment) se mantiene positivo. [[references/cocomo-roi-estimation-guide.md]]
- **Fine-tuning de SLMs:** Evaluar el fine-tuning de modelos pequeños con datos de ejecuciones exitosas de Pragma para reducir la dependencia de modelos grandes y los costos asociados, manteniendo la calidad del código generado. [[business-solutions/standard/jtbd/ai-agent-bronze-ingestion.md]]
