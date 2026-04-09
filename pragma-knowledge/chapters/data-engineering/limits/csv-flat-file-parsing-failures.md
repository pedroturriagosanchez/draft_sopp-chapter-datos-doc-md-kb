# Límite: Fallos de Parseo en Archivos CSV y Planos Sin Contrato

## Contexto de Identificación

Identificado en proyectos de ingesta Bronze dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma) al procesar archivos CSV (Comma-Separated Values), TSV (Tab-Separated Values) y archivos de texto plano sin un contrato de datos formal. Es el límite más frecuente en operaciones de ingesta del día a día, presente en prácticamente todos los clientes que integran fuentes de datos de sistemas legacy, exportaciones manuales o proveedores externos.

El problema se manifiesta en los siguientes escenarios:

- Archivos CSV generados por sistemas distintos que usan delimitadores diferentes: comas en unos, punto y coma en otros, pipes (`|`) en algunos. A veces el mismo archivo alterna delimitadores entre secciones.
- Archivos donde las comillas de texto no están correctamente escapadas (ej. una dirección que contiene una coma dentro de comillas que no se cerraron: `"Calle 45, Edificio "Torres", Piso 3"`).
- Archivos con saltos de línea incrustados dentro de campos de texto libre (ej. campo de observaciones que contiene párrafos completos con enters).
- Archivos donde el sistema origen agrega un header repetido cada N filas por paginación interna (ej. cada 1.000 filas aparece de nuevo la fila de nombres de columnas).
- Archivos con filas de resumen o totales al final que tienen un número diferente de columnas que las filas de datos.

Se ha observado en clientes de todos los sectores, pero con mayor frecuencia en el sector financiero donde múltiples entidades reportan datos en formatos propios sin estandarización.

## La Restricción

Apache Spark asume que los archivos CSV (Comma-Separated Values) siguen un formato **consistente y predecible**: un delimitador único, comillas correctamente escapadas, una fila de header al inicio y filas de datos homogéneas en estructura. Cuando estas suposiciones no se cumplen, Spark produce datos corruptos sin generar errores:

- **Delimitadores inconsistentes:** Si el delimitador real no coincide con el configurado en Spark, cada fila se lee como una sola columna que contiene todo el texto de la fila, o se divide en un número incorrecto de columnas.
- **Comillas mal escapadas:** Spark cierra la comilla prematuramente o no la cierra, provocando que campos se fusionen o que filas enteras se interpreten como un solo campo.
- **Saltos de línea en campos:** Spark interpreta el salto de línea como fin de registro, partiendo una fila lógica en dos o más filas físicas. Las filas subsiguientes quedan desalineadas con el esquema.
- **Headers repetidos:** Las filas de header repetidas se interpretan como datos, insertando los nombres de columnas como valores en la tabla Bronze. Si la columna es numérica, Spark convierte el nombre del header a `null` silenciosamente.
- **Filas de resumen:** Las filas al final del archivo con estructura diferente (ej. "Total: 1,500,000") se intentan parsear con el esquema de las filas de datos, produciendo valores incorrectos o `null` en todas las columnas.

El modo `PERMISSIVE` de Spark (configuración por defecto) agrava el problema: en lugar de fallar, inserta `null` en las columnas que no puede parsear y continúa, propagando datos corruptos a Bronze.

## Evidencia e Impacto

Si se ignora este límite:

- **Filas fantasma en Bronze:** Los headers repetidos se convierten en registros de datos con valores `null` en columnas numéricas y nombres de columnas como valores en columnas de texto. Estos registros pasan todas las validaciones semánticas y contaminan agregaciones.
- **Desalineación de columnas:** Un salto de línea incrustado en un campo de texto partido en dos genera que la segunda mitad del campo se interprete como la siguiente fila. Todas las filas posteriores quedan corridas una posición, asignando valores a columnas incorrectas.
- **Duplicación aparente:** Si hay 10 filas de header repetidas en un archivo de 10.000 filas, se crean 10 registros basura que pueden afectar conteos, promedios y reportes de negocio.
- **Falsos negativos en validación:** El modo `PERMISSIVE` de Spark no genera errores. Las Puertas de Calidad del marco MLOps 2.0 (Machine Learning Operations — Operaciones de Aprendizaje Automático) pueden no detectar el problema si las filas corruptas representan un porcentaje pequeño del total (ej. 10 de 10.000 filas = 0.1%).

## Señales de Alerta

- El número de columnas inferido por Spark es 1 (todo el archivo se leyó como una sola columna), indicando que el delimitador configurado no coincide con el real.
- El número de filas leídas por Spark no coincide con el conteo de líneas del archivo fuente (señal de saltos de línea incrustados que partieron o fusionaron filas).
- Presencia de nombres de columnas (ej. "fecha", "monto", "nombre") como valores de datos en las primeras filas del DataFrame, indicando headers repetidos.
- La columna `_corrupt_record` de Spark contiene registros. **Importante:** esta columna NO se genera automáticamente en modo `PERMISSIVE` — requiere configurar explícitamente la opción `columnNameOfCorruptRecord="_corrupt_record"` al leer el archivo. Si esta opción no se configura, las filas malformadas se procesan con `null` silenciosamente sin dejar rastro.
- El perfilado estadístico del agente muestra valores atípicos en la cardinalidad: por ejemplo, una columna de `monto` que debería ser numérica muestra valores de texto entre sus valores únicos.

## Workarounds y Mitigaciones

- **Detección de delimitador antes del parseo:** En el paso de Reconocimiento de Fuente del agente, analizar las primeras 100 filas del archivo con una librería como `csv.Sniffer` (Python) para detectar automáticamente el delimitador real antes de configurar Spark. No asumir coma como delimitador por defecto.
- **Nunca usar `PERMISSIVE` sin auditoría (la configuración por defecto de Spark):** El modo `PERMISSIVE` de Spark sin la opción `columnNameOfCorruptRecord` es la causa raíz de la corrupción silenciosa. Inserta `null` en filas malformadas sin registrar cuáles ni cuántas. **Esta configuración por defecto no debe usarse en ningún pipeline de ingesta Bronze.** En su lugar, elegir uno de los dos modos siguientes según la criticidad de la fuente:
- **Opción 1 — `PERMISSIVE` con auditoría (recomendado para la mayoría de fuentes):** Configurar `mode=PERMISSIVE` junto con `columnNameOfCorruptRecord="_corrupt_record"`. Esto captura las filas malformadas en una columna separada sin detener el pipeline. Después de la lectura, contar las filas con `_corrupt_record` no nulo y aplicar un umbral: si las filas corruptas superan un porcentaje configurable del total (ej. > 1%), detener el pipeline y alertar al Data Engineer. Si están por debajo del umbral, continuar la ingesta y registrar las filas corruptas en un log de auditoría para revisión posterior. Este modo es el más práctico para fuentes grandes (> 100.000 filas) donde unas pocas filas malformadas no justifican detener toda la ingesta.
- **Opción 2 — `FAILFAST` (para fuentes financieras críticas):** Configurar `mode=FAILFAST` para que el pipeline se detenga ante la primera fila malformada. Este modo es obligatorio para fuentes donde la pérdida o corrupción de un solo registro es inaceptable: transacciones financieras, movimientos contables, datos regulatorios. Garantiza que ningún dato corrupto llegue a Bronze, a costa de bloquear la ingesta hasta que el archivo fuente sea corregido por el equipo productor.
- **Validación de conteo de columnas por fila:** Después de la lectura, verificar que todas las filas tengan el mismo número de columnas que el header. Las filas con un conteo diferente deben enviarse a una tabla de cuarentena para revisión manual.
- **Filtrado de headers repetidos:** Agregar un paso de limpieza que detecte y elimine filas donde los valores coinciden con los nombres de las columnas del header. Implementar como una validación estándar en todos los pipelines CSV. [[references/bronze-ingestion-procedural-guide.md]]
- **Manejo de saltos de línea incrustados:** Configurar Spark con la opción `multiLine=true` para archivos donde se sabe que los campos de texto contienen saltos de línea. Documentar esta configuración en el contrato YAML (YAML Ain't Markup Language) de la fuente.
- **Eliminación de filas de resumen:** Identificar y excluir filas de resumen al final del archivo mediante patrones conocidos (ej. la primera columna contiene "Total", "Subtotal", "Grand Total") o por posición (las últimas N filas).
- **Exigir contrato de datos al productor:** La mitigación definitiva es formalizar un contrato YAML para cada fuente CSV que especifique: delimitador, carácter de comilla, encoding, presencia de header, frecuencia de headers repetidos y existencia de filas de resumen. Sin este contrato, cada ingesta es un ejercicio de ingeniería inversa. [[decisions/001-bronze-layer-standards.md]]
