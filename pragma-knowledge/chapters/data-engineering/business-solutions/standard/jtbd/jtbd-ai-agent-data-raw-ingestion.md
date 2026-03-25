# JTBD: Automatización de Ingestión en Capa Cruda mediante Agentes de IA

## Descripción y Dolor

Los equipos de ingeniería de datos sufren un "Toil" estructural (trabajo manual, repetitivo y automatizable que no aporta valor creativo incremental, según el marco SRE — Site Reliability Engineering). Se manifiesta en la escritura de código boilerplate para lectura de archivos, manejo de excepciones de tipos de datos y configuración de particionamiento. En organizaciones que integran 10 o más fuentes nuevas al mes, esto representa hasta **160 horas-hombre/mes** de bajo valor agregado, convirtiendo a los Data Engineers en "plomeros de archivos" en lugar de arquitectos de valor.

Este JTBD automatiza la fase de captura de datos crudos hacia la capa Bronze del Data Lake (capa de datos crudos con fidelidad total en la Arquitectura Medallón), eliminando el **60% del trabajo repetitivo de ingeniería** mediante la inferencia de esquemas y generación de código en herramientas como Spark/Glue. Asegura calidad preventiva (umbrales de calidad) y trazabilidad de las fuentes origen, acelerando el "Time-to-Insight" (tiempo desde la generación del evento hasta su disponibilidad para análisis) global.

**Perfiles afectados:**

- **Data Engineers:** sobrecarga operativa en tareas repetitivas de ingestión.
- **Data Analysts:** espera prolongada para acceder a datos frescos.
- **Business Owners / CDO (Chief Data Officer):** toma de decisiones lenta por falta de datos disponibles en tiempo.

## Contexto

Esta solución aplica a organizaciones del sector financiero y fintech en etapa de crecimiento o escala de su plataforma de datos, que operan arquitecturas Medallón sobre S3 (Simple Storage Service de AWS, o almacenamiento equivalente) y utilizan motores de procesamiento como AWS Glue, Apache Spark o Delta Live Tables.

Opera dentro del ecosistema **SOPP (Sistema Operativo Potencia Pragma)**, en la capa Nexo del Data Lake inteligente, que centraliza la información a través de herramientas y agentes con observabilidad total.

**Perfil de cliente ideal:**

- Organizaciones financieras o fintech con alto volumen de fuentes de datos heterogéneas (transacciones, comportamiento de usuario, datos de comercios, APIs de terceros) que se incorporan con frecuencia al Data Lake.
- Equipos de Data Engineering con backlog creciente de ingestión y presión por reducir el Time-to-Insight.
- Organizaciones que buscan escalar su capacidad de ingestión sin escalar linealmente el equipo técnico.
- Clientes con regulaciones de datos estrictas (sector financiero) que requieren trazabilidad y auditoría desde la capa Bronze.

**Clientes de referencia con arquitectura Medallón activa:**

- **Nequi (Colombia):** Neobank con más de 22 millones de usuarios, propiedad de Bancolombia. Genera múltiples fuentes de datos de alta frecuencia: transacciones, comportamiento de usuario, pagos de servicios y recargas. Requiere ingestión ágil y estandarizada para sostener su crecimiento exponencial de datos.
- **Redeban (Colombia):** Infraestructura crítica de pagos electrónicos a escala nacional. Procesa transacciones de múltiples entidades bancarias y comercios, generando fuentes de datos heterogéneas con alta frecuencia de cambio de esquema (Schema Drift) por incorporación de nuevos comercios y medios de pago.
- **Banesco (Panamá):** Grupo financiero internacional presente en 14 países de Latinoamérica y Europa. Opera como procesador de pagos para fintechs y bancos de la región, con necesidad de integrar fuentes de datos de múltiples sistemas origen con estándares de seguridad y cumplimiento regulatorio estrictos.

## Alcance

**Incluye:**

- Detección automática de formato, encoding y estructura del archivo fuente.
- Perfilado estadístico de la muestra: análisis de nulos, cardinalidad, distribución de valores y detección de anomalías antes de la inferencia.
- Inferencia y normalización del esquema (nombres de columnas, tipos de datos).
- Definición de columnas de partición (`partition_by`) y estrategia de escritura (overwrite / append / merge) en el script generado.
- Generación de scripts modulares (PySpark / Glue / Lambda) con separación Adapter/Lógica bajo Arquitectura Hexagonal.
- Integración obligatoria con librerías corporativas de Logs (registros de ejecución) y Auditoría en todos los scripts generados.
- Generación del DDL (Data Definition Language) para registro en el catálogo de datos.
- Suite de Tests de Calidad con umbrales de aceptación basados en DQSOps.
- Auto-documentación del dataset (Data Dictionary en Markdown).
- Validación automática del código generado (Patrón Reflection) antes de la entrega.
- Soporte para configuración de seguridad específica del cliente (KMS keys — Key Management Service, VPC endpoints — Virtual Private Cloud) documentada antes de la generación del script.

**No incluye:**

- Transformaciones de negocio hacia capas Silver o Gold (responsabilidad del equipo de Analytics Engineering).
- Definición de reglas de calidad de datos (responsabilidad del Chapter de QA o Data Governance).
- Despliegue automático a producción sin aprobación humana.
- Manejo de fuentes en tiempo real / streaming (aplica para un JTBD separado).

## Cuándo aplicar

- Cuando el equipo recibe una nueva fuente de datos en S3 (CSV, JSON, XML, Parquet, etc.) que debe incorporarse al catálogo Bronze.
- Cuando hay un backlog de ingestión que retrasa proyectos analíticos.
- Cuando se requiere estandarizar la calidad y nomenclatura de los esquemas Bronze en toda la organización.
- Cuando el cliente quiere demostrar valor rápido de IA aplicada a ingeniería de datos (caso piloto ideal).

## Cuándo no aplicar

- Fuentes de datos en tiempo real o con latencia sub-segundo (usar arquitecturas de streaming).
- Archivos con estructuras extremadamente anidadas o sin patrón definible que requieran modelado de dominio profundo.
- Clientes sin equipo técnico interno que pueda revisar y aprobar el código generado antes de producción.
- Cuando el cliente ya tiene un proceso de ingestión automatizado y maduro (el valor agregado sería marginal).
- Fuentes que contengan datos PII (Personally Identifiable Information — Información de Identificación Personal) sin un proceso de anonimización o enmascaramiento previo aprobado por el área de seguridad del cliente.

## Entradas (Inputs)

- **URI de S3 (Source Path):** Ruta al archivo o prefijo del bucket donde residen los datos fuente.
- **Muestra representativa de datos:** Primeras N filas del dataset en formato JSON, CSV o Parquet para análisis de estructura y perfilado estadístico inicial (distribución de valores, nulos, cardinalidad).
- **Metadatos de origen:** Swagger (especificación de API), DDL u otro contrato técnico que describa la fuente.
- **Contratos de Datos:** Definición formal del esquema esperado en formato YAML o JSON, acordada entre el productor y el consumidor del dato.
- **Patrón de particionamiento esperado:** Columnas y criterio de partición deseado (ej. `partition_by=year/month/day`, `partition_by=region`) para optimizar las consultas en la capa Bronze.
- **Estrategia de escritura:** Modo de carga definido por el cliente: `overwrite` (reemplazo total), `append` (acumulación) o `merge` (upsert por clave natural).
- **Estándares de nomenclatura del cliente:** Convenciones de nombres de tablas y columnas si existen.

## Flujo de alto nivel

El agente opera bajo el ciclo **ReAct** (Reasoning and Acting — Razonar y Actuar): un patrón agéntico que intercala trazas de razonamiento verbal con acciones concretas sobre el entorno.

1. **Reconocimiento de Fuente (Pensamiento):** El agente recibe el URI de S3 y razona sobre la muestra del archivo. Detecta el formato (CSV, JSON, Parquet, XML), el encoding (UTF-8, Latin-1, etc.) y la estructura de particionamiento si existe.

2. **Perfilado Estadístico:** El agente analiza la muestra para calcular métricas de calidad base: porcentaje de nulos por columna, cardinalidad, rango de valores numéricos y detección de outliers. Este perfilado determina la estrategia de tipado y los umbrales iniciales de la Suite de Tests de Calidad.

3. **Inferencia y Normalización de Esquema (Acción):** El agente propone nombres de columnas normalizados (sin espacios, tildes ni caracteres especiales) y mapea los tipos de datos al estándar de la plataforma, evitando el antipatrón de "todo es String". Define columnas de partición y estrategia de escritura.

4. **Observación y Ajuste (Observación):** El agente recibe el resultado de la inferencia, razona sobre inconsistencias detectadas (ej. campo numérico con caracteres especiales en el 1% de registros) y ajusta la estrategia de generación antes de continuar.

5. **Generación de Artefactos:** Con el esquema validado, el agente genera scripts modulares (Adapter + Lógica), DDL para el catálogo y columnas de auditoría estándar (`_ingestion_timestamp`, `_source_file`, `_batch_id`).

6. **Validación Automática (Patrón Reflection):** El agente aplica auto-crítica revisando su propio código en busca de errores de sintaxis, incumplimiento de estándares de nombres y referencias rotas antes de entregar el output.

7. **Auto-Documentación:** Genera el Data Dictionary y la Suite de Tests de Calidad con umbrales configurables.

8. **Revisión y Aprobación Humana:** El Data Engineer revisa el esquema propuesto y el código generado. Solo tras su aprobación explícita se procede con la primera ejecución en producción.

## Participación del chapter

El **Chapter de Ciencia de Datos** lidera de principio a fin esta solución con los siguientes roles:

- **Data Engineer Senior / Tech Lead de Datos:** Lidera el diseño de la arquitectura del agente y los estándares Bronze aplicables a todos los clientes. Define las convenciones de nomenclatura, tipos de datos y columnas de auditoría. Valida y aprueba los artefactos generados antes de la primera ejecución en producción. Es el punto de escalamiento ante casos de inferencia compleja (archivos anidados, Schema Drift severo).

- **Data Engineer:** Ejecuta el agente para cada nueva fuente de datos asignada. Revisa el código PySpark/Glue generado, ajusta los parámetros de inferencia cuando el agente requiere orientación, y aprueba el esquema Bronze propuesto. Realiza la primera ejecución supervisada en el entorno del cliente.

- **AI/ML Engineer (IA Enablement):** Mantiene y optimiza los prompts de inferencia del agente. Implementa y evoluciona los patrones ReAct (orquestación) y Reflection (validación automática). Gestiona el ciclo de vida del modelo subyacente y adapta el agente ante cambios en las plataformas de destino (AWS Glue, Databricks, Snowflake).

- **Data Steward / Analista de Calidad de Datos:** Valida que los umbrales de calidad definidos en DQSOps sean correctos y representativos para cada fuente de datos. Revisa y aprueba los diccionarios de datos generados. Es el interlocutor con el Product Owner del cliente para alinear expectativas sobre la calidad del dato en la capa Bronze.

## Dependencias

- **Cloud Ops (Roles IAM / VPC):** Permisos IAM (Identity and Access Management) de solo lectura al bucket S3 del cliente. Sin este acceso, el agente no puede operar.
- **Product Owners / Data Stewards (Accesos a Fuente):** Validación y aprobación del esquema propuesto antes de la primera ejecución en producción.
- **Framework DQSOps (Data Quality Scoring Operations):** Sistema de calidad corporativo para definir y monitorear umbrales de aceptación del dato. [[references/dqsops-framework.md]]
- **Catálogo de Datos del Cliente:** AWS Glue Data Catalog, Unity Catalog (Databricks) o equivalente activo y accesible.
- **Herramienta de Orquestación:** AWS Glue, Apache Airflow o Databricks Workflows activo en el entorno del cliente.
- **Decisión de Estándares Bronze — Arquitectura Medallón:** Convenciones de nombres, tipos, columnas de auditoría y patrón base de organización del Data Lakehouse (Bronze / Silver / Gold) sobre el que opera este JTBD. [[decisions/001-bronze-layer-standards.md]]
- **Arquitectura Hexagonal (Puertos y Adaptadores):** Patrón de diseño requerido para la generación de scripts modulares con separación Adapter/Lógica. [[references/hexagonal-architecture-guide.md]]
- **Patrón ReAct (Reasoning and Acting):** Marco de orquestación agéntica requerido para el ciclo de inferencia del agente. [[references/react-pattern-guide.md]]
- **Patrón Reflection (Reflexión):** Patrón de validación automática del código generado por el agente. [[references/reflection-pattern-guide.md]]

## Salidas esperadas (Outputs)

- **Scripts PySpark / Glue / Lambda 'ETLs' modulares:** Pipelines con separación de Adapters y Lógica de negocio bajo Arquitectura Hexagonal, listos para revisión y despliegue. [[outputs/bronze-ingestion-script.md]]
- **Documentación técnica del esquema:** Especificación del esquema Bronze normalizado con tipos de datos y columnas de auditoría estándar (`_ingestion_timestamp`, `_source_file`, `_batch_id`).
- **Diccionarios de datos:** Documento Markdown con descripción de columnas, tipos, origen y frecuencia estimada de actualización. [[outputs/data-dictionary.md]]
- **Suite de Tests de Calidad:** Conjunto de pruebas automatizadas con umbrales de calidad configurables para validación preventiva basada en DQSOps.
- **Registro en Catálogo de Datos:** DDL ejecutable para crear la tabla en el catálogo (AWS Glue Data Catalog, Unity Catalog o equivalente) con metadatos completos.

## Riesgos y consideraciones

- **Schema Drift:** Archivos cuya estructura cambia entre entregas (columnas nuevas, tipos modificados) pueden romper el pipeline generado. Mitigación: validación de esquema en cada ejecución y alertas ante cambios. [[limits/schema-drift-handling.md]]
- **Silent Drift:** Cambios sutiles en la distribución estadística de los datos (media, varianza) no detectados por CI/CD estándar. Pueden degradar silenciosamente la calidad en Bronze. Mitigación: validación distribucional con métricas KS (Kolmogorov-Smirnov) o PSI (Population Stability Index) en cada carga.
- **Encoding inconsistente:** Archivos legacy que mezclan encodings dentro del mismo fichero generan errores silenciosos de lectura. Mitigación: detección de encoding a nivel de muestra estadística, no solo del primer registro.
- **Volúmenes extremos en muestreo:** Si el archivo fuente supera el umbral de muestreo del agente, la muestra puede no ser representativa del esquema real. Mitigación: definir un tamaño de muestra mínimo garantizado y alertar cuando el archivo supere el umbral.
- **Archivos altamente anidados:** JSON o XML con jerarquías profundas aumentan la complejidad de inferencia y pueden requerir intervención manual.
- **Dependencia de permisos:** Sin acceso de lectura al bucket S3, el agente no puede operar. Debe gestionarse antes del inicio.
- **Aprobación humana obligatoria:** El código generado NO debe ejecutarse en producción sin revisión explícita del Data Engineer responsable.

## Variantes comunes

- **Traducción de ETLs Legacy (SQL → PySpark):** Variante orientada a migración de Stored Procedures y scripts SQL On-Premise hacia PySpark. Mayor complejidad, menor frecuencia. Prioridad 2.
- **Agente de Code Review para Pipelines:** Variante de análisis estático y seguridad sobre Pull Requests de código de datos. Complementario a este JTBD. Prioridad 3.
- **Versión acelerada (Hackathon / PoC):** Alcance reducido a inferencia de esquema y generación de DDL, sin script de ingestión completo. Ideal para demostrar valor en 1-2 días.

## Relación con otros chapters

- **Chapter de Cloud:** Colabora en la configuración de permisos IAM y la integración con servicios AWS (S3, Glue, Lake Formation). Punto de interacción: paso 1 del flujo (acceso a la fuente).
- **Chapter de QA / Data Governance:** Define las reglas de calidad de datos que el pipeline Bronze debe respetar. Punto de interacción: criterios de validación en el paso de auto-crítica.
- **Chapter de Architecture:** Valida que el patrón de ingestión generado sea coherente con la arquitectura Medallion (Bronze/Silver/Gold) definida para el cliente.

## Referencias relacionadas

- Decisión de estándares Bronze, Arquitectura Medallón y métricas de valor del JTBD (reducción >60% horas-hombre, margen >60%, entrega <5 días, uso recurrente ≥1 vez/semana, cobertura 90% clientes): [[decisions/001-bronze-layer-standards.md]]
- Límite conocido — Schema Drift: [[limits/schema-drift-handling.md]]
- Referencia de nomenclatura de columnas: [[references/column-naming-standard.md]]
- Framework DQSOps: [[references/dqsops-framework.md]]
- Guía Arquitectura Hexagonal: [[references/hexagonal-architecture-guide.md]]
- Guía Patrón ReAct: [[references/react-pattern-guide.md]]
- Guía Patrón Reflection: [[references/reflection-pattern-guide.md]]
