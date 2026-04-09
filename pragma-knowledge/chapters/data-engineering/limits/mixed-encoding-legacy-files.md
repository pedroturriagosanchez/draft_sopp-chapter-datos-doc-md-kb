# Límite: Encoding Mixto en Archivos Legacy

## Contexto de Identificación

Identificado en proyectos de ingesta Bronze dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma) al procesar archivos CSV (Comma-Separated Values), TXT y archivos de ancho fijo provenientes de sistemas legacy del sector financiero latinoamericano. El problema se presenta con mayor frecuencia en fuentes de datos generadas por:

- Sistemas core bancarios legacy (mainframes, AS/400) que exportan datos en encodings propios de la región (Latin-1, Windows-1252, ISO-8859-1) diferentes al estándar UTF-8.
- Procesos batch que concatenan archivos de múltiples sistemas origen, cada uno con su propio encoding, en un solo archivo de entrega.
- Archivos generados por aplicaciones de escritorio (Excel, Access) que mezclan caracteres del idioma español (tildes, eñes, signos de interrogación y exclamación invertidos) con caracteres especiales de otros idiomas.
- Exportaciones de bases de datos relacionales donde las columnas de texto libre contienen datos ingresados por usuarios en múltiples periodos con diferentes configuraciones regionales del sistema.

Se ha observado en clientes que operan en Colombia, Panamá y otros países de Latinoamérica donde los nombres de personas, direcciones y descripciones de productos contienen caracteres del español que no son compatibles con todos los encodings.

## La Restricción

Apache Spark (y por extensión AWS Glue) lee archivos de texto con un **encoding único y uniforme** por archivo. Cuando un archivo contiene registros en múltiples encodings, Spark aplica el encoding configurado (UTF-8 por defecto) a todo el archivo, produciendo uno de dos resultados:

- **Corrupción silenciosa de caracteres:** Los caracteres especiales (tildes, eñes, cedillas) se reemplazan por secuencias de bytes ilegibles (ej. `Ã±` en lugar de `ñ`, `Ã¡` en lugar de `á`) o por el carácter de reemplazo Unicode `�` (U+FFFD). Spark **no genera error ni warning** — los datos se escriben en Bronze como si fueran correctos.
- **Pérdida completa de registros:** Si el encoding incompatible genera bytes que Spark interpreta como caracteres de control (ej. fin de línea, fin de archivo), registros completos pueden ser truncados o fusionados con el siguiente registro.

La restricción aplica a los formatos CSV, TXT, archivos de ancho fijo y cualquier formato basado en texto plano. No aplica a Parquet, ORC (Optimized Row Columnar) ni Avro, que almacenan el encoding como metadato del archivo.

## Evidencia e Impacto

Si se ignora este límite:

- **Datos basura en Bronze:** Nombres de clientes, direcciones y descripciones de productos quedan ilegibles en la capa Bronze. Ejemplo: "José María Peña" se almacena como "JosÃ© MarÃ­a PeÃ±a". Estos datos se propagan a Silver y Gold y aparecen en reportes del cliente.
- **Fallos en joins y filtros:** Las columnas corruptas no matchean correctamente en operaciones de join o filtro. Un `WHERE ciudad = 'Bogotá'` no encuentra registros porque el valor almacenado es `BogotÃ¡`, generando datos faltantes en reportes que nadie investiga porque no hay error explícito.
- **Violación de regulaciones:** En el sector financiero, los nombres de clientes y datos personales deben almacenarse de forma legible para cumplir con regulaciones de auditoría (PCI DSS — Payment Card Industry Data Security Standard, GDPR — General Data Protection Regulation). Datos corruptos pueden ser interpretados como negligencia en la gestión de datos personales.
- **Detección tardía:** La corrupción de caracteres no genera errores en el pipeline. Se detecta típicamente cuando un usuario de negocio reporta que los nombres en un reporte están "mal escritos", semanas o meses después de la ingesta.

## Señales de Alerta

- Presencia de secuencias de bytes como `Ã±`, `Ã¡`, `Ã©`, `Ã³`, `Ã¼` en columnas de texto que deberían contener caracteres del español (ñ, á, é, ó, ü). Esta es la señal más común de un archivo Latin-1 leído como UTF-8.
- Incremento en la longitud promedio de columnas de texto respecto a la línea base histórica (los caracteres corruptos ocupan más bytes que los originales).
- Presencia del carácter de reemplazo Unicode `�` (U+FFFD) en los datos.
- Registros cuyo conteo de campos no coincide con el número de columnas esperado (señal de que un carácter de control fue interpretado como delimitador o fin de línea).
- El perfilado estadístico del agente detecta valores de cardinalidad anormalmente alta en columnas de texto (porque `José` y `JosÃ©` se cuentan como valores distintos).

## Workarounds y Mitigaciones

- **Detección de encoding a nivel de muestra estadística:** En el paso de Reconocimiento de Fuente del agente, no confiar en el encoding declarado ni en la detección del primer registro. Usar librerías como `chardet` (Python) o `ICU` para analizar una muestra estadística de al menos 100 registros distribuidos a lo largo del archivo (no solo los primeros) y determinar el encoding predominante. [[references/bronze-ingestion-procedural-guide.md]]
- **Modo de lectura con fallback de encoding:** Configurar Spark para leer el archivo con el encoding detectado y, en caso de caracteres no decodificables, registrar el número de registro y la columna afectada en un log de auditoría en lugar de reemplazar silenciosamente.
- **Validación post-ingesta de caracteres:** Después de la ingesta a Bronze, ejecutar una validación que busque las secuencias de bytes comunes de corrupción UTF-8/Latin-1 (`Ã±`, `Ã¡`, `Ã©`, `Ã³`) en las columnas de texto. Si se detectan, bloquear la ejecución y alertar al Data Engineer.
- **Pre-procesamiento de encoding:** Para archivos legacy conocidos con encoding mixto, incluir un paso previo de conversión a UTF-8 usando `iconv` o scripts Python antes de alimentar al agente de ingesta.
- **Documentar el encoding en el contrato de datos:** El contrato YAML (YAML Ain't Markup Language) de cada fuente debe incluir un campo `encoding` que especifique el encoding esperado del archivo (ej. `encoding: latin-1`, `encoding: utf-8`). El agente debe validar contra esta declaración.
- **Solicitar archivos en UTF-8 al cliente:** Cuando sea posible, negociar con el equipo productor del dato para que estandarice la exportación en UTF-8. Esta es la solución definitiva pero no siempre es viable en sistemas legacy. [[business-solutions/standard/jtbd/ai-agent-bronze-ingestion.md]]
