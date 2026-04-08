# 001 - Arquitectura Medallion como arquitectura de referencia de datos

## Estado
En aprobación

## Contexto
El agente genera ETLs como output para el JTBD de la ingesta automatizada en la capa bronze. Sin una decisión explícita sobre la arquitectura, el agente podría generar código ETL sin una visión de lo que podría contener la zona bronze: nivel de refinamiento de los datos, falta  de trazabilidad de los datos ingestados, formato físico de los datos, formato de tabla de los datos, registro en el catálogo de datos y mecanismos de seguridad de la información. Esto produciría outputs que probablemente funcionarían pero que a la larga el datalake se iría convirtiendo en un pantano de datos con lecturas de información ineficiente, poco trazable, sin una práctica clara de gobierno de datos, información no protegida e insegura, con distintos niveles de refinamiento de datos que no reflejan de una manera transparente la información recolectada de las fuentes OLTP — lo cual contradice el propósito del ecosistema. El pragmático necesita recibir artefactos ETL que puedan construir un datalake auditable, seguro, protegido, governable, legible, eficiente y optimizado.

## Opciones Consideradas

- **Opción 1:** El agente elige la arquitectura de capas según su criterio por proyecto. Pros: flexibilidad. Contras: capas inconsistentes, el pragmático no sabe qué estructura esperar, no se puede estandarizar la auditoría.
- **Opción 2:** El agente siempre genera bajo Arquitectura Medallion con la variante indicada por el stack del cliente (puede ser con capa landing o sin capa landing), distintos nombres para las tres capas (bronze, silver y gold). Pros: outputs consistentes, predecibles, auditables. Contras: más rígido, puede ser sobreingeniería con muchas capas de refinamiento y tiempos de desarrollo mas prolongados.
- **Opción 3:** El agente decide una arquitectura simple para fuentes de datos no heterogeneas, con complejidad de la información y con pocos casos de usos predecibles. Pros: pragmatismo. Contras: el agente tendría que decidir el alcance y escalabilidad de esa arquitectura de datos, dado la dificultad con la que llevaría poder integrar nuevas fuentes, con estructuras de datos mas complejas y con diferentes formatos de datos, nivel de acoplamiento de la capa de integración a diferentes tecnologías y el nivel de escalabilidad a extender de manera empresarial.

## Decisión Tomada

Opción 2: Arquitectura Medallion alineada con los lineamientos empresariales siempre. Todo output del ecosistema agéntico debe contemplar las siguientes capas de la arquitectura medallion:

### Capas de Jerarquía de Arquitectura Medallion
```
{datalake-root}/
  bronze/                 → Capa Raw (Ingesta). Reflejo exacto de la fuente. Inmutable, append-only.
    {source-system}/      → Organizado por sistema origen (ej. crm_salesforce, erp_sap, google_analytics).
      {entity}/           → Archivos en formato nativo (JSON, CSV, Parquet) particionados por fecha de ingesta (ej. year=2026/month=04).
  silver/                 → Capa Validada. Verdad técnica. Formatos transaccionales (Delta Lake, Apache Iceberg).
    master_data/          → Entidades core (clientes, productos). Datos limpios, deduplicados y tipificados.
    transactions/         → Hechos estandarizados. Nulos gestionados y PII (Datos Personales) enmascarados/tokenizados.
    quarantine/           → Datos que no pasaron los controles de calidad (Data Expectations) para revisión posterior.
  gold/                   → Capa Curada. Verdad de negocio. Optimizada para lectura masiva (BI) e inferencia. Formatos de tabla abierta (Delta Lake, Apache Iceberg)
    data_marts/           → Modelos dimensionales (Star Schema: Tablas de Hechos y Dimensiones) por área (Ventas, Riesgos).
    feature_store/        → Variables consolidadas y listas para el entrenamiento y consumo de modelos de Machine Learning.
```

### Reglas de diseño
- Las entidades manejadas en la capa bronze son puros appends, esto con la finalidad de manejar un histórico a nivel de auditoría de lo que se va a ingestando al datalake, debe ser considerado en la ingesta el particionamiento temporal y las columnas de auditoría para poder mantener una trazabilidad de la información ingestada.
- Las entidades en la capa silver son organizados por dominios de datos y pueden ser separadas a distintos niveles de refinamiento, se pueden tener tablas validadas y limpias bajo un prefijo que identifique ese primer nivel de refinamiento, adicionalmente un segundo nivel de refinamiento para las tablas que requiera historización y enriquecimiento bajo formas modeladas como por ejemplo, 3NF (tercera forma normal) y OBT (one big table), dependiendo del caso de uso en específico.
- Las entidades en la capa gold pueden ser organizadas por nivel de refinamiento de optimización para el consumo analítico, como por ejemplo se puede tener un primer nivel de modelo de datos más general: un modelo estrella o copo de nieve, para que despues a partir de este modelo exponer la información en modelos más especificos, organizados por casos de uso expuestos como datamarts.

## Implicaciones

- **Positivas:**
  - **Estructura de arquitectura medallion predecible, robusta y flexible:** el pragmático sabe exactamente cómo se estructuran los diferentes niveles de refinamiento de los datos y para qué sirve cada capa.
  - **La auditoría se estandariza:** es más fácil verificar reglas de dependencia, aislamiento de modelos, linaje y ubicación física de los datos.
  - **Reutilización y democratización eficiente:** al tener una capa Silver (verdad técnica) bien consolidada, múltiples áreas de negocio pueden construir sus propios modelos Gold sin necesidad de volver a conectarse a los sistemas origen, evitando cuellos de botella.
  - **Resiliencia y recuperación ante desastres (Reprocesamiento):** como la capa Bronze conserva el histórico crudo e inmutable, si se introduce un "bug" en las reglas de negocio de la capa Gold, el equipo puede destruir y reconstruir las capas superiores sin perder información.
  - **Aplicación de reglas de calidad es más fácil:** el pragmático sabe dónde y en qué punto puede aplicar las reglas de calidad de datos (Data Expectations) con la finalidad de proteger la integridad y consistencia de la información antes de que llegue al usuario final.
  - **Seguridad y gobierno granulares:** permite implementar el principio de menor privilegio. Los analistas solo acceden a Gold, mientras que los datos con PII (Información Personal Identificable) en Bronze/Silver quedan en cuarentena o enmascarados y restringidos solo a ingenieros autorizados.

- **Negativas / Riesgos:**
  - **Overengineering (Complejidad innecesaria):** aplicar rígidamente las tres capas para un proyecto pequeño, un flujo de datos muy simple o una PoC (Prueba de Concepto) puede resultar en una arquitectura sobredimensionada que prolonga el tiempo de desarrollo.
  - **Aumento de costos de infraestructura:** la redundancia controlada tiene un precio. Almacenar el mismo dato en tres estados diferentes (crudo, limpio, agregado) multiplica los costos de almacenamiento (aunque S3/ADLS sea barato) y, sobre todo, aumenta los costos de cómputo por el procesamiento continuo (ETL/ELT) entre capas.
  - **Mayor latencia en el "Time-to-insight" del dato:** el rigor de pasar los datos por múltiples procesos de validación, limpieza y agregación significa que un nuevo dato generado en el origen puede tardar más en estar visible en los dashboards finales, a menos que se invierta en costosos pipelines de streaming.
  - **Sobrecarga de mantenimiento y evolución de esquemas:** si un sistema origen cambia su estructura (ej. añade o elimina columnas), el equipo de datos debe gestionar la evolución del esquema (*Schema Evolution*) a través de los pipelines de Bronze, Silver y Gold, lo que requiere contratos de datos muy sólidos y mantenimiento continuo.


## Referencias Relacionadas
- `kb/references/01-architecture/` (references de arquitectura medallion)

