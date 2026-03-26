# Referencia: Guía Procedimental — Ejecución del Agente de Ingestión Bronze

## Propósito

Esta guía define el proceso paso a paso para ejecutar el Agente de Ingestión Automatizada en la Capa Bronze (Cruda) del Data Lakehouse. Al finalizar su lectura y ejecución, el Data Engineer habrá integrado una nueva fuente de datos en el catálogo Bronze con su pipeline PySpark/Glue generado, validado y aprobado para producción, reduciendo el tiempo de integración de 4-16 horas a menos de 15 minutos de configuración activa.

## Ámbito de Aplicación

Esta guía es de aplicación **obligatoria** para todo Data Engineer del Chapter de Datos de Pragma que ejecute el JTBD de ingestión Bronze en cualquier cliente que opere:
- Data Lake sobre AWS S3 (o almacenamiento equivalente compatible con S3 API).
- Motor de procesamiento: AWS Glue, Apache Spark, Databricks o Delta Live Tables.
- Catálogo de datos: AWS Glue Data Catalog, Unity Catalog (Databricks) o equivalente.

No aplica para fuentes de datos en tiempo real / streaming (ver JTBD de streaming CDC).

---

## Paso a Paso / Lineamientos

### Fase 0 — Preparación y Validación de Prerrequisitos

Antes de iniciar cualquier ejecución del agente, verificar que se cumplen todas las condiciones de entrada.

**0.1. Recopilar las entradas requeridas:**
- URI de S3 del archivo o prefijo del bucket fuente (ej. `s3://bucket-cliente/ventas/2026/`).
- Muestra representativa del archivo (mínimo 100 filas, máximo 1000 filas).
- Contexto de negocio en lenguaje natural (ej. "Ventas mensuales de Partner X, frecuencia mensual").
- Estándares de nomenclatura del cliente (si existen convenciones propias).
- Credenciales IAM de solo lectura al bucket S3.

**0.2. Verificar permisos IAM:**
Ejecutar el siguiente comando de validación antes de iniciar:
```bash
aws s3 ls s3://[bucket-cliente]/[prefijo]/ --profile [perfil-iam]
```
Si el comando retorna `AccessDeniedException`, detener el proceso y gestionar los permisos con el Chapter de Cloud antes de continuar.

**0.3. Verificar el entorno de ejecución:**
- Confirmar que AWS Glue / Databricks está activo y accesible.
- Confirmar que el catálogo de datos (Glue Data Catalog / Unity Catalog) está disponible.
- Confirmar que existe un entorno de staging/desarrollo para la primera ejecución.

**0.4. Aplicar convención de nomenclatura SOPP:**
El nombre de la tabla Bronze debe seguir el formato estándar:
```
[environment]-[project]-[layer]-[source]
```
Ejemplos:
- `dev-retail-bronze-sales-partner-x`
- `prod-banking-bronze-transactions-core`
- `stg-health-bronze-patients-ehr`

---

### Fase 1 — Reconocimiento de Fuente

**1.1.** Proporcionar al agente el URI de S3 y el contexto de negocio.

**1.2.** El agente ejecuta el análisis de muestra:
- Detecta el formato del archivo: CSV, JSON, Parquet, XML, ORC.
- Identifica el encoding: UTF-8, Latin-1, UTF-16.
- Detecta delimitadores (para CSV): coma, punto y coma, pipe (`|`), tabulación.
- Identifica estructura de particionamiento si existe (ej. `year=2026/month=03/`).

**1.3.** Revisar el **Perfil Técnico del Archivo** generado por el agente:
- Confirmar que el formato detectado es correcto.
- Confirmar que el encoding es el esperado.
- Si el archivo tiene estructura anidada > 3 niveles, activar el protocolo de intervención manual (ver [[limits/schema-drift-and-known-failures.md]] — Límite 2).

---

### Fase 2 — Inferencia y Normalización de Esquema

**2.1.** El agente propone el **Esquema Bronze Normalizado**:
- Nombres de columnas en `snake_case`, sin espacios, tildes ni caracteres especiales.
- Tipos de datos mapeados al estándar de la plataforma (evitar el antipatrón "todo es String").
- Mapeo de tipos recomendado:

| Tipo Origen | Tipo Bronze Recomendado |
|-------------|------------------------|
| Texto libre | `StringType` |
| Número entero | `LongType` o `IntegerType` |
| Número decimal | `DoubleType` o `DecimalType(18,4)` |
| Fecha | `DateType` (formato ISO 8601) |
| Timestamp | `TimestampType` (UTC) |
| Booleano | `BooleanType` |
| JSON anidado | `StringType` (serializado) o `StructType` |

**2.2.** Revisar y aprobar el esquema propuesto:
- Verificar que los nombres de columnas reflejan el dominio de negocio.
- Verificar que los tipos de datos son correctos para el uso analítico previsto.
- Agregar o modificar columnas si es necesario antes de proceder.

**2.3.** Registrar el esquema aprobado como **contrato de datos** en formato YAML:
```yaml
contract:
  name: [nombre-tabla-bronze]
  version: "1.0.0"
  source: [uri-s3]
  columns:
    - name: [nombre_columna]
      type: [tipo_dato]
      nullable: [true/false]
      description: [descripcion]
```

---

### Fase 3 — Generación de Artefactos

**3.1.** El agente genera el **Script de Ingestión** siguiendo la Arquitectura Hexagonal (Puertos y Adaptadores):
- La lógica de transformación reside en el núcleo del hexágono, desacoplada de la infraestructura.
- Los adaptadores primarios (triggers) son independientes del motor: EventBridge, Lambda, o ejecución manual.
- Los adaptadores secundarios (destinos) son intercambiables: S3, RDS, SNS/SQS.

**3.2.** El script generado debe incluir obligatoriamente:
```python
# Columnas de auditoría estándar SOPP
df = df.withColumn("_ingestion_timestamp", current_timestamp()) \
       .withColumn("_source_file", lit(source_uri)) \
       .withColumn("_batch_id", lit(batch_id)) \
       .withColumn("_schema_version", lit("1.0.0"))
```

**3.3.** El agente genera el **DDL para el catálogo de datos**:
- Nombre de tabla siguiendo la convención SOPP.
- Descripción de la tabla y cada columna.
- Particionamiento recomendado (si aplica).
- Tags de clasificación de datos (PII, confidencial, público).

**3.4.** El agente genera el **Data Dictionary** en Markdown con:
- Descripción de cada columna.
- Tipo de dato y restricciones.
- Origen del campo en el sistema fuente.
- Frecuencia estimada de actualización.
- Indicador de PII (Información Personalmente Identificable).

---

### Fase 4 — Validación Automática (Auto-Crítica del Agente)

El agente ejecuta las **4 Puertas de Calidad MLOps 2.0** sobre el código y los datos generados:

**4.1. Validación Estructural (Severidad: BLOCK)**
- Verifica que el esquema del archivo cumple con el contrato YAML registrado.
- Detecta columnas faltantes, tipos incorrectos o columnas adicionales no declaradas.
- Si falla: el pipeline se detiene. No se escribe ningún dato en Bronze.

**4.2. Validación Semántica (Severidad: WARN/BLOCK según configuración)**
- Verifica reglas de negocio básicas:
  - Fechas de transacción no son futuras.
  - Precios y cantidades no son negativos.
  - Campos obligatorios no son nulos.
  - Identificadores únicos no están duplicados.
- Si falla en campo crítico: BLOCK. Si falla en campo no crítico: WARN + registro en log de auditoría.

**4.3. Validación Temporal (Severidad: AUDIT)**
- Verifica el orden cronológico de los registros.
- Detecta "fugas temporales" (temporal leakage): registros con fechas futuras que podrían contaminar modelos de ML.
- Verifica la frescura de los datos: alerta si los datos tienen más de N días de antigüedad respecto a la fecha esperada.
- Resultado: registro en tablero de gobernanza sin bloquear el pipeline.

**4.4. Validación Distribucional (Severidad: WARN)**
- Compara las estadísticas actuales (media, varianza, percentiles) contra la línea base histórica.
- Utiliza métricas KS (Kolmogorov-Smirnov) o PSI (Population Stability Index).
- Si PSI > 0.2 o KS p-value < 0.05: genera alerta de posible Schema Drift o cambio en el comportamiento del proveedor.
- Resultado: alerta al Data Engineer sin bloquear el pipeline en la primera ejecución.

**4.5. Validación Sintáctica del Código:**
- El agente revisa el código PySpark generado buscando:
  - Errores de sintaxis Python.
  - Referencias a columnas que no existen en el esquema.
  - Imports faltantes.
  - Incumplimiento de convenciones de nomenclatura SOPP.

---

### Fase 5 — Revisión y Aprobación Humana

**5.1.** El Data Engineer revisa los artefactos generados:
- Esquema Bronze Normalizado.
- Script PySpark / Glue Job.
- DDL del catálogo.
- Data Dictionary.
- Reporte de validaciones (resultados de las 4 puertas de calidad).

**5.2.** Ejecutar el script en el **entorno de staging** con datos sintéticos o una muestra reducida.

**5.3.** Verificar los resultados en staging:
- La tabla Bronze se creó correctamente en el catálogo.
- Los tipos de datos son los esperados.
- Las columnas de auditoría están presentes.
- No hay errores en los logs de Glue/Spark.

**5.4.** Registrar la aprobación formal:
- Nombre del Data Engineer aprobador.
- Fecha y hora de aprobación.
- Versión del esquema aprobado.
- Resultado de las validaciones en staging.

**5.5.** Promover el pipeline a producción solo tras la aprobación registrada.

---

### Fase 6 — Auto-Documentación y Cierre

**6.1.** Actualizar el `manifest.json` del repositorio pragma-knowledge con la nueva fuente integrada.

**6.2.** Registrar el contrato de datos YAML en el repositorio de contratos del cliente.

**6.3.** Configurar las alertas de monitoreo continuo:
- Alerta de Schema Drift (validación distribucional en cada ejecución).
- Alerta de frescura de datos (validación temporal).
- Dashboard de gobernanza con métricas KS/PSI históricas.

**6.4.** Comunicar al Data Analyst y Business Owner que la fuente está disponible en Bronze.

---

## Checklist de Verificación

### Pre-ejecución
- [ ] URI de S3 proporcionado y accesible.
- [ ] Permisos IAM verificados (`s3:GetObject`, `s3:ListBucket`).
- [ ] Contexto de negocio documentado.
- [ ] Convención de nomenclatura SOPP aplicada al nombre de la tabla.
- [ ] Entorno de staging disponible.
- [ ] Catálogo de datos activo y accesible.

### Durante la ejecución
- [ ] Perfil técnico del archivo revisado y confirmado.
- [ ] Esquema Bronze Normalizado aprobado por el Data Engineer.
- [ ] Contrato de datos YAML generado y registrado.
- [ ] Script PySpark generado con columnas de auditoría estándar.
- [ ] DDL del catálogo generado.
- [ ] Data Dictionary generado.
- [ ] Las 4 puertas de calidad ejecutadas sin errores BLOCK.

### Post-ejecución
- [ ] Pipeline ejecutado exitosamente en staging.
- [ ] Aprobación formal registrada (nombre, fecha, versión).
- [ ] Pipeline promovido a producción.
- [ ] Alertas de monitoreo configuradas.
- [ ] `manifest.json` actualizado.
- [ ] Data Analyst y Business Owner notificados.

---

## Herramientas y Recursos

| Herramienta | Propósito | Documentación |
|-------------|-----------|---------------|
| AWS Glue | Motor de procesamiento ETL serverless | [AWS Glue Docs](https://docs.aws.amazon.com/glue/) |
| Apache Spark / PySpark | Motor de procesamiento distribuido | [Spark Docs](https://spark.apache.org/docs/latest/) |
| AWS Glue Data Catalog | Catálogo de metadatos | [Glue Catalog Docs](https://docs.aws.amazon.com/glue/latest/dg/catalog-and-crawler.html) |
| Delta Lake | Formato de tabla con soporte para schema evolution | [Delta Lake Docs](https://docs.delta.io/) |
| AWS Deequ | Validación de calidad de datos en Spark | [Deequ GitHub](https://github.com/awslabs/deequ) |
| Great Expectations | Framework de validación de datos | [GE Docs](https://docs.greatexpectations.io/) |
| AWS Lake Formation | Gobernanza y control de acceso al Data Lake | [Lake Formation Docs](https://docs.aws.amazon.com/lake-formation/) |
| AWS CloudTrail | Auditoría de accesos y operaciones | [CloudTrail Docs](https://docs.aws.amazon.com/cloudtrail/) |

### Convención de Nomenclatura SOPP
Formato obligatorio para tablas Bronze:
```
[environment]-[project]-[layer]-[source]
```
- `environment`: `dev`, `stg`, `prod`
- `project`: nombre del proyecto o cliente en kebab-case
- `layer`: `bronze`, `silver`, `gold`
- `source`: nombre de la fuente de datos en kebab-case

### Arquitectura de Referencia
- Patrón de Arquitectura Hexagonal (Puertos y Adaptadores): [[decisions/002-hexagonal-architecture.md]]
- Estándares de la Capa Bronze: [[decisions/001-bronze-layer-standards.md]]
- Límites conocidos del agente: [[limits/schema-drift-and-known-failures.md]]

---

## Sección Complementaria: Optimización del Agente y Técnicas Avanzadas

### Optimización de Prompts e Inyección de Dependencias

La eficiencia del agente se mejora mediante dos patrones técnicos que el Data Engineer debe conocer:

**Optimización Automática de Prompts:**
El agente ajusta sus prompts según las métricas de evaluación y el modelo específico utilizado. Si la tasa de errores de inferencia supera el 5%, el sistema activa un ciclo de optimización que refina las instrucciones del prompt para el tipo de archivo problemático. Este proceso es transparente para el Data Engineer pero debe monitorearse en el dashboard de Nexo.

**Inyección de Dependencias para Modelos:**
El agente implementa el patrón de Inyección de Dependencias para intercambiar modelos o herramientas sin reescribir la lógica de orquestación:
```python
# Ejemplo conceptual de inyección de dependencias
class BronzeIngestionAgent:
    def __init__(self, llm_provider, schema_validator, catalog_client):
        self.llm = llm_provider          # GPT-4, Claude, Llama, etc.
        self.validator = schema_validator # Deequ, Great Expectations
        self.catalog = catalog_client    # Glue, Unity Catalog
```
Esto permite pasar de GPT-4 a Claude o a un modelo local pequeño (SLM) sin modificar la lógica de inferencia de esquema.

---

### Modelo de Severidad CDV (Continuous Data Validation) — Referencia Completa

El agente opera bajo el siguiente modelo de severidad definido en la política de MLOps 2.0:

| Puerta de Calidad | Criterio de Validación | Severidad | Acción del Agente |
|-------------------|----------------------|-----------|-------------------|
| Estructural | Cumplimiento de tipos y columnas en contrato YAML | **BLOCK** | Detiene el pipeline. No escribe datos en Bronze. |
| Semántica (crítica) | Violación de regla de negocio en campo obligatorio | **BLOCK** | Detiene el pipeline. Genera reporte de error. |
| Semántica (no crítica) | Violación de regla de negocio en campo opcional | **WARN** | Continúa. Registra en log de auditoría. |
| Temporal | Orden cronológico, frescura, temporal leakage | **AUDIT** | Continúa. Registra en tablero de gobernanza. |
| Distribucional | PSI > 0.2 o KS p-value < 0.05 | **WARN** | Continúa. Alerta al Data Engineer. |
| Sintáctica del código | Logprob promedio < -2.0 en segmento generado | **REFLECTION** | Activa ciclo de auto-corrección del agente. |

**Umbrales de referencia para validación distribucional:**
- PSI < 0.1: Sin cambio significativo en la distribución. ✅
- PSI 0.1 - 0.2: Cambio moderado. Monitorear. ⚠️
- PSI > 0.2: Cambio significativo. Alerta de Schema Drift. 🚨
- KS p-value > 0.05: Distribuciones estadísticamente similares. ✅
- KS p-value < 0.05: Distribuciones estadísticamente diferentes. 🚨

---

### Integración con la Capa Nexo del SOPP

La **Capa Nexo** es el DataLake inteligente del SOPP que centraliza la observabilidad de todos los agentes. Todo pipeline generado debe instrumentarse para reportar a Nexo:

**Métricas obligatorias a registrar en Nexo:**
- Latencia total del pipeline (desde inicio hasta escritura en Bronze).
- Número de tokens consumidos por el LLM en la generación de artefactos.
- Resultado de cada puerta de calidad (PASS/FAIL/WARN).
- Volumen de registros procesados vs. rechazados.
- Identificador del cliente para facturación granular.

**Documentación bajo MCP (Model Context Protocol):**
Las arquitecturas de pipeline deben documentarse bajo el protocolo MCP para que los agentes de IA puedan consultar definiciones estandarizadas y ejecutar tareas con mayor precisión. Esto incluye:
- Definición de los puertos de entrada y salida del pipeline.
- Contratos de datos en formato YAML versionado.
- Políticas de calidad aplicables al pipeline.

---

### Modelado Económico — Estimación de ROI con COCOMO III

Para proyectos que arrancan desde cero, el Data Engineer debe documentar el ROI estimado usando el modelo COCOMO III:

**Parámetros clave:**
- **KLOC reducido:** El agente reduce el KLOC escrito por humanos en un 80%, disminuyendo directamente el esfuerzo estimado.
- **EAF (Effort Adjustment Factor):** Los multiplicadores de esfuerzo mejoran al eliminar el Toil: menor complejidad del producto, mayor capacidad del equipo.
- **Meta de rentabilidad:** > 60% de margen en proyectos desde cero.

**Fórmula de ahorro estimado:**
```
Horas ahorradas = (Fuentes nuevas/mes × Horas promedio/fuente) × % reducción del agente
Ejemplo: 10 fuentes × 10 horas × 80% = 80 horas/mes ahorradas
Costo ahorrado = 80 horas × tarifa hora Data Engineer
```

**Registro obligatorio en el JTBD Tracker:**
Cada ejecución del agente debe registrar en el JTBD Tracker:
- `jtbd-file-name`: nombre del archivo JTBD ejecutado.
- `Responsable`: Data Engineer que ejecutó el JTBD.
- `Estado`: Propuesto / En Ejecución / Completado / Disponibilizado.
- `Cuentas impactando`: clientes donde se desplegó el pipeline.
- `% de clientes que les serviría`: estimación de cobertura potencial en el portafolio de Pragma.
- `Eficiencia Operativa`: reducción de tiempo comprobada vs. fuerza bruta (mínimo 50%).
- `Margen de rentabilidad`: calculado con COCOMO III (meta > 60%).
- `Tiempo de entrega SOPP`: días desde inicio hasta aprobación del cliente (meta < 5 días).
- `Uso recurrente SOPP`: número de Data Engineers que usan el JTBD al menos una vez por semana.

---

### Tendencias 2026 — Evolución del Agente de Ingestión

El Data Engineer debe estar preparado para adaptar el agente a las siguientes tendencias que impactarán la ingeniería de datos en 2026:

| Tendencia | Impacto Tecnológico | Rol del Agente |
|-----------|--------------------|--------------| 
| Real-time Data | Kafka / Flink como protocolos estándar | Generación de conectores CDC automáticos |
| Data Mesh | Propiedad de datos descentralizada | Agente como facilitador de "Data Contracts" entre dominios |
| Lakehouse Architecture | Convergencia de Data Warehouse y Data Lake | Optimización de tablas Delta y linaje de datos |
| Quantum Computing | Impacto futuro en procesamiento masivo | Monitoreo de algoritmos de optimización |
| SLM + Cuantización | Modelos pequeños en Edge Computing | Inferencia de esquema de bajo costo en el borde |

**Implicaciones para el Chapter de Datos:**
- Preparar variantes del agente para streaming CDC (Kafka/Flink) como próximo JTBD prioritario.
- Desarrollar capacidad de "Data Contracts" para entornos Data Mesh.
- Evaluar el uso de SLMs cuantizados para reducir costos de inferencia en clientes con alto volumen de fuentes nuevas.
