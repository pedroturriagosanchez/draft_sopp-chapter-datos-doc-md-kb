# Límite: Memoria y Timeout de AWS Glue Jobs

## Contexto de Identificación

Identificado en la ejecución de pipelines de ingesta Bronze dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma) sobre AWS Glue (servicio de ETL — Extract, Transform, Load — gestionado de Amazon Web Services). El problema se manifiesta cuando los archivos fuente superan la capacidad de procesamiento de los workers de Glue configurados, ya sea por volumen de datos, complejidad del esquema o acumulación de particiones.

Escenarios donde se presenta:

- Archivos Parquet o CSV (Comma-Separated Values) individuales que superan los 10 GB (Gigabytes) sin particionamiento, cargados completamente en memoria por el driver de Spark.
- Fuentes de datos con cientos de columnas donde las operaciones de inferencia de esquema y conversión de tipos consumen memoria de forma proporcional al ancho del esquema.
- Ingesta de backlog acumulado: cuando un Glue Job intenta procesar semanas de archivos atrasados en una sola ejecución.
- Operaciones de `explode` sobre archivos JSON (JavaScript Object Notation) anidados que generan explosión combinatoria de filas. Ver [[limits/nested-json-xml-depth.md]].
- Acumulación de particiones pequeñas ("small files problem") donde miles de archivos de menos de 1 MB (Megabyte) por partición generan overhead de I/O (Input/Output) que satura la coordinación del driver.

Los tipos de workers disponibles en AWS Glue al momento de este documento son:

- **G.1X:** 4 vCPU (Virtual CPU), 16 GB RAM (Random Access Memory). Worker estándar.
- **G.2X:** 8 vCPU, 32 GB RAM. Para cargas de trabajo intensivas en memoria.
- **G.4X:** 16 vCPU, 64 GB RAM. Recomendado para datasets muy grandes.
- **G.8X:** 32 vCPU, 128 GB RAM. Para cargas extremas.

## La Restricción

AWS Glue impone restricciones de memoria y tiempo que afectan directamente la ejecución de pipelines de ingesta:

- **Memoria del driver:** El driver de Spark en Glue tiene una memoria máxima determinada por el tipo de worker. En workers G.1X (los más comunes), el driver dispone de aproximadamente 10 GB de memoria efectiva para la JVM (Java Virtual Machine). Si un DataFrame excede esta memoria, el Job falla con `java.lang.OutOfMemoryError: Java heap space` o se degrada con garbage collection excesivo.
- **Timeout máximo:** Los Glue Jobs tienen un timeout configurable de hasta 48 horas. Sin embargo, un Job que se acerca a este límite indica un problema de diseño: procesos de ingesta Bronze no deberían tardar más de 1-2 horas incluso para fuentes grandes.
- **Límite de DPU (Data Processing Units):** Cada cuenta de AWS tiene un límite de DPU concurrentes (por defecto 100 DPU). Si múltiples Glue Jobs se ejecutan simultáneamente, pueden competir por DPU disponibles, encolando ejecuciones o fallando por falta de recursos.
- **Límite de particiones en el Glue Data Catalog:** Una tabla en el catálogo soporta hasta 10 millones de particiones. Esquemas de particionamiento granulares (ej. `year/month/day/hour/source`) pueden alcanzar este límite en 2-3 años para clientes con alto volumen de fuentes.
- **Tamaño máximo de response del catálogo:** Las operaciones de `GetPartitions` del Glue Data Catalog tienen un límite de 1 MB por respuesta. Tablas con muchas particiones requieren paginación, lo que ralentiza el descubrimiento de datos.

## Evidencia e Impacto

Si se ignora este límite:

- **Fallo del Job sin mensaje claro:** El `OutOfMemoryError` de la JVM frecuentemente se reporta como un error genérico en los logs de Glue, sin indicar qué operación específica agotó la memoria. El Data Engineer puede pasar horas investigando la causa raíz.
- **Costos descontrolados:** Un Glue Job que corre por 48 horas en workers G.2X consume 48 × 2 × $0.44 = $42.24 USD por ejecución. Si esto ocurre repetidamente, erosiona el margen de rentabilidad del proyecto calculado con COCOMO III (Constructive Cost Model). [[references/cocomo-roi-estimation-guide.md]]
- **Bloqueo de pipeline:** Si un Job falla por timeout o memoria después de 6 horas de ejecución, el pipeline completo queda bloqueado hasta que el Data Engineer investiga, corrige y re-ejecuta manualmente, incrementando el Time-to-Insight.
- **Datos parcialmente escritos:** Si el Job falla a la mitad de la escritura en S3 (Simple Storage Service), pueden quedar archivos parciales en la ruta de destino que contaminarán la siguiente ejecución si no se limpian.

## Señales de Alerta

- Uso de memoria del driver reportado en los logs de Glue superior al 80% de la capacidad del worker.
- Mensajes de garbage collection frecuentes en los logs: `GC overhead limit exceeded` o `Full GC` con pausas superiores a 10 segundos.
- Tiempo de ejecución del Job que supera el doble del tiempo promedio histórico para la misma fuente de datos.
- Número de archivos por partición superior a 1.000 (señal de "small files problem").
- Errores de `ThrottlingException` al interactuar con el Glue Data Catalog, indicando que se alcanzó el límite de solicitudes por segundo.

## Workarounds y Mitigaciones

- **Dimensionar el tipo de worker según el volumen:** Documentar el volumen máximo esperado de cada fuente en el contrato YAML (YAML Ain't Markup Language) y seleccionar el tipo de worker acorde: G.1X para fuentes < 5 GB, G.2X para 5-20 GB, G.4X para > 20 GB. Incluir esta selección como parte del paso de Planificación Inicial (Pre-Act) del agente.
- **Particionamiento del archivo fuente:** Si el archivo fuente supera los 10 GB, solicitar al equipo productor que entregue los datos particionados por fecha, región u otro criterio. Si no es posible, implementar un paso previo de segmentación antes de la ingesta.
- **Procesamiento incremental:** En lugar de procesar todo el backlog acumulado en una sola ejecución, configurar el pipeline para procesar archivos en lotes de tamaño controlado (ej. 5 GB por lote), encadenando ejecuciones secuenciales.
- **Compactación de archivos pequeños:** Implementar un paso de compactación que consolide miles de archivos pequeños (< 1 MB) en archivos más grandes (128 MB-256 MB, el tamaño óptimo de bloque de Spark) antes de la ingesta. Esto reduce el overhead de I/O.
- **Timeout conservador:** Configurar un timeout de 2 horas para los Glue Jobs de ingesta Bronze. Si el Job necesita más de 2 horas, es una señal de que necesita redimensionamiento o particionamiento, no más tiempo.
- **Monitoreo de costos por Job:** Implementar alertas en AWS CloudWatch cuando el costo de un Glue Job supere un umbral predefinido (ej. $10 USD por ejecución). Integrar esta métrica con el tablero de observabilidad de la capa Nexo del SOPP. [[business-solutions/standard/jtbd/ai-agent-bronze-ingestion.md]]
- **Limpieza de archivos parciales:** Incluir en el pipeline una lógica de rollback que elimine archivos parciales escritos en S3 si el Job falla, para evitar contaminación en la siguiente ejecución.
