# Límite: Profundidad de Anidamiento en Archivos JSON y XML

## Contexto de Identificación

Identificado en proyectos de ingesta Bronze dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma) al procesar archivos JSON (JavaScript Object Notation) y XML (Extensible Markup Language) provenientes de APIs (Application Programming Interfaces) de terceros, sistemas ERP (Enterprise Resource Planning) legacy y plataformas de pagos electrónicos. El problema se manifiesta cuando la estructura del archivo supera los 3 niveles de anidamiento jerárquico.

Fuentes típicas donde se presenta:

- Respuestas de APIs REST (Representational State Transfer) de pasarelas de pago que anidan objetos de transacción dentro de objetos de comercio dentro de objetos de lote.
- Archivos XML de entidades bancarias que usan namespaces complejos con múltiples niveles de etiquetas contenedoras.
- Exportaciones de sistemas ERP (SAP, Oracle) donde las relaciones maestro-detalle se representan como objetos anidados en lugar de tablas planas.
- Archivos JSON generados por sistemas de eventos (event-driven) donde el payload del evento contiene subobjetos con metadatos, datos de negocio y trazas de auditoría anidadas.

Se ha observado en archivos con 4 a 7+ niveles de profundidad, frecuentemente en clientes que integran múltiples sistemas origen en una sola fuente de datos.

## La Restricción

Cuando un archivo JSON o XML tiene más de **3 niveles de anidamiento**, la inferencia automática de esquema del agente de IA y el aplanamiento (flattening) de Apache Spark producen resultados inmantenibles o incorrectos:

- **Nombres de columnas ilegibles:** Spark aplana las jerarquías concatenando los nombres de cada nivel con un separador (ej. `transaction.merchant.address.city.zipcode`), generando nombres de columnas que exceden los 128 caracteres permitidos por la mayoría de catálogos de datos y que violan las convenciones de nomenclatura SOPP (formato `[environment]-[project]-[layer]-[source]`).
- **Explosión combinatoria en arrays anidados:** Si un objeto JSON contiene arrays de arrays (ej. `orders[].items[].variants[]`), la operación `explode` de Spark genera un producto cartesiano que multiplica el número de filas de forma exponencial. Un archivo de 10.000 registros puede convertirse en 10 millones de filas tras múltiples `explode`, saturando la memoria del Glue Job.
- **Pérdida de contexto semántico:** Al aplanar más de 3 niveles, la relación padre-hijo entre los datos se pierde. El esquema resultante es una tabla plana donde ya no es posible reconstruir la jerarquía original sin lógica adicional.
- **Inferencia incorrecta del agente:** El LLM (Large Language Model) que alimenta al agente tiene dificultad para razonar sobre estructuras profundamente anidadas, especialmente cuando la muestra de datos no contiene todos los niveles posibles de anidamiento. Puede generar código PySpark que omite niveles o que aplica `explode` en el orden incorrecto.

## Evidencia e Impacto

Si se ignora este límite:

- **Esquema Bronze inmantenible:** Las tablas resultantes tienen decenas de columnas con nombres largos e ininteligibles que ningún analista o modelo de ML (Machine Learning) puede consumir sin un mapeo manual previo.
- **OutOfMemoryError en Glue Jobs:** La explosión combinatoria de arrays anidados supera la memoria disponible de los workers de AWS Glue (16 GB (Gigabytes) en workers G.1X), provocando la caída del Job sin un mensaje de error claro. Ver [[limits/glue-job-memory-timeout.md]].
- **Datos incorrectos en Bronze:** Si el `explode` se aplica en el orden incorrecto, los registros hijos se asocian con los padres equivocados, produciendo combinaciones de datos que no existían en la fuente original. Este error es silencioso — los datos "parecen" válidos pero son incorrectos.
- **Tiempo de desarrollo manual:** Resolver la ingesta de un archivo con 5+ niveles de anidamiento requiere intervención manual del Data Engineer Senior para diseñar una estrategia de aplanamiento customizada, con un esfuerzo estimado de 8 a 24 horas por fuente.

## Señales de Alerta

- El agente genera nombres de columnas que exceden 128 caracteres o que contienen más de 3 separadores de nivel (ej. `root.level1.level2.level3.level4.field`).
- El `count()` del DataFrame tras un `explode` es 10x o más mayor que el número de registros del archivo fuente original.
- El Glue Job reporta uso de memoria superior al 80% del worker durante la fase de inferencia de esquema.
- La muestra de datos proporcionada al agente contiene campos de tipo `StructType` o `ArrayType` anidados dentro de otros `StructType` o `ArrayType` a más de 3 niveles de profundidad.
- El agente genera código PySpark con más de 3 operaciones `explode` encadenadas para una misma fuente.

## Workarounds y Mitigaciones

- **Establecer un límite de 3 niveles como umbral de automatización:** El agente de ingesta Bronze procesa automáticamente archivos con hasta 3 niveles de anidamiento. Archivos que superen este umbral deben ser escalados al Data Engineer Senior para diseño manual de la estrategia de aplanamiento. [[business-solutions/standard/jtbd/ai-agent-bronze-ingestion.md]]
- **Pre-procesamiento con jq o xmlstarlet:** Para archivos conocidos con jerarquías profundas, aplicar un paso previo de pre-procesamiento que extraiga los subobjetos relevantes y los aplane a archivos más simples antes de alimentar al agente.
- **Ingesta en múltiples tablas Bronze:** En lugar de aplanar toda la jerarquía en una sola tabla, descomponer el archivo en múltiples tablas Bronze que respeten la relación padre-hijo (ej. `bronze_orders`, `bronze_order_items`, `bronze_order_item_variants`), cada una con su propia clave de relación.
- **Limitar la profundidad del `explode`:** Configurar el agente para que aplique `explode` solo al primer nivel de arrays. Los niveles inferiores se almacenan como JSON string en la columna Bronze y se procesan en capas Silver donde el equipo de Analytics Engineering define la lógica de desanidamiento.
- **Solicitar archivos planos al cliente:** Cuando sea posible, negociar con el equipo productor del dato para que exporte los archivos en formato plano (CSV — Comma-Separated Values — o Parquet con estructura tabular) en lugar de JSON/XML anidado.
- **Documentar la estructura en el contrato de datos:** Si el archivo anidado es recurrente, documentar su estructura jerárquica completa en el contrato YAML (YAML Ain't Markup Language) para que el Data Engineer tenga un mapa antes de diseñar la estrategia de ingesta. [[references/sopp-naming-conventions.md]]
