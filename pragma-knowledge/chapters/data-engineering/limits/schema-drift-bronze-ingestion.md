# Límite: Schema Drift en Ingesta de Capa Bronze

## Contexto de Identificación

Identificado en proyectos de ingesta Bronze dentro del ecosistema SOPP (Sistema Operativo de Potencia Pragma) con clientes del sector financiero que operan arquitecturas Medallón (Bronze / Silver / Gold) sobre AWS S3 (Simple Storage Service). El problema se manifiesta de manera recurrente en fuentes de datos que provienen de sistemas transaccionales legacy, APIs (Application Programming Interfaces) de terceros y archivos generados por procesos batch de entidades bancarias.

El Schema Drift ocurre con mayor frecuencia en los siguientes escenarios:

- Fuentes que incorporan nuevos comercios o medios de pago (ej. clientes como procesadores de pagos donde cada nuevo comercio puede agregar columnas al archivo de transacciones).
- APIs de terceros que versionan su contrato sin aviso previo al consumidor del dato.
- Archivos CSV (Comma-Separated Values) o JSON (JavaScript Object Notation) generados por sistemas legacy que no implementan contratos de datos formales.
- Migraciones de sistemas origen donde el esquema cambia entre fases del proyecto.

Se ha observado en volúmenes desde 10 hasta 500+ fuentes de datos activas por cliente, con frecuencia de cambio de esquema entre el 5% y el 15% de las fuentes por mes.

## La Restricción

El **Schema Drift** es el cambio no anunciado en la estructura de los archivos fuente entre entregas sucesivas. Se manifiesta en tres variantes:

- **Drift aditivo:** Aparecen columnas nuevas que no existían en el esquema original. Apache Spark infiere el esquema de la primera partición leída y las columnas nuevas se ignoran silenciosamente en las particiones posteriores, o generan un error de merge de esquemas si `mergeSchema` no está habilitado.
- **Drift sustractivo:** Desaparecen columnas esperadas. El pipeline falla con un `AnalysisException: cannot resolve column name` si el código referencia la columna eliminada.
- **Drift de tipo:** Una columna cambia de tipo (ej. de `integer` a `string` porque el sistema origen empezó a incluir valores alfanuméricos en un campo que antes era numérico). Spark lanza un `MatchError` o, peor aún, realiza un cast silencioso que produce valores `null` sin error visible.

Esta restricción es inherente a cualquier arquitectura de ingesta que consuma fuentes externas sin contrato de datos formalizado en YAML (YAML Ain't Markup Language) o JSON Schema. No es un bug del agente ni de Spark — es una propiedad del ecosistema de datos del cliente.

## Evidencia e Impacto

Si se ignora este límite:

- **Contaminación de capas superiores:** Los datos con esquema incorrecto llegan a Bronze y se propagan a Silver y Gold antes de ser detectados. La corrección retroactiva puede requerir reprocesamiento completo de las capas afectadas, con un costo estimado de 8 a 40 horas-hombre dependiendo del volumen y la cantidad de transformaciones dependientes.
- **Rotura de modelos de ML (Machine Learning):** Los modelos entrenados sobre datos de capas superiores reciben features (variables predictivas) con distribuciones alteradas o columnas faltantes, degradando su precisión sin generar un error explícito.
- **Pérdida de confianza del cliente:** En el sector financiero, un reporte analítico con datos incorrectos puede derivar en decisiones de negocio erróneas y en la pérdida de credibilidad del equipo de datos ante el CDO (Chief Data Officer).
- **Incremento del Toil:** Cada incidente genera trabajo manual de investigación, corrección y reprocesamiento que escala linealmente con el número de fuentes afectadas.

La causa raíz no es técnica sino organizacional: la falta de contratos de datos formalizados entre el productor y el consumidor del dato.

## Señales de Alerta

- Logs de Spark con warnings de merge de esquema: mensajes como `Found conflicting types for column X` o `mergeSchema is enabled, adding new columns`.
- Incremento súbito de valores `null` en una columna que históricamente tenía baja nulidad (ej. salta de 2% a 40% en una sola ejecución), indicando un cast silencioso por cambio de tipo.
- Fallo del pipeline con `AnalysisException: cannot resolve column name`, indicando Drift sustractivo.
- Cambio en el conteo de columnas del archivo fuente respecto al conteo registrado en el contrato YAML.
- Alertas de la Puerta Estructural (severidad BLOCK) del marco MLOps 2.0 (Machine Learning Operations — Operaciones de Aprendizaje Automático), que detecta discrepancias entre el esquema inferido y el contrato de datos. [[references/bronze-ingestion-procedural-guide.md]]

## Workarounds y Mitigaciones

- **Validación estructural bloqueante en cada ejecución:** Implementar la Puerta Estructural de MLOps 2.0 con severidad BLOCK. Antes de procesar cualquier archivo, el agente compara el esquema inferido contra el contrato YAML registrado. Si hay discrepancias, el pipeline se detiene y genera una alerta al Data Engineer responsable. [[decisions/001-bronze-layer-standards.md]]
- **Contrato de datos YAML como fuente de verdad:** Para cada fuente de datos, formalizar un contrato YAML que defina tipos, columnas obligatorias y rangos válidos. Este contrato debe ser versionado junto con el pipeline.
- **Modo `mergeSchema` controlado:** Habilitar `mergeSchema` de Spark únicamente cuando el Drift aditivo es esperado y documentado, nunca como configuración por defecto. Registrar cada activación como un evento de auditoría.
- **Columna de auditoría `_schema_version`:** Incluir esta columna en cada registro Bronze para permitir trazabilidad del esquema con el que se ingirió cada lote de datos.
- **Alerta proactiva al equipo del cliente:** Establecer un canal de comunicación con el equipo productor del dato para que notifique cambios de esquema antes de la siguiente entrega.
- **Escalamiento al Data Engineer Senior:** Cuando el Drift involucra cambios de tipo en columnas críticas (claves primarias, montos financieros), escalar al Tech Lead de Datos para decisión de reprocesamiento. [[business-solutions/standard/jtbd/ai-agent-bronze-ingestion.md]]
