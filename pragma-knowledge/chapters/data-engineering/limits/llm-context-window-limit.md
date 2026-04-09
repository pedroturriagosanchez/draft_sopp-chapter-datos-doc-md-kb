# Límite: Ventana de Contexto del LLM en el Agente de Ingesta

## Contexto de Identificación

Identificado en el desarrollo y operación del Agente de IA para ingesta Bronze dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma). El límite se manifiesta cuando el agente necesita procesar simultáneamente múltiples artefactos de entrada que en conjunto superan la ventana de contexto del LLM (Large Language Model — Modelo de Lenguaje Grande) subyacente.

El problema se presenta con mayor frecuencia en los siguientes escenarios:

- Fuentes de datos con más de 100 columnas donde el esquema completo, la muestra de datos y las convenciones SOPP exceden la capacidad del modelo.
- Ejecuciones donde el agente necesita mantener en contexto el plan Pre-Act (planificación multietapa), el esquema inferido, el contrato YAML (YAML Ain't Markup Language), el código PySpark generado y los resultados de las Puertas de Calidad de MLOps 2.0 (Machine Learning Operations — Operaciones de Aprendizaje Automático) simultáneamente.
- Archivos con tipos de datos complejos donde la descripción de cada columna requiere anotaciones extensas (ej. formatos de fecha regionales, monedas con precisión decimal variable, campos con lógica de enmascaramiento PII — Personally Identifiable Information).
- Iteraciones del patrón Reflection (Reflexión) donde el agente revisa su propio código, lo que duplica el uso de tokens al incluir tanto la salida original como la corrección.

Los modelos actuales tienen ventanas de contexto que varían entre 8K y 200K tokens, pero el costo por token escala proporcionalmente y los modelos más pequeños (SLMs — Small Language Models) optimizados para costo tienen ventanas de 8K a 32K tokens.

## La Restricción

Todo LLM opera con una **ventana de contexto finita**: el número máximo de tokens (unidades de texto, aproximadamente 0.75 palabras por token en español) que puede procesar como entrada más la salida generada. Cuando la suma de todos los artefactos de entrada supera esta ventana, el modelo se comporta de manera impredecible:

- **Truncamiento de instrucciones iniciales:** El modelo "olvida" las convenciones de nomenclatura SOPP o los estándares de Arquitectura Hexagonal (Puertos y Adaptadores) que se proporcionaron al inicio del prompt, generando código que no cumple los estándares.
- **Truncamiento de la salida:** El código PySpark generado se corta abruptamente antes de completar todas las funciones requeridas, entregando un script parcial que no compila.
- **Degradación de precisión en pasos finales:** En un plan Pre-Act de 10 pasos, los pasos 8-10 tienen menor precisión que los pasos 1-3 porque el modelo ha acumulado contexto y la capacidad de atención se diluye.
- **Alucinaciones por contexto insuficiente:** Si la muestra de datos se trunca para caber en la ventana de contexto, el modelo infiere tipos de datos basándose en información parcial, produciendo esquemas incorrectos.

La estimación de consumo de tokens por artefacto es aproximadamente:

- Convenciones SOPP y estándares: ~2.000 tokens.
- Muestra de datos (100 filas × 50 columnas): ~8.000-15.000 tokens.
- Contrato YAML de esquema: ~1.500-3.000 tokens.
- Plan Pre-Act con historial de pasos: ~2.000-5.000 tokens.
- Código PySpark generado: ~3.000-8.000 tokens.
- Total estimado por ejecución: ~16.500-33.000 tokens sin contar Reflection.

## Evidencia e Impacto

Si se ignora este límite:

- **Código incompleto:** El agente entrega scripts PySpark que se cortan a la mitad de una función, requiriendo intervención manual del Data Engineer para completarlos.
- **Violación de estándares:** El código generado no sigue las convenciones de nomenclatura SOPP ni la estructura de Arquitectura Hexagonal porque las instrucciones se perdieron por truncamiento.
- **Esquemas incorrectos:** La inferencia de tipos se basa en una muestra truncada, generando columnas con tipos incorrectos (ej. un campo numérico clasificado como `string` porque la muestra truncada solo contenía valores alfanuméricos atípicos).
- **Ciclos de Reflection infinitos:** Si el código generado tiene errores por contexto insuficiente, el ciclo de Reflection intenta corregirlo pero agrega más tokens al contexto, agravando el problema.
- **Costos inesperados:** Usar modelos con ventanas más grandes (128K-200K tokens) resuelve el truncamiento pero multiplica el costo por ejecución, erosionando el margen de rentabilidad del proyecto. Ver [[limits/llm-token-cost-scaling.md]].

## Señales de Alerta

- El código PySpark generado termina abruptamente sin cierre de funciones o sin la sección de adaptadores de la Arquitectura Hexagonal.
- El agente omite columnas de auditoría estándar (`_ingestion_timestamp`, `_source_file`, `_batch_id`, `_schema_version`) que estaban definidas en las instrucciones iniciales.
- El Data Dictionary generado tiene descripciones genéricas o vacías para las últimas columnas del esquema.
- La API (Application Programming Interface) del modelo retorna un código de error `context_length_exceeded` o un warning de `max_tokens reached`.
- El tiempo de respuesta del agente se incrementa significativamente (> 60 segundos) respecto a ejecuciones previas con fuentes más pequeñas.

## Workarounds y Mitigaciones

- **Segmentación del prompt en fases:** Dividir la ejecución del agente en fases independientes que no requieran mantener todo el contexto simultáneamente. Fase 1: inferencia de esquema (muestra + convenciones). Fase 2: generación de código (esquema inferido + estándares de Arquitectura Hexagonal). Fase 3: validación y documentación (código generado + criterios de calidad). Cada fase recibe solo el contexto necesario.
- **Compresión de la muestra de datos:** En lugar de enviar 100 filas completas, enviar un resumen estadístico de cada columna (tipo inferido, % nulos, cardinalidad, min/max, primeros 5 valores únicos). Esto reduce el consumo de ~15.000 tokens a ~3.000 tokens manteniendo la información necesaria para la inferencia.
- **Límite de columnas por ejecución:** Si la fuente tiene más de 100 columnas, dividir la inferencia en lotes de 50 columnas y consolidar los resultados. Documentar este umbral en la guía procedimental del agente. [[references/bronze-ingestion-procedural-guide.md]]
- **Selección del modelo por complejidad:** Usar SLMs con cuantización de 4 bits para fuentes simples (< 30 columnas, formatos estándar) y reservar modelos con ventanas grandes para fuentes complejas (> 100 columnas, JSON (JavaScript Object Notation) anidado). La selección puede automatizarse en el paso 1 del plan Pre-Act basándose en el perfil técnico de la fuente. [[references/pre-act-pattern-guide.md]]
- **Caché de estándares:** Almacenar las convenciones SOPP y los estándares de Arquitectura Hexagonal como instrucciones del sistema (system prompt) que el modelo procesa con prioridad, evitando que sean desplazadas por el contexto de datos.
- **Monitoreo de tokens por ejecución:** Registrar el consumo de tokens de entrada y salida en cada ejecución dentro de la capa Nexo del SOPP. Configurar alertas cuando una ejecución supere el 80% de la ventana de contexto del modelo utilizado.
