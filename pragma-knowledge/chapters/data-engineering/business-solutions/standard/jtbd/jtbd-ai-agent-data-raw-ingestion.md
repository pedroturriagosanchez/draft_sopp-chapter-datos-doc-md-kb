# JTBD: Automatización de Ingestión en Capa Bronze mediante Agente de IA con Arquitectura Pre-Act y MLOps 2.0

## Descripción y Dolor

Los equipos de ingeniería de datos de Pragma y sus clientes enfrentan un problema estructural conocido como **Toil** (trabajo manual, repetitivo, automatizable, táctico y sin valor perdurable, según el marco SRE — Site Reliability Engineering). En el contexto de la ingestión Bronze (capa de datos crudos con fidelidad total en la Arquitectura Medallón), el Toil se manifiesta en:

- Creación manual de pipelines de ingesta para cada nueva fuente de datos, con un esfuerzo estimado de 4 a 16 horas por dataset según su complejidad.
- Corrección reactiva de errores de esquema (**Schema Drift**: cambios en la estructura de los archivos entre entregas) que se detectan tarde, cuando ya contaminaron capas superiores (Silver, Gold).
- Codificación repetitiva de validaciones básicas (tipos, nulos, rangos) para cientos de fuentes heterogéneas.
- **Silent Drift**: cambios sutiles en la distribución estadística de los datos (media, varianza, cardinalidad) no detectados por CI/CD (Continuous Integration / Continuous Delivery — Integración Continua / Entrega Continua) estándar, que degradan silenciosamente modelos de ML (Machine Learning) en producción.
- **Engineering Bottleneck**: silos de conocimiento donde solo un ingeniero entiende un script artesanal (_bespoke_), generando un punto único de falla para el equipo.

Este trabajo escala de forma lineal (O(n)) con el crecimiento del volumen de datos y el número de fuentes. En organizaciones que integran 10 o más fuentes nuevas al mes, esto representa hasta **160 horas-hombre/mes** en tareas de bajo valor agregado, convirtiendo a los Data Engineers en "plomeros de archivos" en lugar de arquitectos de valor. El **Time-to-Insight** (tiempo desde la identificación de una fuente de datos hasta su disponibilidad para analítica) se extiende entre 1 y 2 semanas, retrasando la toma de decisiones del negocio.

Este JTBD documenta la capacidad del Chapter de Datos para resolver este problema mediante un **Agente de IA con arquitectura Pre-Act** (planificación multietapa), validación continua de datos bajo el marco **MLOps 2.0** y generación de código alineado con el patrón de **Arquitectura Hexagonal** (Puertos y Adaptadores). El agente opera dentro del **SOPP (Sistema Operativo de Potencia Pragma)** y sus métricas de éxito están registradas en el JTBD Tracker corporativo.

**Metas cuantificadas (JTBD Tracker):**

| KPI | Meta | Método de Medición |
|-----|------|-------------------|
| Eliminación de Toil | > 80% del Toil de ingeniería eliminado | % de fuentes integradas con el agente vs. manualmente |
| Eficiencia Operativa | > 70% reducción en horas-hombre | Comparación de horas registradas antes/después por proyecto |
| Margen de Rentabilidad | > 60% en proyectos desde cero | Modelo COCOMO III (Constructive Cost Model) aplicado al proyecto [[references/cocomo-roi-estimation-guide.md]] |
| SOPP Delivery Time | < 5 días hábiles desde inicio hasta aprobación | Registro en JTBD Tracker |
| Uso Recurrente | ≥ 1 uso por semana por Data Engineer | Métricas de uso del SOPP |

## Contexto

Esta solución aplica a organizaciones que operan un **Data Lakehouse** (convergencia de Data Warehouse y Data Lake) sobre AWS S3 (Simple Storage Service) o almacenamiento equivalente, con **Arquitectura Medallón** (Bronze / Silver / Gold) y motores de procesamiento como AWS Glue, Apache Spark o Delta Live Tables.

Opera dentro del ecosistema **SOPP (Sistema Operativo de Potencia Pragma)**, específicamente en la **capa Nexo** del Data Lake inteligente, que centraliza la información a través de herramientas y agentes con observabilidad total (seguimiento granular de latencias, errores y consumo de tokens de modelos de lenguaje por cliente).

**Perfil de cliente ideal:**

- Organizaciones financieras o fintech con alto volumen de fuentes de datos heterogéneas (transacciones, comportamiento de usuario, datos de comercios, APIs de terceros) que se incorporan con frecuencia al Data Lake.
- Equipos de Data Engineering con backlog creciente de ingesta y presión por reducir el Time-to-Insight.
- Organizaciones que buscan escalar su capacidad de ingesta sin escalar linealmente su equipo técnico.
- Clientes con regulaciones de datos estrictas (sector financiero) que requieren trazabilidad y auditoría desde la capa Bronze.
- Equipos que han sufrido incidentes de Schema Drift o Silent Drift en producción y buscan una solución preventiva.

**Clientes de referencia con arquitectura Medallón activa:**

- **Nequi (Colombia):** Neobank con más de 22 millones de usuarios, propiedad de Bancolombia. Genera múltiples fuentes de datos de alta frecuencia: transacciones, comportamiento de usuario, pagos de servicios y recargas. Requiere ingesta ágil y estandarizada para sostener su crecimiento exponencial de datos.
- **Redeban (Colombia):** Infraestructura crítica de pagos electrónicos a escala nacional. Procesa transacciones de múltiples entidades bancarias y comercios, generando fuentes de datos heterogéneas con alta frecuencia de Schema Drift por incorporación de nuevos comercios y medios de pago.
- **Banesco (Panamá):** Grupo financiero internacional presente en 14 países de Latinoamérica y Europa. Opera como procesador de pagos para fintechs y bancos de la región, con necesidad de integrar fuentes de datos de múltiples sistemas origen con estándares de seguridad y cumplimiento regulatorio estrictos.

**Perfiles afectados:**

- **Data Engineers:** liberados del Toil para enfocarse en arquitectura de valor (capas Silver/Gold).
- **Data Analysts:** acceso a datos frescos en horas en lugar de semanas.
- **Business Owners / CDO (Chief Data Officer):** toma de decisiones basada en datos actualizados y confiables.
- **Equipos de ML (Machine Learning):** modelos entrenados con datos de calidad garantizada desde Bronze, sin Silent Drift.

## Alcance

**Incluye:**

- Reconocimiento automático de formato, encoding y estructura de archivos fuente (CSV, JSON, Parquet, XML, ORC).
- Perfilado estadístico de la muestra: análisis de nulos, cardinalidad, distribución de valores y detección de anomalías antes de la inferencia de esquema.
- Inferencia y normalización del esquema (nombres de columnas, tipos de datos) con mapeo al estándar de la plataforma, evitando el antipatrón de "todo es String".
- Definición de columnas de partición (`partition_by`) y estrategia de escritura (`overwrite` / `append` / `merge`) en el script generado.
- Generación de scripts PySpark modulares bajo **Arquitectura Hexagonal** (Puertos y Adaptadores) con separación de adaptadores de infraestructura y lógica de negocio.
- Generación del DDL (Data Definition Language) para registro en el catálogo de datos (AWS Glue Data Catalog, Unity Catalog o equivalente).
- Implementación de las **4 Puertas de Calidad MLOps 2.0** con niveles de severidad predeterminados:
  - **Estructural (BLOCK):** Cumplimiento de tipos y columnas contra el contrato de datos YAML.
  - **Semántica (WARN / BLOCK):** Reglas de negocio del dominio (fechas no futuras, precios no negativos, IDs únicos).
  - **Temporal (AUDIT):** Verificación de orden cronológico, frescura de datos (T < 24h) y detección de temporal leakage (filtración de información futura hacia el pasado).
  - **Distribucional (WARN):** Comparación de estadísticas actuales contra línea base histórica mediante métricas KS (Kolmogorov-Smirnov) o PSI (Population Stability Index) para detectar Silent Drift.
- Generación de contrato de datos en formato YAML con tipos, restricciones y metadatos.
- Columnas de auditoría estándar SOPP: `_ingestion_timestamp`, `_source_file`, `_batch_id`, `_schema_version`.
- Integración obligatoria con librerías corporativas de Logs (registros de ejecución) y Auditoría en todos los scripts generados.
- Auto-documentación del dataset: Data Dictionary en Markdown con descripción de columnas, tipos, origen, frecuencia estimada de actualización y clasificación de PII (Personally Identifiable Information — Información de Identificación Personal).
- Suite de Tests de Calidad con umbrales de aceptación basados en DQSOps (Data Quality Scoring Operations).
- Validación automática del código generado mediante análisis de logprobs (probabilidades logarítmicas de tokens) y ciclo de Reflexión (Reflection) antes de la entrega.
- Soporte para configuración de seguridad específica del cliente (KMS keys — Key Management Service, VPC endpoints — Virtual Private Cloud) documentada antes de la generación del script.

**No incluye:**

- Transformaciones de negocio hacia capas Silver o Gold (responsabilidad del equipo de Analytics Engineering).
- Definición de reglas de calidad de datos complejas del dominio de negocio (responsabilidad del Chapter de QA o Data Governance).
- Despliegue autónomo a producción sin aprobación humana explícita.
- Manejo de fuentes en tiempo real / streaming (aplica para un JTBD separado de streaming CDC — Change Data Capture).
- Modelado dimensional o diseño de Data Warehouse.

## Cuándo aplica

- Cuando el equipo recibe una nueva fuente de datos en S3 (CSV, JSON, XML, Parquet, ORC) que debe incorporarse al catálogo Bronze.
- Cuando hay un backlog de ingesta que retrasa proyectos analíticos o de ML.
- Cuando se requiere estandarizar la calidad y nomenclatura de los esquemas Bronze en toda la organización.
- Cuando el cliente ha sufrido incidentes de Schema Drift o Silent Drift en producción y necesita una solución preventiva.
- Cuando se quiere demostrar ROI (Return on Investment) de IA aplicada a ingeniería de datos en menos de 5 días hábiles (caso piloto de alto impacto).
- Cuando el equipo de Data Engineering dedica más del 40% de su tiempo a tareas de ingesta manual.

## Cuándo no aplica

- Fuentes de datos en tiempo real con latencia sub-segundo (usar arquitecturas de streaming con Apache Kafka / Apache Flink).
- Archivos con estructuras extremadamente anidadas (> 3 niveles de profundidad) sin posibilidad de pre-procesamiento que requieran modelado de dominio profundo.
- Clientes sin equipo técnico in-house que pueda revisar y aprobar el código generado antes de producción.
- Cuando el cliente ya tiene un proceso de ingesta automatizado y maduro con cobertura > 90% (el valor agregado sería marginal).
- Fuentes que contengan datos PII (Personally Identifiable Information) sin un proceso de anonimización o enmascaramiento previo aprobado por el área de seguridad del cliente.
- Cuando los requisitos de seguridad del cliente no permiten que el agente acceda a datos reales (en este caso, evaluar el uso de datos sintéticos como alternativa).

## Entradas (Inputs)

### Entradas Transversales (Estándares SOPP)

- **Estándares de Arquitectura Hexagonal:** Guías de Puertos y Adaptadores para PySpark/Glue. [[references/hexagonal-architecture-guide.md]]
- **Guías de Clean Code:** Convenciones de nomenclatura, estructura de módulos y documentación del SOPP.
- **Convenciones de Nomenclatura SOPP:** Formato estricto `[environment]-[project]-[layer]-[source]` que permite al agente identificar el contexto de ejecución (Desarrollo, Pruebas, Producción) y la ubicación lógica del dato en las capas de arquitectura (Bronze, Silver, Gold). [[references/sopp-naming-conventions.md]]
- **Plantillas de Contrato de Datos YAML:** Estructura estándar para definición formal de esquemas entre el productor y el consumidor del dato.

### Entradas Específicas por Ejecución

- **URI (Uniform Resource Identifier) de S3 (Source Path):** Ruta al archivo o prefijo del bucket donde residen los datos fuente.
- **Muestra representativa de datos:** Mínimo 100 filas del dataset en formato JSON, CSV o Parquet para análisis de estructura y perfilado estadístico inicial (distribución de valores, nulos, cardinalidad).
- **Metadatos de origen:** Swagger (especificación de API), DDL u otro contrato técnico que describa la fuente.
- **Contexto de negocio:** Descripción del contenido del dataset, frecuencia de actualización y uso previsto en capas superiores.
- **Patrón de particionamiento esperado:** Columnas y criterio de partición deseado (ej. `partition_by=year/month/day`, `partition_by=region`) para optimizar consultas en la capa Bronze.
- **Estrategia de escritura:** Modo de carga definido por el cliente: `overwrite` (reemplazo total), `append` (acumulación) o `merge` (upsert por clave natural).
- **Estándares de nomenclatura del cliente:** Convenciones propias de nombres de tablas y columnas, si existen (complementarias a las convenciones SOPP).
- **Permisos IAM (Identity and Access Management):** Credenciales de solo lectura al bucket S3 (`s3:GetObject`, `s3:ListBucket`).
- **Requisitos de seguridad:** Nivel de encriptación requerido (SSE-S3 — Server-Side Encryption con claves gestionadas por S3, o KMS CMK — Customer Master Key), regulaciones aplicables (PCI DSS — Payment Card Industry Data Security Standard, GDPR — General Data Protection Regulation, HIPAA — Health Insurance Portability and Accountability Act) y configuración de VPC endpoints si aplica.

## Flujo de alto nivel

### Arquitectura del Agente: Pre-Act (Planificación Multietapa)

A diferencia del paradigma **ReAct** (Reasoning + Acting — razonamiento reactivo paso a paso), este agente opera bajo el marco **Pre-Act**, que genera un plan integral de ejecución multietapa de n+1 pasos desde la primera interacción. Pre-Act supera las limitaciones del ReAct puro, que puede ser "miope" al centrarse solo en el siguiente paso inmediato sin visión de la complejidad total del pipeline de datos.

| Dimensión | ReAct (Iterativo) | Pre-Act (Planificado) |
|-----------|-------------------|----------------------|
| Génesis del Plan | Reactivo al paso anterior | Plan integral de n+1 pasos desde el inicio |
| Manejo de Contexto | Memoria de trabajo volátil | Incorporación incremental de pasos pasados y futuros |
| Robustez en Ingeniería | Media (puede entrar en bucles) | Alta (refinamiento continuo del plan maestro) |
| Optimización de Modelo | Basado en prompts extensos | Basado en fine-tuning con datasets de planificación |

Pre-Act permite que modelos más pequeños (ej. Llama 3.1 de 8 mil millones de parámetros con fine-tuning orientado a planificación) alcancen una precisión de acción y una tasa de finalización de tareas comparables a modelos gigantescos. [[references/pre-act-pattern-guide.md]]

### Interacción del Agente con Herramientas: Model Context Protocol (MCP)

El agente interactúa con herramientas externas (S3, AWS Glue, catálogos de datos, sistemas de orquestación) a través del **MCP (Model Context Protocol)**, una capa de abstracción que estandariza el acceso a herramientas y refuerza el desacoplamiento de la Arquitectura Hexagonal. MCP asegura que la lógica central del agente permanezca "limpia" de detalles técnicos de implementación de bajo nivel, permitiendo que las herramientas de destino se intercambien sin reescribir la lógica de orquestación.

### Pasos del Flujo

1. **Planificación Inicial (Pre-Act):** El agente recibe todas las entradas (URI, muestra, contexto de negocio, estándares) y genera un plan de ejecución completo con n+1 pasos secuenciales, estimando la complejidad de cada fase antes de ejecutar ninguna acción. Este plan incluye el contexto de pasos previos vacío (primera ejecución) y la hoja de ruta de pasos siguientes.

2. **Reconocimiento de Fuente:** El agente lee la muestra del archivo vía MCP y genera el Perfil Técnico: formato (CSV, JSON, Parquet, XML, ORC), encoding (UTF-8, Latin-1, etc.), delimitadores, estructura de particionamiento existente y nivel de anidamiento.

3. **Perfilado Estadístico:** El agente analiza la muestra para calcular métricas de calidad base: porcentaje de nulos por columna, cardinalidad, rango de valores numéricos y detección de outliers. Este perfilado determina la estrategia de tipado y los umbrales iniciales de la Suite de Tests de Calidad.

4. **Inferencia y Normalización de Esquema:** Propone nombres de columnas normalizados (sin espacios, tildes ni caracteres especiales) y mapea los tipos de datos al estándar de la plataforma, evitando el antipatrón de "todo es String". Define columnas de partición y estrategia de escritura. Genera el contrato de datos en YAML.

5. **Refinamiento del Plan (Revise Plan):** Tras cada observación, el agente reevalúa y actualiza todo el plan de pasos siguientes, adaptándose a hallazgos inesperados (ej. archivos más complejos de lo estimado, encoding mixto, campos con anomalías). Si detecta inconsistencias (ej. campo numérico con caracteres especiales en el 1% de registros), ajusta la estrategia de generación antes de continuar.

6. **Generación de Artefactos bajo Arquitectura Hexagonal:**
   - **Script PySpark modular:** Con separación de adaptadores de infraestructura (Adaptadores Primarios — Drivers: Glue Jobs, EventBridge — bus de eventos de AWS, Lambda; Adaptadores Secundarios — Driven: S3, RDS — Relational Database Service, SNS/SQS — servicios de mensajería de AWS) y lógica de negocio (transformación de datos en el centro del hexágono).
   - **DDL para el catálogo de datos:** Con metadatos completos y tags de clasificación.
   - **Columnas de auditoría estándar SOPP:** `_ingestion_timestamp`, `_source_file`, `_batch_id`, `_schema_version`.

7. **Implementación de las 4 Puertas de Calidad MLOps 2.0:**
   - **Estructural (Severidad: BLOCK):** Valida que los datos cumplan con los tipos y columnas definidos en el contrato YAML. Si falla, el pipeline se detiene para evitar que datos corruptos lleguen a capas superiores.
   - **Semántica (Severidad: WARN / BLOCK):** Valida reglas de negocio del dominio: fechas no futuras, precios no negativos, IDs únicos, límites de crédito lógicos. BLOCK para violaciones críticas, WARN para las demás.
   - **Temporal (Severidad: AUDIT):** Verifica orden cronológico, frescura de datos (T < 24h desde la generación del evento) y detección de temporal leakage. Se registran métricas para tableros de gobernanza sin detener el flujo.
   - **Distribucional (Severidad: WARN):** Compara estadísticas actuales (media, varianza, cardinalidad) contra una línea base histórica mediante métricas KS (Kolmogorov-Smirnov) o PSI (Population Stability Index). Si la distribución cambia drásticamente, genera una alerta de Silent Drift.

8. **Validación Sintáctica con Logprobs (Detección de Alucinaciones):** El agente analiza las probabilidades logarítmicas de los tokens generados por el modelo para identificar segmentos de baja confianza que podrían indicar una alucinación o un error técnico. Si detecta incertidumbre superior al umbral configurado, activa automáticamente un ciclo de **Reflexión (Reflection)** para corregir su propia salida antes de la entrega. [[references/reflection-pattern-guide.md]]

9. **Auto-Documentación:** Genera el Data Dictionary en Markdown con descripción de cada columna, tipo de dato, origen, frecuencia estimada de actualización y clasificación de PII si aplica. Genera la Suite de Tests de Calidad con umbrales configurables basados en DQSOps.

10. **Revisión y Aprobación Humana:** El Data Engineer responsable revisa todos los artefactos generados (esquema, scripts, DDL, tests, documentación). Ejecuta el pipeline en un entorno de staging y aprueba formalmente antes del despliegue a producción. **El código generado NO debe ejecutarse en producción sin esta aprobación explícita.**

## Participación del chapter

El **Chapter de Datos** lidera de principio a fin esta solución con los siguientes roles:

- **Data Engineer Senior / Tech Lead de Datos:** Lidera el diseño de la arquitectura del agente y los estándares Bronze aplicables a todos los clientes. Define las convenciones de nomenclatura, tipos de datos y columnas de auditoría. Valida y aprueba los artefactos generados antes de la primera ejecución en producción. Es el punto de escalamiento ante casos de inferencia compleja (archivos altamente anidados, Schema Drift severo).

- **Data Engineer:** Ejecuta el agente para cada nueva fuente de datos asignada. Revisa el código PySpark/Glue generado, ajusta los parámetros de inferencia cuando el agente requiere orientación y aprueba el esquema Bronze propuesto. Realiza la primera ejecución supervisada en el entorno del cliente.

- **AI/ML Engineer (IA Enablement):** Mantiene y optimiza los prompts de inferencia del agente y el plan Pre-Act. Implementa y evoluciona los patrones de orquestación (Pre-Act) y validación automática (Reflection con logprobs). Gestiona el ciclo de vida del modelo subyacente, evalúa el uso de SLMs (Small Language Models) con cuantización de 4 bits para operaciones a gran escala, y adapta el agente ante cambios en las plataformas de destino (AWS Glue, Databricks, Snowflake).

- **Data Steward / Analista de Calidad de Datos:** Valida que los umbrales de calidad definidos en las 4 Puertas de MLOps 2.0 sean correctos y representativos para cada fuente de datos. Revisa y aprueba los diccionarios de datos generados. Es el interlocutor con el Product Owner del cliente para alinear expectativas sobre la calidad del dato en la capa Bronze.

- **Consultor de Datos:** Acompaña al cliente en la identificación de fuentes prioritarias, la definición de requerimientos de negocio y la comunicación del valor generado. Gestiona el JTBD Tracker y documenta el ROI de cada ejecución siguiendo el modelo COCOMO III. [[references/cocomo-roi-estimation-guide.md]]

## Dependencias

- **Chapter de Cloud / Infraestructura:** Permisos IAM de solo lectura al bucket S3 del cliente, configuración de VPC Endpoints y roles de AWS Glue. Sin este acceso, el agente no puede operar. Punto de interacción: Fase 0 (prerrequisitos de seguridad).
- **Chapter de QA / Data Governance:** Define las reglas de calidad semántica específicas del dominio de negocio del cliente que alimentan la Puerta Semántica de MLOps 2.0. Punto de interacción: Paso 7 del flujo (puertas de calidad).
- **Chapter de Architecture:** Valida que el patrón de ingesta generado sea coherente con la arquitectura Medallón (Bronze / Silver / Gold) definida para el cliente y el uso correcto del patrón Hexagonal. Punto de interacción: revisión del DDL y estructura de particionamiento.
- **Product Owners / Data Stewards del cliente:** Validación y aprobación del esquema propuesto antes de la primera ejecución en producción.
- **Catálogo de Datos del Cliente:** AWS Glue Data Catalog, Unity Catalog (Databricks) o equivalente activo y accesible.
- **Herramienta de Orquestación:** AWS Glue, Apache Airflow, Databricks Workflows o equivalente activo en el entorno del cliente.
- **Framework de Validación:** AWS Deequ o Great Expectations para las puertas de calidad distribucionales.
- **Model Context Protocol (MCP):** Capa de abstracción que estandariza cómo el agente accede a herramientas externas (S3, Glue, catálogos), asegurando el desacoplamiento de la Arquitectura Hexagonal.
- **Framework DQSOps (Data Quality Scoring Operations):** Sistema de calidad corporativo para definir y monitorear umbrales de aceptación del dato. [[references/dqsops-framework.md]]
- **Decisión de Estándares Bronze — Arquitectura Medallón:** Convenciones de nombres, tipos, columnas de auditoría y patrón base de organización del Data Lakehouse sobre el que opera este JTBD. [[decisions/001-bronze-layer-standards.md]]
- **Decisión de Arquitectura Hexagonal:** Patrón de diseño requerido para la generación de scripts modulares con separación Adapter/Lógica. [[decisions/002-hexagonal-architecture.md]]
- **Guía Procedimental de Ejecución del Agente:** Referencia paso a paso para la ejecución del agente en entornos de cliente. [[references/bronze-ingestion-procedural-guide.md]]
- **Convenciones de Nomenclatura SOPP:** Formato `[environment]-[project]-[layer]-[source]`. [[references/sopp-naming-conventions.md]]

## Salidas esperadas (Outputs)

- **Scripts PySpark / Glue modulares:** Pipelines con separación de Adaptadores (Primarios y Secundarios) y Lógica de negocio bajo Arquitectura Hexagonal, listos para revisión y despliegue. [[outputs/bronze-ingestion-script.md]]
- **Contrato de Datos YAML:** Definición formal del esquema Bronze con tipos, restricciones, metadatos y tags de clasificación.
- **DDL del Catálogo:** Script ejecutable para crear la tabla en el catálogo de datos (AWS Glue Data Catalog, Unity Catalog o equivalente) con metadatos completos.
- **Data Dictionary:** Documento Markdown con descripción de cada columna, tipo de dato, origen, frecuencia estimada de actualización y clasificación de PII. [[outputs/data-dictionary.md]]
- **Reporte de Validaciones MLOps 2.0:** Resultado de las 4 Puertas de Calidad con severidades aplicadas (BLOCK / WARN / AUDIT), métricas KS/PSI y recomendaciones de acción.
- **Suite de Tests de Calidad:** Conjunto de pruebas automatizadas con umbrales configurables para validación preventiva basada en DQSOps, reutilizables en ejecuciones futuras del pipeline. [[outputs/data-quality-test-suite.md]]
- **Documentación técnica del esquema:** Especificación del esquema Bronze normalizado con tipos de datos, columnas de auditoría estándar (`_ingestion_timestamp`, `_source_file`, `_batch_id`, `_schema_version`) y estructura de particionamiento.

## Riesgos y consideraciones

- **Schema Drift:** Archivos cuya estructura cambia entre entregas (columnas nuevas, tipos modificados, columnas eliminadas) pueden romper el pipeline generado. Mitigación: la Puerta Estructural (BLOCK) de MLOps 2.0 valida el esquema contra el contrato YAML en cada ejecución y detiene el pipeline ante violaciones críticas. Ver [[limits/schema-drift-and-known-failures.md]].
- **Silent Drift:** Cambios sutiles en la distribución estadística de los datos no detectados por CI/CD estándar. Puede degradar modelos de ML hasta un 30% antes de ser detectado manualmente. Mitigación: la Puerta Distribucional (WARN) monitorea con métricas KS/PSI en cada carga y genera alertas automáticas.
- **Alucinaciones del LLM (Large Language Model — Modelo de Lenguaje Grande):** El modelo puede generar código sintácticamente correcto pero lógicamente incorrecto. Mitigación: validación con logprobs para detectar segmentos de baja confianza, seguida de un ciclo de Reflexión (Reflection) automático, más la aprobación humana obligatoria antes de producción.
- **Encoding inconsistente:** Archivos legacy que mezclan encodings dentro del mismo fichero generan errores silenciosos de lectura. Mitigación: detección de encoding a nivel de muestra estadística (no solo del primer registro) en el paso de Reconocimiento de Fuente.
- **Volúmenes extremos en muestreo:** Si el archivo fuente supera el umbral de muestreo del agente, la muestra puede no ser representativa del esquema real. Mitigación: definir un tamaño de muestra mínimo garantizado (100 filas) y alertar cuando el archivo supere el umbral configurable.
- **Archivos altamente anidados:** JSON o XML con jerarquías > 3 niveles de profundidad aumentan la complejidad de inferencia y requieren intervención manual del Data Engineer Senior. Ver [[limits/schema-drift-and-known-failures.md]].
- **Dependencia de permisos IAM:** Sin acceso de lectura al bucket S3 del cliente, el agente no puede operar. Este prerrequisito debe gestionarse con el Chapter de Cloud antes del inicio del JTBD.
- **Vendor Lock-in:** El uso de servicios específicos de AWS (Glue, Lake Formation) puede generar dependencia de proveedor. Mitigación: la Arquitectura Hexagonal garantiza que la lógica de negocio sea portable entre plataformas (AWS, Databricks, Snowflake) mediante el intercambio de adaptadores sin reescribir el núcleo.
- **Requisitos de seguridad del cliente:** Clientes regulados (PCI DSS, HIPAA, GDPR) requieren configuraciones de seguridad adicionales (KMS CMK, VPC Endpoints, pistas de auditoría CloudTrail — servicio de registro de actividad de AWS) no generadas automáticamente por el agente. Estas deben documentarse como entradas antes de la generación del script. Ver [[limits/schema-drift-and-known-failures.md]].
- **Aprobación humana obligatoria:** El código generado NO debe ejecutarse en producción sin revisión explícita del Data Engineer responsable. Este es un principio innegociable del JTBD.

## Variantes comunes

- **Versión PoC (Proof of Concept) / Hackathon:** Alcance reducido a inferencia de esquema y generación de DDL, sin script de ingesta completo ni puertas de calidad. Ideal para demostrar valor de IA aplicada en 1-2 días. Prioridad 1 en el JTBD Tracker.
- **Versión para clientes regulados (Banking / Health):** Incluye configuración de KMS CMK, VPC Endpoints, pistas de auditoría CloudTrail y cumplimiento de regulaciones (PCI DSS, GDPR, HIPAA). Mayor tiempo de configuración (hasta 2 días adicionales). Prioridad 1.
- **Traducción de ETLs (Extract, Transform, Load) Legacy (SQL → PySpark):** Variante orientada a migración de Stored Procedures y scripts SQL On-Premise hacia PySpark bajo Arquitectura Hexagonal. Mayor complejidad, menor frecuencia. Prioridad 2.
- **Versión con Streaming CDC:** Extensión del agente para fuentes en tiempo real usando Apache Kafka / Apache Flink como protocolo estándar y generación de conectores CDC (Change Data Capture) automáticos. Prioridad 2.
- **Versión Data Mesh:** Adaptación para entornos donde cada unidad de negocio es "dueña" de sus datos, con el agente como facilitador de "Data Contracts" entre dominios bajo gobernanza descentralizada. Prioridad 3.
- **Agente de Code Review para Pipelines:** Variante de análisis estático y seguridad sobre Pull Requests de código de datos. Complementario a este JTBD. Prioridad 3.

## Relación con otros chapters

- **Chapter de Cloud:** Configura permisos IAM, roles de Glue y VPC Endpoints. Punto de interacción: Fase 0 (prerrequisitos de seguridad) y revisión de configuraciones para clientes regulados.
- **Chapter de QA / Data Governance:** Define las reglas de calidad semántica de las Puertas de Calidad MLOps 2.0 específicas del dominio de negocio del cliente. Punto de interacción: diseño de validaciones semánticas y distribucionales.
- **Chapter de Architecture:** Valida la coherencia con la arquitectura Medallón del cliente, el uso correcto del patrón Hexagonal y la integración con MCP. Punto de interacción: revisión del DDL, estructura del pipeline y configuración de adaptadores.

**Nota:** Las especialidades internas del Chapter de Datos que consumen los datos Bronze generados por este JTBD (Científicos de Datos para entrenamiento de modelos de ML, Analistas de Datos para analítica descriptiva y dashboards) están documentadas en la sección "Perfiles afectados" y "Participación del chapter" de este mismo documento.

## Referencias relacionadas

- Límites conocidos del agente (Schema Drift, archivos anidados, seguridad): [[limits/schema-drift-and-known-failures.md]]
- Guía procedimental de ejecución del agente: [[references/bronze-ingestion-procedural-guide.md]]
- Guía de Arquitectura Hexagonal para PySpark/Glue: [[references/hexagonal-architecture-guide.md]]
- Guía del Patrón Pre-Act (Planificación Multietapa): [[references/pre-act-pattern-guide.md]]
- Guía del Patrón Reflection (Reflexión y Validación Automática): [[references/reflection-pattern-guide.md]]
- Framework DQSOps (Data Quality Scoring Operations): [[references/dqsops-framework.md]]
- Convenciones de Nomenclatura SOPP: [[references/sopp-naming-conventions.md]]
- Guía de estimación económica COCOMO III y ROI: [[references/cocomo-roi-estimation-guide.md]]
- Tendencias 2026 en ingeniería de datos: [[references/data-engineering-trends-2026.md]]
- Decisión de Estándares Bronze — Arquitectura Medallón: [[decisions/001-bronze-layer-standards.md]]
- Decisión de Arquitectura Hexagonal: [[decisions/002-hexagonal-architecture.md]]
- Output — Script de Ingesta Bronze: [[outputs/bronze-ingestion-script.md]]
- Output — Data Dictionary: [[outputs/data-dictionary.md]]
- Output — Suite de Tests de Calidad: [[outputs/data-quality-test-suite.md]]

## Notas
- Roles y permisos IAM
- Política de ciclo de vida y retención de datos
- Encriptación de datos en bronze (Reposo y tránsito)
- Definir estrategia de auditoría operacional
- Manejo de formatos de tablas abiertas a partir de bronze
- Estructura de manejo de Arquitectura hexagonal
- Estructura de Seguridad (Plano de datos e infra)
