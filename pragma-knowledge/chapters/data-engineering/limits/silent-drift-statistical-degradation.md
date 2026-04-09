# Límite: Silent Drift — Degradación Estadística No Detectada por CI/CD

## Contexto de Identificación

Identificado en proyectos de ingesta y analítica avanzada dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma) donde modelos de ML (Machine Learning) consumen datos de la capa Bronze para entrenamiento e inferencia. A diferencia del Schema Drift (cambio estructural en columnas o tipos), el Silent Drift opera sobre la **distribución estadística** de los datos sin alterar su estructura, lo que lo hace invisible para las validaciones estructurales y semánticas convencionales de CI/CD (Continuous Integration / Continuous Delivery — Integración Continua / Entrega Continua).

El problema se manifiesta en fuentes de datos con alta variabilidad estacional o de negocio:

- Datos transaccionales del sector financiero donde el comportamiento de usuarios cambia por campañas de marketing, estacionalidad o nuevos productos.
- Fuentes de datos de comercios donde la incorporación o baja de comercios altera la distribución de categorías.
- APIs (Application Programming Interfaces) de terceros que modifican el rango o la granularidad de los valores retornados sin cambiar el esquema.

Se ha observado que el Silent Drift puede degradar modelos de ML hasta un 30% en métricas de precisión antes de ser detectado manualmente, según las referencias del marco MLOps 2.0 (Machine Learning Operations — Operaciones de Aprendizaje Automático).

## La Restricción

El **Silent Drift** es el cambio gradual o abrupto en las propiedades estadísticas de los datos (media, varianza, cardinalidad, distribución de frecuencias, rango de valores) que no es detectado por los pipelines de CI/CD (Continuous Integration / Continuous Delivery) estándar porque la estructura del archivo permanece idéntica al contrato de datos.

Ejemplos concretos:

- Una columna de `monto_transaccion` que históricamente tenía una media de $50.000 COP (Pesos Colombianos) empieza a reportar una media de $150.000 COP porque el sistema origen incorporó transacciones internacionales sin diferenciarlas.
- Una columna categórica de `tipo_comercio` que tenía 15 categorías pasa a tener 45 porque se incorporaron nuevos verticales de negocio.
- La proporción de nulos en una columna de `email` pasa del 5% al 60% porque un segmento de usuarios dejó de proporcionar esa información.

Estas variaciones son **estadísticamente significativas** pero **estructuralmente invisibles**: los tipos de datos no cambian, las columnas no se agregan ni se eliminan, y los valores individuales pueden ser perfectamente válidos según las reglas semánticas. La Puerta Estructural de MLOps 2.0 no los detecta; solo la **Puerta Distribucional** puede hacerlo.

## Evidencia e Impacto

Si se ignora este límite:

- **Degradación silenciosa de modelos de ML:** Los modelos entrenados con datos de distribución A reciben datos de distribución B en producción. La precisión se degrada progresivamente sin generar un error o alerta, porque las predicciones siguen siendo numéricamente válidas — simplemente son incorrectas.
- **Decisiones de negocio basadas en datos sesgados:** Reportes analíticos que muestran tendencias aparentemente normales pero que reflejan una distribución subyacente fundamentalmente diferente. En el sector financiero, esto puede derivar en políticas de crédito o detección de fraude calibradas sobre supuestos estadísticos obsoletos.
- **Detección tardía:** Sin monitoreo distribucional activo, el Silent Drift se detecta típicamente cuando un usuario de negocio nota resultados "extraños" en un reporte o cuando la precisión de un modelo cae por debajo de un umbral visible. Para ese momento, pueden haber pasado semanas o meses de datos contaminados en la capa Bronze.
- **Costo de reentrenamiento:** Corregir un modelo afectado por Silent Drift requiere identificar desde cuándo se produjo la desviación, reprocesar los datos desde esa fecha y reentrenar el modelo. El costo se multiplica si múltiples modelos consumen la misma fuente.

## Señales de Alerta

- Cambio significativo (> 2 desviaciones estándar) en la media o varianza de columnas numéricas respecto a la línea base histórica calculada en las últimas N ejecuciones.
- Aumento o disminución abrupta en la cardinalidad de columnas categóricas (ej. la columna `tipo_comercio` pasa de 15 a 45 valores distintos en una sola carga).
- Métricas KS (Kolmogorov-Smirnov) o PSI (Population Stability Index) que superan el umbral de tolerancia configurado. Valores de referencia: PSI > 0.2 indica un cambio significativo que requiere investigación; PSI > 0.25 indica degradación severa.
- Cambio en la proporción de nulos de una columna que supera el porcentaje histórico por más de 10 puntos porcentuales.
- Alertas de la Puerta Distribucional (severidad WARN) del marco MLOps 2.0, que compara las estadísticas de cada carga contra la línea base. [[references/bronze-ingestion-procedural-guide.md]]

## Workarounds y Mitigaciones

- **Implementar la Puerta Distribucional de MLOps 2.0:** Configurar validación distribucional con severidad WARN en cada ejecución del pipeline de ingesta Bronze. Calcular métricas KS y PSI para todas las columnas numéricas y categóricas, comparando contra una línea base histórica que se recalcula periódicamente (ej. cada 30 días). [[decisions/001-bronze-layer-standards.md]]
- **Definir umbrales de tolerancia por columna:** No todas las columnas tienen la misma sensibilidad. Columnas críticas para modelos de ML (ej. `monto_transaccion`, `tipo_comercio`) deben tener umbrales más estrictos (PSI < 0.1) que columnas informativas (PSI < 0.25).
- **Alertas diferenciadas por severidad:** WARN para desviaciones moderadas que requieren investigación pero no detienen el pipeline. BLOCK para desviaciones extremas (PSI > 0.5) que indican un cambio fundamental en la fuente que debe ser investigado antes de permitir la ingesta.
- **Dashboard de monitoreo distribucional:** Mantener un tablero de gobernanza con las métricas KS/PSI históricas de cada fuente, visible para el Data Steward y el Data Engineer responsable.
- **Comunicación con el equipo de ML:** Cuando se detecta Silent Drift en una columna consumida por modelos de ML, notificar inmediatamente al equipo de Data Science para que evalúe el impacto en los modelos activos y decida si es necesario reentrenar.
- **Línea base dinámica:** Recalcular la línea base histórica de forma periódica (no estática) para que las variaciones estacionales legítimas no generen falsos positivos. El periodo de recálculo debe ser acordado con el cliente. [[business-solutions/standard/jtbd/ai-agent-bronze-ingestion.md]]
