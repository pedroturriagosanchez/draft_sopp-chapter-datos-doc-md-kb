# JTBD: Agente IA de Ingestión Automatizada — Arquitectura Agéntica de Alto Rendimiento (Pre-Act + MLOps 2.0)

## Descripción y Dolor

Los equipos de ingeniería de datos de Pragma y sus clientes enfrentan un problema estructural conocido como **Toil**: trabajo manual, repetitivo, automatizable y sin valor perdurable que escala linealmente con el crecimiento del volumen de datos. En el contexto de la ingestión Bronze, este Toil se manifiesta en:

- Creación manual de pipelines de ingesta para cada nueva fuente de datos (4-16 horas por dataset).
- Corrección reactiva de errores de esquema (Schema Drift) que se detectan tarde, cuando ya contaminaron capas superiores.
- Codificación repetitiva de validaciones básicas para cientos de fuentes heterogéneas.
- Silos de conocimiento donde solo un ingeniero entiende un script bespoke, creando un "Engineering Bottleneck".

En organizaciones que integran 10 o más fuentes nuevas al mes, esto representa hasta **160 horas hombre/mes** en tareas de bajo valor. El "Time-to-Insight" (tiempo desde la identificación de la fuente hasta la disponibilidad de analítica) se extiende entre 1 y 2 semanas, retrasando la toma de decisiones del negocio.

Este JTBD documenta la capacidad avanzada del Chapter de Datos para resolver este problema mediante un **Agente de IA con arquitectura Pre-Act**, validación continua de datos (MLOps 2.0) y generación de código bajo el patrón de Arquitectura Hexagonal, logrando una reducción del **80% del Toil de ingeniería** y un Time-to-Insight de menos de 15 minutos de configuración activa.

## Contexto

Esta solución aplica a organizaciones que operan un **Data Lakehouse** sobre AWS S3 (o almacenamiento equivalente) con arquitectura Medallion (Bronze/Silver/Gold). Es ideal para:

- Empresas en etapa de crecimiento de su plataforma de datos con backlog creciente de ingesta.
- Organizaciones que buscan escalar su Data Lake sin escalar linealmente su equipo técnico.
- Clientes que quieren demostrar valor tangible de IA aplicada a ingeniería de datos (caso piloto de alto impacto).
- Equipos que han sufrido incidentes de Silent Drift o Schema Drift en producción y buscan una solución preventiva.

**Perfiles afectados:**
- **Data Engineers:** liberados del Toil para enfocarse en arquitectura de valor (capas Silver/Gold).
- **Data Analysts:** acceso a datos frescos en horas en lugar de semanas.
- **Business Owners / CDO:** toma de decisiones basada en datos actualizados y confiables.
- **Equipos de ML:** modelos entrenados con datos de calidad garantizada, sin Silent Drift.

## Alcance

- **Incluye:**
  - Reconocimiento automático de formato, encoding y estructura de archivos (CSV, JSON, Parquet, XML, ORC).
  - Inferencia y normalización de esquema con mapeo de tipos al estándar de la plataforma.
  - Generación de scripts PySpark modulares bajo Arquitectura Hexagonal (Puertos y Adaptadores).
  - Generación de DDL para registro en catálogo de datos (Glue Data Catalog, Unity Catalog).
  - Implementación de las **4 Puertas de Calidad MLOps 2.0**: Estructural, Semántica, Temporal y Distribucional.
  - Generación de contrato de datos en formato YAML.
  - Auto-documentación: Data Dictionary en Markdown con descripción de columnas, tipos, origen y frecuencia.
  - Validación sintáctica del código generado mediante análisis de logprobs (detección de alucinaciones).
  - Columnas de auditoría estándar SOPP: `_ingestion_timestamp`, `_source_file`, `_batch_id`, `_schema_version`.
  - Monitoreo continuo de Schema Drift mediante métricas KS y PSI.

- **No incluye:**
  - Transformaciones de negocio hacia capas Silver o Gold.
  - Definición de reglas de calidad de datos complejas (responsabilidad de Data Governance).
  - Despliegue autónomo a producción sin aprobación humana explícita.
  - Fuentes en tiempo real / streaming (aplica para JTBD de streaming CDC).
  - Modelado dimensional o diseño de Data Warehouse.

## Cuándo aplica

- Cuando el equipo recibe una nueva fuente de datos en S3 que debe incorporarse al catálogo Bronze.
- Cuando hay un backlog de ingesta que retrasa proyectos analíticos o de ML.
- Cuando se requiere estandarizar la calidad y nomenclatura de los esquemas Bronze en toda la organización.
- Cuando el cliente ha sufrido incidentes de Schema Drift o Silent Drift en producción.
- Cuando se quiere demostrar ROI de IA en ingeniería de datos en menos de 5 días hábiles.
- Cuando el equipo de Data Engineering dedica más del 40% de su tiempo a tareas de ingesta manual.

## Cuándo no aplica

- Fuentes de datos en tiempo real con latencia sub-segundo (usar arquitecturas de streaming con Kafka/Flink).
- Archivos con estructuras altamente anidadas (> 3 niveles) sin posibilidad de pre-procesamiento.
- Clientes sin equipo técnico in-house que pueda revisar y aprobar el código generado.
- Cuando el cliente ya tiene un proceso de ingestión automatizado y maduro con cobertura > 90%.
- Cuando los requisitos de seguridad del cliente no permiten que el agente acceda a datos reales (usar datos sintéticos en su lugar).

## Entradas (Inputs)

### Entradas Transversales (Estándares SOPP)
- **Estándares de Arquitectura Hexagonal:** Guías de Puertos y Adaptadores para PySpark/Glue.
- **Guías de Clean Code:** Convenciones de nomenclatura, estructura de módulos y documentación.
- **Convenciones de Nomenclatura SOPP:** Formato `[environment]-[project]-[layer]-[source]`.
- **Plantillas de Contrato de Datos YAML:** Estructura estándar para definición de esquemas.

### Entradas Específicas por Ejecución
- **URI de S3:** Ruta al archivo o prefijo del bucket fuente.
- **Muestra de datos:** Mínimo 100 filas representativas en formato JSON, CSV o Parquet.
- **Contexto de negocio:** Descripción del contenido y frecuencia de actualización.
- **Estándares de nomenclatura del cliente:** Convenciones propias si existen.
- **Permisos IAM:** Credenciales de solo lectura al bucket S3 (`s3:GetObject`, `s3:ListBucket`).
- **Requisitos de seguridad:** Nivel de encriptación requerido (SSE-S3, KMS CMK), regulaciones aplicables.

## Flujo de alto nivel

### Arquitectura del Agente: Pre-Act (Planificación Multietapa)

A diferencia del paradigma ReAct (reactivo, paso a paso), este agente utiliza el marco **Pre-Act**, que genera un plan integral de ejecución multietapa desde la primera interacción. Esto permite:
- Visión completa de la complejidad del pipeline antes de ejecutar cualquier acción.
- Refinamiento continuo del plan tras cada observación.
- Mayor robustez ante errores: el agente reevalúa todo el plan, no solo el siguiente paso.
- Modelos más pequeños (ej. Llama 3.1 8B con fine-tuning) alcanzan rendimiento comparable a modelos gigantescos.

**Comparación ReAct vs Pre-Act:**

| Dimensión | ReAct (Iterativo) | Pre-Act (Planificado) |
|-----------|-------------------|----------------------|
| Génesis del Plan | Reactivo al paso anterior | Plan integral desde el inicio |
| Manejo de Contexto | Memoria de trabajo volátil | Incorporación incremental de pasos pasados y futuros |
| Robustez | Media (puede entrar en bucles) | Alta (refinamiento continuo del plan maestro) |
| Optimización | Basado en prompts extensos | Basado en fine-tuning con datasets de planificación |

### Pasos del Flujo

1. **Planificación Inicial (Pre-Act):** El agente recibe todas las entradas y genera un plan de ejecución completo con n+1 pasos secuenciales, estimando la complejidad de cada fase antes de ejecutar ninguna acción.

2. **Reconocimiento de Fuente:** El agente lee la muestra del archivo y genera el Perfil Técnico: formato, encoding, delimitadores, estructura de particionamiento, nivel de anidamiento.

3. **Inferencia y Normalización de Esquema:** Propone nombres de columnas normalizados y tipos de datos mapeados al estándar de la plataforma. Genera el contrato de datos en YAML.

4. **Generación de Artefactos bajo Arquitectura Hexagonal:**
   - Script PySpark modular con separación de adaptadores de infraestructura y lógica de negocio.
   - DDL para el catálogo de datos con metadatos completos.
   - Columnas de auditoría estándar SOPP.

5. **Implementación de las 4 Puertas de Calidad MLOps 2.0:**
   - **Estructural (BLOCK):** Cumplimiento de tipos y columnas en el contrato YAML.
   - **Semántica (WARN/BLOCK):** Reglas de negocio (fechas no futuras, precios no negativos, IDs únicos).
   - **Temporal (AUDIT):** Orden cronológico, frescura de datos (T < 24h), detección de temporal leakage.
   - **Distribucional (WARN):** Métricas KS/PSI contra línea base histórica para detectar Silent Drift.

6. **Validación Sintáctica con Logprobs:** El agente analiza las probabilidades logarítmicas de los tokens generados para identificar segmentos de baja confianza (posibles alucinaciones). Si detecta incertidumbre, activa un ciclo de Reflexión (Reflection) para corregir su propia salida.

7. **Auto-Documentación:** Genera el Data Dictionary en Markdown con descripción de columnas, tipos, origen, frecuencia y clasificación de PII.

8. **Revisión y Aprobación Humana:** El Data Engineer revisa todos los artefactos, ejecuta en staging y aprueba formalmente antes del despliegue a producción.

9. **Refinamiento del Plan (Revise Plan):** Tras cada observación, el agente reevalúa y actualiza el plan de pasos siguientes, adaptándose a hallazgos inesperados (ej. archivos más complejos de lo estimado).

## Participación del chapter

El **Chapter de Datos** lidera end-to-end esta solución:
- **Diseño y mantenimiento del agente:** Define los prompts de inferencia, los criterios de las puertas de calidad y los templates de código bajo Arquitectura Hexagonal.
- **Estándares Bronze:** Define y mantiene las convenciones de nomenclatura, tipos de datos y columnas de auditoría aplicables a todos los clientes.
- **Consultoría técnica:** Adapta el agente a las particularidades de cada plataforma (AWS Glue, Databricks, Snowflake).
- **Acompañamiento en el primer ciclo:** Acompaña la revisión y aprobación del primer pipeline en cada cliente nuevo.
- **Gestión del conocimiento:** Documenta los límites, cicatrices y decisiones técnicas en el repositorio pragma-knowledge para alimentar futuras versiones del agente.

## Dependencias

- **Chapter de Cloud/Infraestructura:** Permisos IAM, configuración de VPC Endpoints, roles de AWS Glue. Punto de interacción: Fase 0 (prerrequisitos).
- **Chapter de QA / Data Governance:** Define las reglas de calidad semántica específicas del dominio de negocio del cliente. Punto de interacción: Fase 4 (puertas de calidad).
- **Chapter de Architecture:** Valida que el patrón de ingestión sea coherente con la arquitectura Medallion definida para el cliente. Punto de interacción: revisión del DDL y estructura de particionamiento.
- **Catálogo de Datos del Cliente:** AWS Glue Data Catalog, Unity Catalog o equivalente activo.
- **Herramienta de Orquestación:** AWS Glue, Apache Airflow, Databricks Workflows o equivalente.
- **Framework de Validación:** AWS Deequ o Great Expectations para las puertas de calidad distribucionales.
- **Decisión de Estándares Bronze:** [[decisions/001-bronze-layer-standards.md]]
- **Decisión de Arquitectura Hexagonal:** [[decisions/002-hexagonal-architecture.md]]

## Salidas esperadas (Outputs)

- **Script de Ingestión:** [[outputs/bronze-ingestion-script.md]] — Pipeline PySpark/Glue modular bajo Arquitectura Hexagonal, listo para revisión y despliegue.
- **Contrato de Datos YAML:** Definición formal del esquema Bronze con tipos, restricciones y metadatos.
- **DDL del Catálogo:** Script ejecutable para crear la tabla en el catálogo con metadatos completos y tags de clasificación.
- **Data Dictionary:** [[outputs/data-dictionary.md]] — Documento Markdown con descripción de columnas, tipos, origen, frecuencia y clasificación PII.
- **Reporte de Validaciones:** Resultado de las 4 puertas de calidad MLOps 2.0 con severidades y métricas KS/PSI.
- **Suite de Pruebas Automatizadas:** [[outputs/data-quality-test-suite.md]] — Tests de validación para ejecuciones futuras del pipeline.

## Riesgos y consideraciones

- **Schema Drift:** Cambios en la estructura del archivo entre entregas pueden romper el pipeline. Mitigación: validación estructural bloqueante en cada ejecución. Ver [[limits/schema-drift-and-known-failures.md]].
- **Silent Drift:** Cambios sutiles en la distribución de datos no detectados por CI/CD estándar. Mitigación: monitoreo continuo con métricas KS/PSI. Puede degradar modelos de ML hasta un 30% antes de ser detectado manualmente.
- **Alucinaciones del LLM:** El modelo puede generar código sintácticamente correcto pero lógicamente incorrecto. Mitigación: validación con logprobs + ciclo de Reflexión + aprobación humana obligatoria.
- **Archivos altamente anidados:** JSON/XML con jerarquías > 3 niveles requieren intervención manual. Ver [[limits/schema-drift-and-known-failures.md]] — Límite 2.
- **Requisitos de seguridad del cliente:** Clientes regulados (PCI DSS, HIPAA, GDPR) requieren configuraciones adicionales no generadas automáticamente. Ver [[limits/schema-drift-and-known-failures.md]] — Límite 6.
- **Vendor Lock-in:** El uso de servicios específicos de AWS (Glue, Lake Formation) puede generar dependencia. Mitigación: Arquitectura Hexagonal garantiza que la lógica de negocio sea portable entre plataformas.

## KPIs y Métricas de Éxito (JTBD Tracker)

| KPI | Meta | Método de Medición |
|-----|------|-------------------|
| Eficiencia Operativa | > 70% reducción en horas-hombre vs. proceso manual | Comparación de horas registradas antes/después |
| Eliminación de Toil | > 80% del Toil de ingeniería eliminado | % de fuentes integradas con el agente vs. manualmente |
| Margen de Rentabilidad | > 60% en proyectos desde cero | Modelo COCOMO III aplicado al proyecto |
| SOPP Delivery Time | < 5 días hábiles desde inicio hasta aprobación del cliente | Registro en JTBD Tracker |
| Uso Recurrente | ≥ 1 uso por semana por Data Engineer | Métricas de uso del SOPP |
| Time-to-Insight | < 15 minutos de configuración activa | Tiempo registrado en el pipeline |

## Variantes comunes

- **Versión PoC / Hackathon:** Alcance reducido a inferencia de esquema y generación de DDL, sin script de ingestión completo ni puertas de calidad. Ideal para demostrar valor en 1-2 días.
- **Versión para clientes regulados (Banking/Health):** Incluye configuración de KMS CMK, VPC Endpoints, pistas de auditoría CloudTrail y cumplimiento GDPR/HIPAA. Mayor tiempo de configuración (hasta 2 días adicionales).
- **Versión con streaming CDC:** Extensión del agente para fuentes en tiempo real usando Kafka/Flink como protocolo estándar y generación de conectores CDC automáticos.
- **Versión Data Mesh:** Adaptación para entornos donde cada unidad de negocio es "dueña" de sus datos, con el agente como facilitador de "Data Contracts" entre dominios.
- **Traducción de ETLs Legacy (SQL → PySpark):** Variante orientada a migración de Stored Procedures y scripts SQL On-Premise hacia PySpark. Mayor complejidad, menor frecuencia. Prioridad 2 en el JTBD Tracker.

## Relación con otros chapters

- **Chapter de Cloud:** Configura permisos IAM, roles de Glue y VPC Endpoints. Punto de interacción: Fase 0 (prerrequisitos) y revisión de código para clientes regulados.
- **Chapter de QA / Data Governance:** Define las reglas semánticas de las puertas de calidad. Punto de interacción: diseño de las validaciones semánticas y distribucionales.
- **Chapter de Architecture:** Valida la coherencia con la arquitectura Medallion del cliente y el uso correcto del patrón Hexagonal. Punto de interacción: revisión del DDL y estructura del pipeline.
- **Chapter de Data Science:** Consume los datos Bronze generados para entrenamiento de modelos. Punto de interacción: definición de requisitos de calidad para evitar Silent Drift en modelos de ML.
- **Chapter de Data Analytics:** Consume los datos Bronze para analítica descriptiva. Punto de interacción: validación de que el esquema Bronze es adecuado para las consultas analíticas previstas.

## Modelado Económico — ROI y COCOMO III

La implementación del agente no es un fin en sí mismo, sino una palanca financiera. El Chapter de Datos debe documentar el impacto económico de cada despliegue usando el modelo COCOMO III.

### Parámetros de Estimación

El modelo COCOMO (Constructive Cost Model) estima el costo de software basado en KLOC (miles de líneas de código) y multiplicadores de esfuerzo (EAF). El agente reduce el KLOC escrito por humanos en un **80%**, optimizando los siguientes conductores de costo:

- **Menor complejidad del producto:** El código generado sigue patrones estándar (Arquitectura Hexagonal), reduciendo la complejidad percibida.
- **Mayor capacidad del equipo:** Los Data Engineers liberados del Toil pueden enfocarse en tareas de mayor valor, mejorando la productividad efectiva del equipo.
- **Reducción de deuda técnica:** El código generado sigue estándares de Clean Code y está documentado automáticamente, reduciendo el costo de mantenimiento futuro.

### Fórmula de Ahorro Estimado

```
Horas ahorradas/mes = Fuentes nuevas/mes × Horas promedio/fuente × % reducción del agente
Ejemplo: 10 fuentes × 10 horas × 80% = 80 horas/mes ahorradas

Costo ahorrado/mes = 80 horas × tarifa hora Data Engineer
ROI 3 años = (Costo ahorrado acumulado - Costo implementación agente) / Costo implementación agente
```

### Benchmarks de Mercado (2024-2026)

| Herramienta / Plataforma | ROI Reportado (3 años) | Impacto en Ingeniería |
|--------------------------|----------------------|----------------------|
| Informatica Cloud | 328% | Reducción de complejidad de integración |
| Snowflake AI Data Cloud | 354% | 50% menos esfuerzo; 75% más rápido Time-to-Insight |
| AWS Glue (Migración) | 80% ahorro en costo | Reducción de $500 a $100 por TB procesado |
| Agente Ingesta Bronze (SOPP) | > 70% ahorro horas-hombre | Eliminación del Toil en 80%; entrega < 5 días |

---

## Campos del JTBD Tracker — Registro Obligatorio

Cada ejecución de este JTBD debe registrarse en el JTBD Tracker con los siguientes campos:

| Campo | Descripción | Meta / Valor Esperado |
|-------|-------------|----------------------|
| `jtbd-file-name` | Nombre del archivo JTBD ejecutado | `ai-agent-bronze-ingestion-advanced.md` |
| `Responsable` | Data Engineer que lidera la ejecución | [Nombre del responsable] |
| `Estado` | Fase actual del JTBD | Propuesto / En Ejecución / Completado / Disponibilizado |
| `Título del Job` | Nombre descriptivo del trabajo ejecutado | Ingestión Bronze para [nombre fuente] en [cliente] |
| `Descripción y Valor de Negocio` | Problema resuelto y valor generado | Reducción de X horas a < 15 min de configuración |
| `Entradas (Inputs)` | Recursos provistos para la ejecución | URI S3, muestra, contexto de negocio |
| `Salidas (Outputs)` | Artefactos entregados | Script PySpark, DDL, Data Dictionary, contrato YAML |
| `Dependencias` | Chapters y sistemas requeridos | Cloud (IAM), QA (reglas semánticas), Architecture |
| `Arquitectura de referencia` | Patrón técnico utilizado | Arquitectura Hexagonal + Pre-Act + MLOps 2.0 |
| `Especificaciones de capas adicionales para clientes` | Personalizaciones por cliente | KMS CMK, VPC Endpoints, regulaciones específicas |
| `Cuentas impactando` | Clientes donde se desplegó | [Lista de clientes] |
| `% de clientes que les serviría` | Cobertura potencial en portafolio Pragma | Estimado > 70% de clientes con Data Lake en S3 |
| `Eficiencia Operativa` | Reducción de tiempo vs. fuerza bruta | > 70% (mínimo requerido: 50%) |
| `Margen de rentabilidad` | Para proyectos desde cero | > 60% (calculado con COCOMO III) |
| `Tiempo de entrega SOPP` | Días desde inicio hasta aprobación del cliente | < 5 días hábiles |
| `Uso recurrente SOPP` | Data Engineers que usan el JTBD ≥ 1 vez/semana | Meta: ≥ 3 Data Engineers/semana |

---

## Tendencias 2026 — Evolución del JTBD

El Chapter de Datos debe anticipar las siguientes tendencias que impactarán la evolución de este JTBD:

| Tendencia 2026 | Impacto en el JTBD | Acción Recomendada |
|----------------|-------------------|-------------------|
| Real-time Data (Kafka/Flink) | El agente debe generar conectores CDC para fuentes en streaming | Desarrollar variante de JTBD para streaming CDC |
| Data Mesh | Cada dominio de negocio será dueño de sus datos; el agente debe facilitar "Data Contracts" | Adaptar el agente para operar en entornos descentralizados |
| Lakehouse Architecture | Convergencia de Data Warehouse y Data Lake; optimización de tablas Delta | Incluir generación de tablas Delta con linaje de datos |
| SLM + Cuantización 4 bits | Modelos pequeños para reducir costos en clientes con alto volumen | Evaluar fine-tuning de SLMs con datos de Pragma |
| Quantum Computing | Impacto futuro en procesamiento masivo de datos | Monitorear evolución para casos de uso de optimización |

**Empresas que automatizan gestión de datos para 2026:** El 78% de empresas en eCommerce planea automatizar la gestión de inventarios y datos para evitar pérdidas por falta de información en tiempo real. Las empresas que usan análisis avanzado para optimizar sus lead times reducen costos operativos hasta en un 20%.

## Referencias relacionadas

- JTBD base (versión simplificada): [[business-solutions/standard/jtbd/bronze-ingestion-agent.md]]
- Límites conocidos del agente: [[limits/schema-drift-and-known-failures.md]]
- Guía procedimental de ejecución: [[references/bronze-ingestion-procedural-guide.md]]
- Decisión de estándares Bronze: [[decisions/001-bronze-layer-standards.md]]
- Decisión de Arquitectura Hexagonal: [[decisions/002-hexagonal-architecture.md]]
- Convenciones de nomenclatura SOPP: [[references/sopp-naming-conventions.md]]
