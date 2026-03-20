# JTBD: Agente de Ingestión y Curación Automática (Capa Bronze)

## Descripción y Dolor

Los equipos de ingeniería de datos invierten entre **4 y 16 horas por dataset** en tareas manuales y repetitivas para integrar nuevas fuentes de datos heterogéneas (CSV, JSON, Parquet, etc.) hacia la capa Bronze del Data Lake. Este cuello de botella genera una "deuda de tiempo" de 1 a 2 semanas al inicio de cada proyecto analítico, retrasando la toma de decisiones del negocio y convirtiendo a los Data Engineers en "plomeros de archivos" en lugar de arquitectos de valor.

En empresas que integran 10 o más fuentes nuevas al mes, esto representa hasta **160 horas hombre/mes** en trabajo de bajo valor agregado.

## Contexto

Esta solución aplica a organizaciones que operan un **Data Lake sobre S3** (o almacenamiento equivalente) y utilizan motores de procesamiento como **AWS Glue, Apache Spark o Delta Live Tables**. Es ideal para:

- Empresas en etapa de crecimiento de su plataforma de datos que reciben fuentes nuevas con frecuencia.
- Equipos de Data Engineering con backlog creciente de ingestión.
- Organizaciones que buscan escalar su Data Lake sin escalar linealmente su equipo técnico.

**Perfiles afectados:**
- **Data Engineers:** sobrecarga operativa en tareas repetitivas.
- **Data Analysts:** espera prolongada para acceder a datos frescos.
- **Business Owners / CDO:** toma de decisiones lenta por falta de datos disponibles.

## Alcance

- **Incluye:**
  - Detección automática de formato, encoding y estructura del archivo fuente.
  - Inferencia y normalización del esquema (nombres de columnas, tipos de datos).
  - Generación del script de ingestión (PySpark / SQL) con manejo de errores y auditoría.
  - Generación del DDL para registro en el catálogo de datos (ej. AWS Glue Data Catalog).
  - Auto-documentación del dataset (Data Dictionary en Markdown).
  - Validación sintáctica del código generado antes de la entrega.

- **No incluye:**
  - Transformaciones de negocio hacia capas Silver o Gold (responsabilidad del equipo de Analytics Engineering).
  - Definición de reglas de calidad de datos (Data Quality — responsabilidad del Chapter de QA o Data Governance).
  - Despliegue automático a producción sin aprobación humana.
  - Manejo de fuentes en tiempo real / streaming (aplica para un JTBD separado).

## Cuándo aplica

- Cuando el equipo recibe una nueva fuente de datos en S3 (CSV, JSON, XML, Parquet, etc.) que debe incorporarse al catálogo Bronze.
- Cuando hay un backlog de ingestión que retrasa proyectos analíticos.
- Cuando se requiere estandarizar la calidad y nomenclatura de los esquemas Bronze en toda la organización.
- Cuando el cliente quiere demostrar valor rápido de IA aplicada a ingeniería de datos (caso piloto ideal).

## Cuándo no aplica

- Fuentes de datos en tiempo real o con latencia sub-segundo (usar arquitecturas de streaming).
- Archivos con estructuras extremadamente anidadas o sin patrón definible que requieran modelado de dominio profundo.
- Clientes sin equipo técnico in-house que pueda revisar y aprobar el código generado antes de producción.
- Cuando el cliente ya tiene un proceso de ingestión automatizado y maduro (el valor agregado sería marginal).

## Entradas (Inputs)

- **URI de S3:** Ruta al archivo o prefijo del bucket donde residen los datos fuente.
- **Muestra del archivo:** Las primeras N filas del dataset para análisis de estructura.
- **Contexto de negocio:** Descripción breve del contenido (ej. "Ventas mensuales de Partner X").
- **Estándares de nomenclatura del cliente:** Convenciones de nombres de tablas y columnas si existen.
- **Permisos de acceso:** Credenciales de solo lectura al bucket S3 para exploración.

## Flujo de alto nivel

1. **Reconocimiento de Fuente:** El agente recibe el URI de S3 y lee una muestra del archivo. Detecta el formato (CSV, JSON, Parquet, XML), el encoding (UTF-8, Latin-1, etc.) y la estructura de particionamiento si existe.

2. **Inferencia y Normalización de Esquema:** A partir del schema crudo, el agente propone nombres de columnas normalizados (sin espacios, tildes ni caracteres especiales) y mapea los tipos de datos al estándar de la plataforma, evitando el antipatrón de "todo es String".

3. **Generación de Artefactos:** Con el esquema validado, el agente genera:
   - Script de ingestión (PySpark / AWS Glue Job / Delta Live Table).
   - DDL para registro en el catálogo (Glue Data Catalog o equivalente).
   - Columnas de auditoría estándar: `_ingestion_timestamp`, `_source_file`, `_batch_id`.

4. **Validación Sintáctica (Auto-Crítica):** El agente revisa su propio código en busca de errores de sintaxis, incumplimiento de estándares de nombres y referencias rotas antes de entregar el output.

5. **Auto-Documentación:** Genera un Data Dictionary en Markdown con descripción de cada columna, tipo de dato, origen y frecuencia estimada de actualización.

6. **Revisión y Aprobación Humana:** El Data Engineer revisa el esquema propuesto y el código generado. Solo tras su aprobación explícita se procede con la primera ejecución en producción.

## Participación del chapter

El **Chapter de Datos** lidera end-to-end esta solución:
- Diseña y mantiene el agente de ingestión y sus prompts de inferencia.
- Define los estándares de nomenclatura Bronze aplicables a todos los clientes.
- Provee consultoría técnica para adaptar el agente a las particularidades de cada plataforma (AWS Glue, Databricks, Snowflake, etc.).
- Acompaña la revisión y aprobación del primer ciclo de ingestión en cada cliente.

## Dependencias

- **Chapter de Cloud/Infraestructura:** Permisos IAM de solo lectura al bucket S3 del cliente.
- **Catálogo de Datos del Cliente:** AWS Glue Data Catalog, Unity Catalog (Databricks) o equivalente activo y accesible.
- **Decisión de Estándares Bronze:** [[decisions/001-bronze-layer-standards.md]] — Convenciones de nombres, tipos y columnas de auditoría.
- **Herramienta de Orquestación:** AWS Glue, Apache Airflow, o Databricks Workflows activo en el entorno del cliente.

## Salidas esperadas (Outputs)

- **Script de Ingestión:** [[outputs/bronze-ingestion-script.md]] — Pipeline PySpark/SQL listo para revisión y despliegue.
- **Registro en Catálogo:** DDL ejecutable para crear la tabla en el catálogo de datos con metadatos completos.
- **Data Dictionary:** [[outputs/data-dictionary.md]] — Documento Markdown con descripción de columnas, tipos, origen y frecuencia.
- **Esquema Bronze Normalizado:** Previsualización tabular del esquema propuesto para aprobación del ingeniero.

## Riesgos y consideraciones

- **Schema Drift:** Archivos cuya estructura cambia entre entregas (columnas nuevas, tipos modificados) pueden romper el pipeline generado. Mitigación: implementar validación de esquema en cada ejecución y alertas ante cambios.
- **Archivos altamente anidados:** JSON o XML con jerarquías profundas aumentan la complejidad de inferencia. El agente puede requerir intervención manual para estos casos.
- **Dependencia de permisos:** Sin acceso de lectura al bucket S3, el agente no puede operar. Debe gestionarse antes del inicio.
- **Aprobación humana obligatoria:** El código generado NO debe ejecutarse en producción sin revisión explícita del Data Engineer responsable.
- **Estándares de seguridad del cliente:** Si el cliente requiere encriptación específica en el pipeline (KMS keys, VPC endpoints), debe documentarse antes de la generación del script.

## Variantes comunes

- **Traducción de ETLs Legacy (SQL → PySpark):** Variante orientada a migración de Stored Procedures y scripts SQL On-Premise hacia PySpark. Mayor complejidad, menor frecuencia. Prioridad 2.
- **Agente de Code Review para Pipelines:** Variante de análisis estático y seguridad sobre Pull Requests de código de datos. Complementario a este JTBD. Prioridad 3.
- **Versión acelerada (Hackathon / PoC):** Alcance reducido a inferencia de esquema y generación de DDL, sin script de ingestión completo. Ideal para demostrar valor en 1-2 días.

## Relación con otros chapters

- **Chapter de Cloud:** Colabora en la configuración de permisos IAM y la integración con servicios AWS (S3, Glue, Lake Formation). Punto de interacción: paso 1 del flujo (acceso a la fuente).
- **Chapter de QA / Data Governance:** Define las reglas de calidad de datos que el pipeline Bronze debe respetar. Punto de interacción: criterios de validación en el paso de auto-crítica.
- **Chapter de Architecture:** Valida que el patrón de ingestión generado sea coherente con la arquitectura Medallion (Bronze/Silver/Gold) definida para el cliente.

## Referencias relacionadas

- Decisión de estándares Bronze: [[decisions/001-bronze-layer-standards.md]]
- Límite conocido — Schema Drift: [[limits/schema-drift-handling.md]]
- Referencia de nomenclatura de columnas: [[references/column-naming-standard.md]]
