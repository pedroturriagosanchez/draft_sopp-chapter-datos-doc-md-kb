# Límite: Restricciones, Fallos Conocidos y Cicatrices del Agente de Ingestión Bronze

## Contexto de Identificación

Estas restricciones fueron identificadas durante la implementación y operación del Agente de Ingestión Automatizada en la Capa Bronze (Cruda) del Data Lakehouse de Pragma. Aplican a entornos que utilizan AWS S3 como almacenamiento, AWS Glue / Apache Spark como motor de procesamiento, y arquitecturas Medallion (Bronze/Silver/Gold). Las cicatrices documentadas aquí representan fallos reales o riesgos comprobados en proyectos de ingeniería de datos con volúmenes de 10 o más fuentes nuevas por mes.

---

## Límite 1: Schema Drift (Deriva de Esquema)

### La Restricción
El agente infiere el esquema a partir de una muestra del archivo en el momento de la configuración inicial. Si el proveedor de datos modifica la estructura del archivo en entregas posteriores (agrega columnas, cambia tipos de datos, renombra campos), el pipeline generado falla silenciosamente o produce datos corruptos en la capa Bronze sin generar alertas visibles en los logs estándar.

Este fenómeno se conoce como **Silent Drift**: cambios sutiles en la distribución o estructura de los datos que no son detectados por los sistemas de CI/CD tradicionales, causando degradación progresiva del rendimiento de modelos de ML en producción.

### Evidencia e Impacto
- Un campo que cambia de `String` a `Integer` provoca un `AnalysisException` en Spark que detiene el job completo.
- La adición de columnas no declaradas en el contrato YAML genera registros con valores `null` en las columnas nuevas, contaminando la capa Silver sin trazabilidad.
- En modelos de ML que consumen datos Bronze, el Silent Drift puede degradar la precisión del modelo hasta un 30% antes de ser detectado manualmente.

### Señales de Alerta
- Incremento súbito en el número de registros rechazados por el pipeline de validación.
- Aparición de columnas con 100% de valores `null` en tablas que históricamente tenían datos completos.
- Cambios en las métricas de distribución (media, varianza) detectados por herramientas como Great Expectations o AWS Deequ.
- Alertas de métricas KS (Kolmogorov-Smirnov) o PSI (Population Stability Index) que superan el umbral definido en la política de CDV (Continuous Data Validation).

### Workarounds y Mitigaciones
1. **Implementar validación estructural bloqueante (Block):** Configurar el agente para comparar el esquema de cada entrega contra el contrato YAML registrado. Si hay desviación, el pipeline se detiene antes de escribir en Bronze.
2. **Activar modo `mergeSchema` en Delta Lake:** Permite absorber columnas nuevas sin romper el pipeline, registrando la evolución del esquema en el catálogo.
3. **Versionado de contratos de datos:** Cada cambio de esquema debe generar una nueva versión del contrato YAML y un ADR documentando la decisión.
4. **Alertas distribucionales automáticas:** Integrar AWS Deequ o Great Expectations para monitorear métricas KS/PSI en cada ejecución del pipeline.

---

## Límite 2: Archivos con Estructuras Altamente Anidadas

### La Restricción
El agente tiene capacidad limitada para inferir esquemas de archivos JSON o XML con jerarquías de anidamiento superiores a 3 niveles de profundidad. La inferencia automática tiende a colapsar estructuras complejas en columnas de tipo `StructType` o `ArrayType` sin desanidar correctamente, generando esquemas Bronze inutilizables para analítica directa.

### Evidencia e Impacto
- Archivos JSON de APIs REST con objetos anidados (ej. `order.customer.address.city`) requieren lógica de aplanamiento (`explode`, `flatten`) que el agente no genera automáticamente.
- El procesamiento de XML con namespaces complejos falla en el parser de Spark con errores `SAXParseException`.
- El tiempo de inferencia se incrementa exponencialmente con la profundidad del anidamiento, pudiendo superar el timeout de AWS Glue (48 horas).

### Señales de Alerta
- Presencia de columnas de tipo `StructType` o `ArrayType` en el esquema inferido que deberían ser campos planos.
- Tiempo de ejecución del job de inferencia superior a 30 minutos para archivos menores a 1 GB.
- Errores `SparkException: Task failed while writing rows` en los logs de Glue.

### Workarounds y Mitigaciones
1. **Intervención manual del Data Engineer:** Para archivos con anidamiento > 3 niveles, el ingeniero debe definir manualmente la lógica de aplanamiento antes de ejecutar el agente.
2. **Pre-procesamiento con Lambda:** Implementar una función AWS Lambda que aplane el JSON antes de que el agente lo procese.
3. **Uso de `schema hints` en el contrato YAML:** Proveer al agente un esquema parcial como guía para los campos más complejos.

---

## Límite 3: Dependencia Crítica de Permisos IAM en S3

### La Restricción
El agente requiere permisos de lectura (`s3:GetObject`, `s3:ListBucket`) sobre el bucket S3 del cliente antes de poder ejecutar cualquier operación. La ausencia o configuración incorrecta de estos permisos bloquea completamente la fase de reconocimiento de fuente, impidiendo incluso la generación del esquema.

### Evidencia e Impacto
- Errores `AccessDeniedException` en la fase de muestreo detienen el pipeline en el primer paso sin generar ningún artefacto.
- En entornos con AWS Lake Formation habilitado, los permisos IAM estándar son insuficientes — se requieren permisos adicionales a nivel de tabla en el catálogo.
- La configuración de VPC Endpoints para S3 privado puede bloquear el acceso desde AWS Glue si no se configura correctamente la política de endpoint.

### Señales de Alerta
- Error `com.amazonaws.services.s3.model.AmazonS3Exception: Access Denied` en los logs del job.
- El agente completa la fase de planificación (Pre-Act) pero falla en el primer paso de ejecución.
- Timeout en la conexión al bucket sin mensaje de error explícito (indica problema de red/VPC, no de permisos).

### Workarounds y Mitigaciones
1. **Checklist de permisos previo al inicio:** Verificar `s3:GetObject`, `s3:ListBucket`, y permisos de Lake Formation antes de iniciar el JTBD.
2. **Rol IAM dedicado para el agente:** Crear un rol específico con permisos de solo lectura, evitando el uso de roles con permisos amplios.
3. **Script de validación de acceso:** Ejecutar un script de verificación de permisos como primer paso del pipeline antes de cualquier operación de inferencia.

---

## Límite 4: Aprobación Humana Obligatoria — No Despliegue Autónomo

### La Restricción
El agente **no puede** ejecutar código en producción de forma autónoma. Todo script PySpark o DDL generado requiere revisión y aprobación explícita del Data Engineer responsable antes de la primera ejecución en producción. Este es un límite de diseño intencional, no una limitación técnica.

### Evidencia e Impacto
- El código generado por LLMs puede contener errores de lógica de negocio que pasan la validación sintáctica pero producen resultados incorrectos (alucinaciones de lógica).
- La ejecución autónoma en producción sin revisión humana viola los principios de gobernanza de datos de Pragma y los requisitos de cumplimiento de clientes regulados (GDPR, HIPAA, regulaciones financieras).

### Señales de Alerta
- Cualquier intento de configurar el agente en modo "auto-deploy" debe ser bloqueado por el Encargado de Validación.
- Pipelines que se ejecutan sin registro de aprobación en el sistema de auditoría.

### Workarounds y Mitigaciones
1. **Gate de aprobación en el pipeline de CI/CD:** Implementar un paso de aprobación manual obligatorio antes del despliegue a producción.
2. **Registro de auditoría:** Cada aprobación debe quedar registrada con el nombre del Data Engineer, fecha y versión del esquema aprobado.
3. **Entorno de staging obligatorio:** Todo código generado debe ejecutarse primero en un entorno de desarrollo/staging con datos sintéticos antes de la aprobación para producción.

---

## Límite 5: Escalabilidad Lineal del Toil sin Automatización

### La Restricción
Sin el agente, el esfuerzo de ingesta escala de forma **lineal** con el número de fuentes de datos. Cada nueva fuente requiere entre 4 y 16 horas de trabajo manual de un Data Engineer. En organizaciones que integran 10 o más fuentes nuevas por mes, esto representa hasta 160 horas hombre/mes en tareas de bajo valor agregado, asfixiando la capacidad de innovación del equipo.

### Evidencia e Impacto
- El modelo COCOMO III confirma que la "carpintería de código" manual incrementa el KLOC (miles de líneas de código) y los factores de complejidad, elevando el costo estimado del proyecto.
- El "Time-to-Insight" (tiempo desde la identificación de la fuente hasta la disponibilidad de analítica) se extiende entre 1 y 2 semanas por fuente nueva.
- Los Data Engineers quedan atrapados en rol de "plomeros de archivos" en lugar de arquitectos de valor (capas Silver/Gold).

### Señales de Alerta
- Backlog de ingesta con más de 5 fuentes pendientes de integración.
- Data Engineers dedicando más del 40% de su tiempo a tareas de ingesta manual.
- Proyectos analíticos bloqueados esperando disponibilidad de datos en Bronze.

### Workarounds y Mitigaciones
1. **Implementar el agente como capacidad estándar:** El JTBD de ingestión Bronze debe ser la primera capacidad desplegada del SOPP para liberar capacidad del equipo.
2. **Priorización por impacto:** Usar la matriz de priorización del JTBD Tracker (Impacto Cliente / Facilidad Técnica) para ordenar el backlog de ingesta.
3. **KPI de seguimiento:** Medir semanalmente el porcentaje de fuentes integradas con el agente vs. manualmente para demostrar el ROI.

---

## Límite 6: Estándares de Seguridad Específicos del Cliente

### La Restricción
El agente genera código con configuraciones de seguridad estándar (encriptación en tránsito con TLS, encriptación en reposo con SSE-S3). Sin embargo, clientes con requisitos regulatorios específicos (entidades financieras, sector salud) pueden requerir configuraciones adicionales como KMS Customer Managed Keys (CMK), VPC Endpoints privados, o políticas de retención de datos específicas que el agente no configura automáticamente.

### Evidencia e Impacto
- Clientes bajo regulación PCI DSS requieren encriptación con KMS CMK, no SSE-S3 estándar.
- Entidades bajo HIPAA requieren pistas de auditoría detalladas (AWS CloudTrail) y controles de acceso cifrados que deben configurarse manualmente.
- La omisión de estos requisitos puede resultar en incumplimiento regulatorio y sanciones para el cliente.

### Señales de Alerta
- Cliente pertenece a sector financiero, salud o gobierno.
- El contrato del cliente menciona regulaciones específicas (PCI DSS, HIPAA, GDPR, Ley 1581 de Colombia).
- El equipo de seguridad del cliente solicita revisión del código antes de la aprobación.

### Workarounds y Mitigaciones
1. **Cuestionario de seguridad previo al inicio:** Antes de ejecutar el agente, completar un checklist de requisitos de seguridad del cliente.
2. **Templates de seguridad por industria:** Mantener templates de configuración de seguridad para sectores regulados (banking, health, government) en `references/security-templates/`.
3. **Revisión conjunta con el Chapter de Cloud:** Involucrar al Chapter de Cloud/Infraestructura en la revisión del código generado para clientes regulados.

---

## Referencias Relacionadas

- JTBD del agente: [[business-solutions/standard/jtbd/bronze-ingestion-agent.md]]
- JTBD avanzado (Pre-Act + MLOps 2.0): [[business-solutions/standard/jtbd/ai-agent-bronze-ingestion-advanced.md]]
- Guía procedimental de ejecución: [[references/bronze-ingestion-procedural-guide.md]]
- Decisión de estándares Bronze: [[decisions/001-bronze-layer-standards.md]]
- Convenciones de nomenclatura SOPP: [[references/sopp-naming-conventions.md]]
