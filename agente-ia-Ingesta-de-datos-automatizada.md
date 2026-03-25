# **Marco Estratégico y Técnico para la Automatización de la Ingestión en Capa Cruda mediante Agentes de Inteligencia Artificial: Definición del Job To Be Done**

La industria de los datos atraviesa un punto de inflexión donde la capacidad de captura de información ha superado con creces la capacidad humana de procesarla y dotarla de estructura. En este contexto, las organizaciones modernas se enfrentan a un desafío crítico: el estancamiento de sus tuberías de datos en las etapas iniciales de la arquitectura de medallón, específicamente en la transición hacia la capa Cruda. Este documento detalla la propuesta estratégica y técnica para la implementación del "Agente Ingestión Cruda", una entidad autónoma basada en arquitecturas agenticas que busca resolver el cuello de botella técnico de la inferencia de esquemas y la generación de código operativo, alineando estos esfuerzos con el imperativo de negocio de reducir el "Time-to-Insight".1

La arquitectura de medallón, pilar de los Data Lakehouses contemporáneos, propone una organización de datos en tres estados: Cruda (datos crudos y fidelidad total), Silver (datos limpios, integrados y enriquecidos) y Gold (datos modelados para consumo de negocio). Sin embargo, la creación de la capa Cruda, a menudo percibida como una tarea trivial de copia de datos, se ha convertido en un ejercicio de ingeniería de software complejo y repetitivo. Los ingenieros de datos deben enfrentarse a una diversidad abrumadora de fuentes, desde bases de datos relacionales tradicionales hasta flujos de eventos en tiempo real y APIs semi-estructuradas. La necesidad de inferir esquemas cambiantes y escribir scripts de Apache Spark o AWS Glue de manera manual introduce una latencia operativa que compromete la agilidad organizacional.1

## **El Problema Estructural: Del Toil Técnico al Bloqueo de Negocio**

El concepto de "Toil" o trabajo penoso, derivado de la disciplina de Site Reliability Engineering (SRE), describe perfectamente el estado actual de las tareas de ingestión. Se trata de trabajo manual, repetitivo, automatizable y que no aporta un valor creativo incremental. En el caso de la capa Cruda, el Toil se manifiesta en la escritura de código "boilerplate" para la lectura de archivos, el manejo de excepciones de tipos de datos y la configuración de particionamiento.3

### **El Cuello de Botella Técnico: Inferencia y Generación de Código**

La inferencia de esquemas es intrínsecamente problemática en sistemas donde el "Schema Drift" (deriva de esquema) es la norma. Cuando una fuente de datos modifica un campo o añade una columna, el proceso de ingestión manual suele fallar, requiriendo la intervención de un ingeniero para ajustar el código Glue o Spark. Este proceso no solo es lento, sino que es propenso a errores de tipado que pueden corromper el Data Catalog. La generación automática de código mediante plantillas estáticas ha demostrado ser insuficiente para manejar la complejidad semántica de los datos modernos, donde se requiere un entendimiento contextual de la información para determinar si un campo numérico representa un ID, una medida monetaria o una marca de tiempo.1

La complejidad matemática de la validación de esquemas puede expresarse mediante la evaluación de la conformidad de un conjunto de datos ![][image1] respecto a un esquema formal ![][image2]. Si definimos ![][image3] como una función de validación que devuelve 1 si el dato ![][image4] cumple con la especificación ![][image5] y 0 en caso contrario, el objetivo de un agente de ingeniería de datos es maximizar la métrica de integridad ![][image6] definida como:

![][image7]  
Lograr que este valor sea igual a 1 de manera consistente requiere una lógica de razonamiento que supere la simple comparación de cadenas de texto.2

### **El Problema del Cliente: Time-to-Insight Comprometido**

Desde la perspectiva estratégica, la plataforma de datos existe únicamente para generar valor a través de la toma de decisiones informada. Cada hora que un dato permanece "atrapado" en el sistema origen sin ser ingerido en la capa Cruda es una hora de pérdida de oportunidad. El "Time-to-Insight" se define como el intervalo temporal entre la generación del evento y su disponibilidad para el análisis final. Los cuellos de botella en la ingestión Cruda expanden este intervalo de manera artificial debido a la dependencia de recursos humanos escasos y costosos.1 La automatización mediante agentes de IA no es solo una mejora técnica; es una respuesta a la necesidad competitiva de operar en tiempo real o casi real, permitiendo ciclos de descubrimiento de conocimiento más ágiles siguiendo modelos como CRISP-DM.5

## **Fundamentos de la Arquitectura Agéntica: El Marco ReAct y la Planeación Multi-paso**

Para abordar estos problemas, la propuesta se centra en un Agente de IA que no solo "sabe" sobre ingeniería de datos, sino que puede "actuar" sobre el entorno de la plataforma. La base técnica de este agente es el marco de trabajo ReAct (Reasoning and Acting), que permite al modelo generar trazas de razonamiento verbal y acciones específicas de manera intercalada.4

### **Mecanismo de Intercalado: Pensamiento, Acción y Observación**

El Agente Ingestión Cruda opera bajo un ciclo continuo que le permite manejar la incertidumbre técnica:

1. **Pensamiento (Thought):** El agente analiza el input inicial (por ejemplo, una muestra de un archivo JSON y un Swagger). Razona sobre las inconsistencias detectadas: "El campo precio aparece como string en la muestra pero como decimal en el Swagger; debo investigar la mejor estrategia de conversión para evitar pérdida de precisión".4  
2. **Acción (Action):** El agente ejecuta una herramienta, como un script de perfilado de datos o una consulta al catálogo de metadatos de AWS Glue.4  
3. **Observación (Observation):** El agente recibe el resultado de la acción: "El perfilado indica que el 99% de los registros son numéricos, pero existen caracteres especiales en el 1%; generaré código de limpieza preventivo".4

Este enfoque de "pensar antes de actuar" y "razonar sobre los resultados" reduce drásticamente la propagación de errores y las alucinaciones típicas de los LLMs cuando se les pide generar código Spark complejo en un solo intento.4

### **Planeación y Reflexión: Mejorando la Calidad del Código Generado**

La generación de código Glue/Spark no es el paso final, sino parte de un proceso iterativo de diseño. Mediante patrones de **Reflexión (Pattern 18\)**, el agente puede revisar su propio código generado frente a estándares de arquitectura hexagonal y Clean Architecture.1 Si el código inicial es demasiado monolítico o carece de manejo de excepciones, el agente se "auto-critica" y produce una versión mejorada que separa la lógica de conexión de la lógica de transformación.2

Además, la implementación de **Pre-Act** permite que el agente realice una planificación multi-paso antes de tocar cualquier entorno productivo, simulando las dependencias y validando que los permisos de IAM y las cuotas de recursos en la nube sean suficientes para la tarea de ingestión encomendada.7

## **Definición del Job To Be Done (JTBD) para el Agente Ingestión Cruda**

Siguiendo la estructura formal del "Propuesta de Tablero de Control: JTBD Tracker", se establece la definición detallada de este trabajo como una capacidad estratégica dentro de la organización. El JTBD no se limita a la descripción de una función de software, sino que captura la esencia del problema de negocio que el agente está contratado para resolver.1

### **Tabla 1: Especificación del JTBD Tracker \- Agente Ingestión Cruda**

| Columna | Definición Técnica y de Negocio |
| :---- | :---- |
| **jtbd-file-name** | jjtbd-ai-agent-data-raw-ingestion.md |
| **Responsable** | Chapter Arquitectura de Datos / IA Enablement Team |
| **Estado** | Definido / En Implementación |
| **Título del Job** | Automatización Agentica de Ingestión en Capa Cruda |
| **Descripción y Valor de Negocio** | Este Job resuelve la fricción extrema en la fase de 'Raw Data Capture'. Automatiza la inferencia de esquemas dinámicos y la generación de scripts AWS Glue/Spark optimizados. El valor reside en eliminar el 80% del Toil de ingeniería, garantizando que el dato llegue a la capa Cruda con trazabilidad, seguridad y calidad, acelerando el Time-to-Insight global.1 |
| **Entradas (Inputs)** | **Transversales:** Estándares de Arquitectura Hexagonal, librerías de seguridad corporativa. **Específicos:** Metadatos origen (Swagger, DDL), muestras de datos (JSON/CSV), contratos de datos (Data Contracts) y requisitos de particionamiento.1 |
| **Salidas (Outputs)** | \- Scripts PySpark/Glue modulares. \- Documentación técnica del esquema en Markdown. \- Pruebas unitarias de calidad de datos. \- Registro de linaje en OpenLineage. \- Trazas ReAct de la lógica de inferencia.1 |
| **Dependencias** | \- Equipo de Infraestructura (Cloud Ops). \- Product Owners de Sistemas Origen. \- Framework de Calidad de Datos (DQSOps).8 |
| **Arquitectura de referencia** | Arquitectura Hexagonal para pipelines; Patrón ReAct para la orquestación del agente; Patrón Reflection para la validación de código.1 |
| **Especificaciones de capas adicionales** | Integración obligatoria con librerías corporativas de Logging y Auditoría; Cifrado de datos en tránsito y reposo; Soporte para multi-región.1 |
| **Cuentas impactando** | Todas las cuentas con plataformas de datos a escala (Ficohsa, Mercantil, etc.).1 |
| **Disponibilidad/Servicio** | Generación de artefactos de ingestión en \< 10 minutos; Disponibilidad del agente 24/7 para procesos de Schema Drift automáticos. |
| **Eficiencia Operativa** | Reducción de \>70% en horas-hombre de ingeniería por pipeline; Reducción de incidentes por fallos de esquema en un 90% mediante Always-On Quality Gates.1 |

## **Integración con MLOps 2.0 y Always-On Data Quality Gates**

La automatización de la ingestión no debe ser una "caja negra". La referencia arquitectónica de MLOps 2.0 establece que los datos deben ser tratados como artefactos de primera clase, sujetos a los mismos rigores que el código fuente.3 El Agente Ingestión Cruda integra estos principios mediante la creación automática de "compuertas de calidad" (Quality Gates) en cada pipeline que genera.

### **Capas de Validación Automatizada**

El agente no solo genera el código para mover el dato; genera el código para *validar* el dato antes de que toque la capa Cruda. Estas validaciones se dividen en cuatro categorías críticas:

1. **Validación de Esquema:** Asegura que la estructura física cumple con el contrato. Detecta cambios inesperados antes de procesar la carga.3  
2. **Validación Semántica:** Utiliza el conocimiento del agente para verificar que los valores "hacen sentido". Por ejemplo, un campo edad no debe contener valores negativos ni superiores a 150\.3  
3. **Validación Temporal:** Comprueba la frescura de los datos (Data Freshness) y detecta duplicados temporales, asegurando que no se procesen datos obsoletos que ensucien la capa Cruda.3  
4. **Validación Distribucional:** El agente integra lógica para comparar las estadísticas del lote actual contra el histórico, identificando anomalías en el volumen o en la media de campos clave (Drift Detection).3

Esta integración asegura que el "Time-to-Insight" no se logre a expensas de la "Quality-of-Insight". Como indica el marco DQSOps (Data Quality Scoring Operations), cada conjunto de datos recibe una puntuación de calidad que el agente utiliza para decidir si el dato es promovido a la capa Cruda o desviado a un "Dead Letter Queue" para revisión humana.8

## **Patrones de Diseño GenAI Aplicados a la Ingeniería de Datos**

Para que el Agente Ingestión Cruda sea una solución de grado producción, se deben aplicar patrones de diseño específicos para aplicaciones generativas que aseguren la confiabilidad y la escalabilidad de la solución.2

### **Uso de Herramientas y Ejecución de Código (Pattern 21 & 22\)**

El agente no es un simple chat de texto. Mediante el **Tool Calling (Pattern 21\)**, el agente puede invocar funciones de Python para realizar cálculos complejos o interactuar con el AWS SDK para consultar el estado de un bucket S3. A través de la **Code Execution (Pattern 22\)**, el agente puede instanciar un entorno de ejecución efímero (como un contenedor Docker o un REPL de Python) para probar el script de Spark generado. Si el código falla al ejecutarse en este entorno controlado, el agente recibe el stack trace del error, razona sobre la causa raíz y corrige el código antes de entregarlo al repositorio final.2

### **Inyección de Dependencias y Model-Agnosticism**

Siguiendo el patrón de **Dependency Injection (Pattern 19\)**, la arquitectura del agente se diseña para ser agnóstica al modelo de lenguaje subyacente. Esto permite intercambiar el "cerebro" del agente (ej. pasar de Claude 3 a GPT-4o o a un modelo local tipo Llama 3\) sin reescribir la lógica de orquestación de datos.2 Esta flexibilidad es vital para organizaciones que deben cumplir con regulaciones estrictas de soberanía de datos y prefieren ejecutar agentes en infraestructura propia o "air-gapped".2

### **Evaluación mediante LLM-as-Judge (Pattern 17\)**

Para garantizar que los scripts generados cumplen con las mejores prácticas de la organización, se implementa un segundo agente evaluador bajo el patrón **LLM-as-Judge**. Este agente tiene un prompt de sistema cargado con las reglas de estilo y arquitectura de la organización (ej. "Todo script de Glue debe usar arquitectura hexagonal y registrar métricas en CloudWatch"). El evaluador califica el trabajo del Agente Ingestión Cruda y solo aprueba el artefacto si cumple con un umbral de calidad técnica predefinido.2

## **Implementación Técnica: De la Inferencia al Script Operativo**

La materialización de este JTBD requiere una orquestación precisa de tecnologías de procesamiento de lenguaje y herramientas de Big Data. El proceso técnico se desglosa en fases que el agente ejecuta de manera autónoma.

### **Perfilado y Análisis de Metadatos**

En la fase inicial, el agente utiliza mecanismos de **Semantic Indexing (Pattern 7\)** para comparar los metadatos de la fuente nueva con ingestiones previas almacenadas en una base de datos vectorial.2 Esto permite al agente "aprender" de patrones pasados: "Esta fuente JDBC de SQL Server se parece estructuralmente a la ingestión de la cuenta Ficohsa; aplicaré los mismos transformadores de zona horaria y manejo de nulos".1

### **Generación de Código bajo Arquitectura Hexagonal**

El código PySpark generado no es un script plano. El agente sigue una estructura modular que facilita el mantenimiento:

| Capa Arquitectónica | Responsabilidad Generada por el Agente |
| :---- | :---- |
| **Input Adapter** | Lectura resiliente desde la fuente (ej. spark.read.format("jdbc")...). Manejo de autenticación vía Secrets Manager. |
| **Domain Logic** | Transformaciones puras de tipado y limpieza de caracteres. Aplicación de reglas de negocio para la normalización de la capa Cruda. |
| **Output Adapter** | Escritura en S3 en formato Parquet/Delta. Registro automático en el Glue Data Catalog. Particionamiento dinámico. |
| **Infrastructure** | Configuración de los recursos de Glue (DPUs, timeouts) y habilitación de logs corporativos.1 |

Esta separación de preocupaciones asegura que, si el sistema origen cambia de base de datos a un sistema de mensajería (ej. Kafka), solo se deba regenerar el adaptador de entrada, manteniendo intacta la lógica de dominio.1

## **Medición del Impacto y Eficiencia Operativa**

La efectividad del Agente Ingestión Cruda se cuantifica a través de métricas claras definidas en el tracker, alineando el rendimiento técnico con el ahorro económico organizacional.1

### **KPI 1: Reducción del Ciclo de Desarrollo (Time to Market)**

El desarrollo manual de un pipeline de ingestión robusto, incluyendo pruebas y documentación, puede tomar entre 3 y 5 días de un ingeniero senior. El agente reduce este tiempo a menos de 10 minutos. Este ahorro de tiempo permite que el equipo de datos maneje un volumen de fuentes significativamente mayor sin incrementar la plantilla, mejorando el margen de rentabilidad de los proyectos de datos en más de un 60%.1

### **KPI 2: Calidad y Resiliencia (Always-On DQ)**

Mediante el uso de **Self-Check (Pattern 31\)**, el agente monitoriza la ejecución de los pipelines en producción. Si se detecta una anomalía en el esquema (Schema Drift), el agente no solo alerta, sino que puede proponer automáticamente el código de corrección basándose en los registros de error.2 Esto transforma el mantenimiento de reactivo a proactivo, reduciendo el "Down-time" de los datos analíticos.

### **KPI 3: Consistencia y Gobernanza**

Al centralizar la lógica de generación en un agente supervisado por el Chapter de Arquitectura, se garantiza que el 100% de las ingestiones sigan los mismos estándares. Esto elimina la fragmentación técnica donde cada desarrollador implementa soluciones "ad-hoc" que luego son imposibles de gobernar. El linaje de datos se genera de forma nativa, permitiendo que cualquier usuario de negocio en la organización pueda rastrear el origen de un dato en la capa Cruda hasta su fuente primordial.1

## **Roadmap de Implementación y Evolución**

La adopción de este JTBD se plantea en un proceso incremental que busca ganar la confianza de los equipos de ingeniería y los stakeholders de negocio.

### **Fase 1: Pilotaje en Fuentes Volátiles**

Se seleccionan sistemas origen con cambios frecuentes de esquema (ej. aplicaciones web con bases de datos NoSQL) para demostrar la capacidad de inferencia y adaptación del agente. En esta fase, el ingeniero de datos actúa como supervisor, revisando cada script generado antes del despliegue.2

### **Fase 2: Integración con la Plataforma de Operaciones (SOPP)**

El agente se integra como una herramienta dentro del "Standard Operating Process Platform". Los usuarios pueden "contratar" al agente para una tarea de ingestión simplemente proporcionando el acceso a la fuente. El agente se encarga de la creación del repositorio, la configuración del CI/CD y la publicación del esquema inicial.1

### **Fase 3: Autonomía y Self-Healing**

En la fase de madurez, el agente adquiere autonomía para gestionar el ciclo de vida completo del dato en la capa Cruda. Esto incluye el escalamiento dinámico de los recursos de cómputo en la nube basándose en el volumen de datos detectado y la auto-reparación de tuberías ante cambios menores en el origen.3

## **Conclusión: La IA como Motor de la Infraestructura de Datos**

La propuesta del "Agente Ingestión Cruda" representa una evolución necesaria en la arquitectura de datos moderna. Al definir este trabajo mediante el marco JTBD, la organización deja de ver la ingestión como un costo técnico inevitable y empieza a tratarla como una capacidad estratégica automatizable. La combinación del marco ReAct para el razonamiento técnico, los principios de MLOps 2.0 para la gobernanza y los patrones de diseño agéntico para la confiabilidad, crea una solución que no solo resuelve el problema del "Time-to-Insight", sino que posiciona a la organización en la vanguardia de la ingeniería de datos impulsada por IA.1

El éxito de esta iniciativa no se medirá solo por la sofisticación del modelo de IA utilizado, sino por la integración fluida de estos agentes en el día a día de los equipos de datos, permitiendo que el talento humano se desplace desde la carpintería de código boilerplate hacia la arquitectura de soluciones de alto impacto que transformen los datos en activos estratégicos reales para el negocio.

#### **Obras citadas**

1. Propuesta de Tablero de Control: JTBD Tracker (Temporal)  
2. lakshmanok/generative-ai-design-patterns  
3. MLOps 2\_0- A Reference Architecture for CI\_CD with Always-On Data Quality Gates.pdf  
4. REACT: SYNERGIZING REASONING AND ACTING IN LANGUAGE MODELS.pdf  
5. A Data Mining & Knowledge Discovery Process Model.pdf  
6. Guia\_paso\_a\_paso\_de\_Mineria\_de\_Datos.pdf  
7. Pre-Act: Multi-Step Planning and Reasoning Improves Acting in LLM Agents.pdf  
8. DQSOps- Data Quality Scoring Operations Framework for Data-Driven Applications.pdf  
9. Machine Learning Operations (MLOps)- Overview Definition and Architecture.pdf

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABEAAAAXCAYAAADtNKTnAAAA70lEQVR4Xu2Suw5BQRCGx61To1BQaRUkavEWepWCpxCN6EWhU6LVewKd+yUuiaChUOAfc87JZtwegC/5mv0ns7uzS/TnG0k4gGu4gSs4gSM4hX1YgkGr/iNNeIMpY80Pi3BB0jRsZC8ZwwN06wCkSTZo6MCEd+Citg4sPHALL9CrMocsSZOCDgx6JDURte5QIymI68BgR1IT0oENv86eXs+D4ZfhBifoUtkDex4tHRjk6Mtg6/T5Khl4hRUdmPDHeneVKFzCLvSpzCFGcoqOWg/APDzCMskTP8G/ckgyKG5yhnOSbz6zrMKEVf/n97gDdUU146lezDIAAAAASUVORK5CYII=>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA0AAAAYCAYAAAAh8HdUAAABAklEQVR4Xu3SP0tCcRTG8ZOKFSW4Ce5NEoENEeHSFAWNtQXh1NDg4htwNIiIegUu7W3SFrSEtjRJtAYNjrlUfg/ncu/vHm1zCXzgw4XzR+VcRf5tFlBDFUtRrYzleMKljgdc4x4fOMYAhWAuThNPKAa1NfzgOajFqeAb675BHnHhixr9ll+s+obYTz3wRc2N2FILGdcLD5LKodiS+sQdTrEYDvnomRv4kmRZdZEL5qZmBfu4xEhscS81EWXDF6KciS3pM5WS2Muclm2xpU3fOMIbsr5BrvAuk9eUW7FPO3H1HQzFzj2RHs7xij7a6OBF/jiAZit65sX+2fpudpP2PLPLGJWFLMbXxFTsAAAAAElFTkSuQmCC>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADoAAAAYCAYAAACr3+4VAAADUElEQVR4Xu2XWchNURTHl3mIzDKWoYSIB1NRMkSReUwUyVySF6TMQ0lkKoRShJTyYo4QDyJEZMqnvCljMmX4/++6291n2ee75zrXV+r+6vdw1tr3nLPPXnu4IiVK5KM6bGyDFURLG/hXVIGnYR+biKEOrGuDKTgIJ9ugzx1YBp/Ap1n7+w3AcfhStN0LOCuSVbaKPiwfS+En+BNuNLk0NIevYD+b8Jkj+uATsJLJkarwBtwDO5kcGQE/iD4sCWNFnzfcJlKyBD6H1WzC0Vv0wadsIksPeMEGPS7DZTZYDuvgd1jPJlLCNYKVOdcmHBwJdvSeTYDK8BrsaBNZuon+tpBFiPe7ZYNFYgW8a4MOlusX+M4mwAK41gY9lsO3NmjgQjUItoe14Ve4JdIiOW1Fp0oXCU+zKfAHbGITjmeiI+OXE0eaX76mF7Psgzdt0IMdfAh3wu3wouhzRvqNErIaXoEL4TG4OZrO0Ev0/mNswnFJtEFXL3YUDvWuQ/DF2S7EQPhZosv+ddH52cCLJaE1fC+53+2A93Pp3zQU7QcrMQi3BjYYlr1mB+M64MNVLlTaXKkfyZ+jfRXeNrEkzBMtycWih4PBsHukRY7XcKUNOviy7ChXLJYqSzbJdsGvzIdbxovez/8ItUTXAu65hdIUfhS9J+VUiIPTkCt7EB4CeANu4mwUO/QGHiBCD+V9eD/OUQdLmbHRXqwQesL1opXC+0yNpjNw4eNiFzrUZGCp8scsNZYXt5UkcOTP2CBYJVpqXGUda7IxzqNFolsT4XUz1ygAFzHujw5WGisjdORrJ9oPlnYQ7pNs8E1yL5CEQ6LHRktf0ftxESEcDc6dMtFtgYsY5zHhPOfJqkP22vIGHvCuecx7DOt7MccQ0eeyw0E4f/i1N9lEHqaLfhz30g6W0H7RCtkrWsoTREfipOjvHEdEOzrTi/nMEK0yrrTs8FnYxm/gMV/0PGDfJ8I0KX/PDNFCdE7wmBiiEWzlXfNfC0vVMhHOtkEPfjiOUr7348fdZoPFYjc8b4MFwj8MPPWkobNoxcRNgdRwxDiPRtlEQjh/D9vgX3AO7rLBYsOt4wGsYRMJmCRa4mnglsXjYexftGIyDg6wwQpigxT2D6rEf8sv1CmiHfCbk8IAAAAASUVORK5CYII=>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAoAAAAYCAYAAADDLGwtAAAA3UlEQVR4XuWQz+rBURDFZyGy8+eXYiErSwsbdiTFQvECtvIQdmKjpCjqV17C3oJkxyvIE9hYUOKMubem8QLKqU/fe885zdy+RF+tEIhZU6sELuAJ9ib7EE/i4sgGVlWSYsMGVgPwABEbsPKg6M47cFDZW2mwBSvQc987GOtSEpzBQnlDkvc1lUdLcANx5fVJ3hf1RsIZvFZrA47aqJGs4AleYZINE+VRmaRYV17FeS1QAF02g+AK2q70R7KSixkwAzmXUQecwBz8k0zkv7AGU1/y4ndl1T0AUur+g3oBR8kohfRh6YEAAAAASUVORK5CYII=>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAkAAAAYCAYAAAAoG9cuAAAAmUlEQVR4XmNgGAXkAkUg9gViHSBmRJMDg0YgPgTEeUC8Eoh7UKUZGGSB+BMQC0L5k4H4CkIaAjKB+B8QFwGxNBC7ALEBigogEAPir0D8H4qnoEojgCkQtwLxTQaIwhhkyUlAfBuJLwnEP4E4AkmM4T0Qz0Pi2wDxLSAWQBJjSATiwwwQH4EU7wRiBWQFMMAMxEpAzIEuMbwBAMj1F+ENSt3lAAAAAElFTkSuQmCC>

[image6]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAoAAAAYCAYAAADDLGwtAAAAjElEQVR4XmNgGHogAogfAPFjKH0PiG8DcRRCCSrYBMT/gdgDXQIZMALxKyD+DMSsaHIoQJsBYto2dAl0kM0AUViKLoEOVjNAFJqiSyADmPs+ATEzmhwKINl9ZegS6ADmPjN0CXTwkgESfizoEsjAiAFi2m50CRgIZYDE5wcg/sYAMfEhECcgqRkF+AEA8SAfH8Q2reIAAAAASUVORK5CYII=>

[image7]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAmwAAABFCAYAAAD3qbryAAAITklEQVR4Xu3deaitVRnH8aeupeaQZWU0mGU2R0nWjYq4WpBokaVBAxFWFo00adF49UrSPEgKRfZHGJlhs0VUbqFostKgefBmWJnNWlGW9fxYe3We/bT2ft89Hc/efj/wcN53rXX2PefeP+6P912DGQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAALaMY3PDBPOO3Wf4ddvw681rBwAAAMY7Nd1f7XX88PoBXleEvjx2kjh2P687hvvtoR0AAAAdTkv313sdGO7fE67z2Eni2BPCtewx/EpgAwAA6GFXun9Xur8gXOexk8Sxe3v9y+s/XheGdgIbAABADzmEPSFc7+X1l3Cfx06Sxx7g9WIroa0isAEAAPRwerh+QbjWgoBveh0c2uLYLnXsR7yOG14f4vXt4bUQ2AAAAHqowepMr597vXdYX/Latw4amiWwXex1ttdJXh/3Oux/IwhsAABgzT0jN8xolhDWRx17u+HXh9SOgMAGAADWjkLPM70e5PWZ1DerWUJYH33GEtgAAMBaW1Rgu2VumGBZYwEAANbSogIbAAAAloTABgAAsMXFDWgBAACwBX02N0zhOisb2D47dyQ38bqNlRWpb/b6pZXvuywOCnT26POsjFH90MqGua3Sz1/HvcVYgAAAANbQ53LDFF5pJSj91es+qW+SbV7fsdHTClpqEPte7gg+aWXMebkDAABglR3j9ROvK60cGXWVzb4a8/a2EaziAe993MzrTrkxmCawvT53zOm1XqfkxiBvBrxsXU8xAQDAgr3D6wqvy62cCvDj0e6Vc71thKtp3T03BIsKbPq7VkD9qY2GsJd67R7W+0O7zkH9kZVjtVr+brP9rvPQa+WTcyMAAFium3r9zevWuWMFvc02wtWdU988FhXY7mVlzC1yhxt4HWklEFXXeD0u3GdPtM0PbKI5g/pdAADAJtEB6ZNeua0aPSlT0FGQOTX1zWpRgU30xOzA1Pb2dC96qnZCbky+4vWt3LgJHun179wIAACW53yvB+fGFfd0K+FpUaFikYHt816Hh3utRNXB9NnDvR6YG60slHiU16Fe/7R22JvVY7welhsb7mA3zJM9AABulO5r3f/xalL7xY26yErQ+KKNBpCtQvPx9Lt9LHfMYJGB7Q1eLwr3Z4TrqPU5R3k9OdwrkN4q3M9DT/7keNs44H6S7+YGAACwHHod2hXY5lXDzqKqLx0o/w8r36O91OZR/+xFBLYTrewBJwdZez6bfCDd72EboarSliSRnpBFx1q/uWb3tvKz39/a45+bG9wFuQEAACyHXodOE4JWzUus/H56DTmPRQY2vc788PD63NiR6PMizWeL/1Z7W1nlG52V7s+x/k8/v2Hl87+a2rUp8KdTm8TVrAAAYIn0H3TXhrV6AqS9wCaV5lNtRdoW4265cQY1sH0/dwR9A9teXr+1slhAc8HG0YrXuGJ0p5VtS6rTrKzsVSjVv9EuK3P3qjd6PS3cy2HpXjTmd173tLJi+NLRbnurtec4aosSAACwCRQwXpMbl0RPhGY17rXhJNu9rs6NM6qB7Qe5I+gb2ETj9Dp6Em1QGzfz1SKE+oRNAeoPVgKd5hG+3Epo/s2wX5sP39br2uF9pcCXQ5zm+OnV6v5Wju36WejTn6Mne/kJm0JnDI8AAGBNtLbY2G0lZKgUFFRf83qhjW4W+wUbfdrURdtmPD83zmHRge2uuWGMV+QGGw1xce+8S7weHe61mEFnmkZaTPCh1FaNO+lBr0g1Fy7Sk7xx568CAIAV1gpsorD2p3CvbSr0ZCg+1ZkmsOnpz5dz45xqYNPh7+NME9j60hNCzSHrovNTdZB9XDHaClpPtbLoYRo6TSEfF6ZgzRFVAACsoVZgO9hKyPlEatf8rPr6T7R1SJ/ApjlYWkhRJ/VP4xG5IbihApsWAuSFBS138fq6jb7eVtDSa85o4LVPauuinyHT/DsFYwAAsGZage19VkJO3iB2TxudI6V5Wn0CmxYFTLsv2T2s+wzVGtjythrRMgKbaMuNrvlukcKtnqLlLT6msZ+VOXIPtf9fGHFSugcAAGukFdh0ELqCgZ6MRfez0SdsfQKbJscfkhs76MnUH230z2qpgU0/7zifsjJmZ2rfbE+y8ZvxTkMHvO/MjQAAYL21ApsCjralyDQxPoaoi2xyYNNcrxqqZql3WpterV5uo2M1P+5NYczLbOPs0lpXeR0XxgAAAKyEcYEtvw7V9h8KQL8PbQObHNi0F9zrrBz7pNo5pmq/Xluq9D0qHRjfop+5Va8KY57T6FcdE8YAAACshFZga70OrQsOHh/aBjY5sAEAAGABcmDT+ZR6nVjpDEvt7h8PN68GRmADAABYuhjYfuF1nZUJ/7rebWUj1leHMdHACGwAAABLl5+wTWNg/QObNnTNG722HGRl/lkf2oBWTwC7bPM60sp2GAAAACtnMwKbFixoe42u0wHebeXQ+ytzxxjneB2eGxs+aOUz2asMAACspM0IbNPYYf0D2zQGRmADAABo0mrTp9jGsVSnh9o1LIXGesj5DusObHq1eoqVs02r/Lla1arD0KuBEdgAAACadCTVR72elTvG2OH1q9yY6NgqHdPUOk9znIH1nxsHAABwo3Ou11Fe272OHlN1ftsOr18Pr7to0UGVP091ROgfWNmyBAAAAA3nWfuoq0yvRi/0+rPXY1Nftq/X/rlxjLOsvGa9xBY/5w4AAAANJ1o5xxQAAABb1Bk2+roTAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACg038BFbTAt+TcprcAAAAASUVORK5CYII=>