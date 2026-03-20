# **Pragma Knowledge: Estándar Global de Gestión de Conocimiento**

[**1\. Propósito y Visión Estratégica	3**](#1.-propósito-y-visión-estratégica)

[1.1. Las 5 Reglas de Oro (Innegociables)	3](#1.1.-las-5-reglas-de-oro-\(innegociables\))

[**2\. Arquitectura del árbol de directorios	4**](#2.-arquitectura-del-árbol-de-directorios)

[2.1. Datos propios del modelo	5](#2.1.-datos-propios-del-modelo)
[2.2. Reglas y Contextos de Negocio	6](#2.2.-reglas-y-contextos-de-negocio)
[2.2.1. Reglas Organizacionales (organizational-rules/)	6](#2.2.1.-reglas-organizacionales-\(organizational-rules/\))
[2.2.2. Dominios de Negocio (business-domain/)	6](#2.2.2.-dominios-de-negocio-\(business-domain/\))
[2.3. Conocimiento específico de Pragma (chapters)	7](#2.3.-conocimiento-específico-de-pragma-\(chapters\))
[2.3.1. Arquetipo de Repositorio del Chapter	7](#2.3.1.-arquetipo-de-repositorio-del-chapter)
[2.3.1.1. Estructura Raíz del Repositorio	7](#2.3.1.1.-estructura-raíz-del-repositorio)
[2.3.1.2. El archivo README.md	8](#2.3.1.2.-el-archivo-readme.md)
[2.3.1.3. El directorio kb/ (Knowledge Base)	8](#2.3.1.3.-el-directorio-kb/-\(knowledge-base\))
[2.3.1.4. Subdirectorios Tecnológicos (Organización Interna)	8](#2.3.1.4.-subdirectorios-tecnológicos-\(organización-interna\))
[Requisitos de Cumplimiento:	8](#requisitos-de-cumplimiento:)
[Ejemplo de Estructura Aceptada	9](#ejemplo-de-estructura-aceptada)
[2.3.1.5. Especificaciones o particularidades de clientes (directorio clients)	9](#2.3.1.5.-especificaciones-o-particularidades-de-clientes-\(directorio-clients\))
[2.3.2. Modelo de Trabajo y Gobernanza	10](#2.3.2.-modelo-de-trabajo-y-gobernanza)
[2.3.2.1. Flujo de Control de Versiones: Trunk-Based Development (TBD)	10](#2.3.2.1.-flujo-de-control-de-versiones:-trunk-based-development-\(tbd\))
[2.3.2.2. Proceso de Integración (Merge Requests)	10](#2.3.2.2.-proceso-de-integración-\(merge-requests\))
[2.3.3. Future Journey (Evolución del Repositorio)	11](#2.3.3.-future-journey-\(evolución-del-repositorio\))
[2.3.3.1. Validación Automatizada (Sustitución del Encargado de Formato)	11](#2.3.3.1.-validación-automatizada-\(sustitución-del-encargado-de-formato\))
[2.3.3.2. Sincronización Continua con S3 y la IA (Deploy Pipeline)	11](#2.3.3.2.-sincronización-continua-con-s3-y-la-ia-\(deploy-pipeline\))

[**3\. Plantillas de archivos para el árbol de directorios	12**](#3.-plantillas-de-archivos-para-el-árbol-de-directorios)
[3.1. Plantillas de los Chapters	12](#3.1.-plantillas-de-los-chapters)
[3.1.1. La Perspectiva (perspective.md)	12](#heading=)
[3.1.2. Decisiones (decisions/000-decision-name.md)	13](#3.1.2.-decisiones-\(decisions/000-decision-name.md\))
[3.1.3. Límites (limits/limit-name.md)	14](#heading=)
[3.1.4. Referencias (references/reference-name.md)	15](#heading=)
[3.1.5. Soluciones de Negocio (business-solutions/)	16](#3.1.5.-soluciones-de-negocio-\(business-solutions/\))
[3.1.5.1. Jobs To Be Done (standard/hitos/hito-title.md)	16](#3.1.5.1.-jobs-to-be-done-\(standard/jtbd/jtbd-title.md\))
[3.1.5.2. Generables / Entregables (outputs/output-title.md)	18](#3.1.5.2.-generables-/-entregables-\(standard/outputs/output-title.md\))

# **1\. Propósito y Visión Estratégica** {#1.-propósito-y-visión-estratégica}

El repositorio pragma-knowledge es la única fuente de verdad organizacional para el conocimiento técnico, metodológico y de negocio de Pragma. Su propósito dual es alinear a todos los colaboradores bajo un mismo marco de trabajo y servir como el **Grafo de Conocimiento Activo (Knowledge Base)** que alimenta a los Agentes de IA de Pragma.

A través de este repositorio, cualquier persona o agente debe ser capaz de responder con exactitud: **¿Cómo piensa Pragma?, ¿Qué decisiones ha tomado?, ¿Cuáles son sus límites? y ¿Qué capacidades tiene para resolver problemas de negocio?**.

## 1.1. Las 5 Reglas de Oro (Innegociables) {#1.1.-las-5-reglas-de-oro-(innegociables)}

1. **Nomenclatura Estricta (English & Kebab-case)**: Todo directorio, subdirectorio, archivo markdown e imagen debe ser nombrado en **inglés** y utilizar el formato **kebab-case** (minúsculas separadas por guiones). Ejemplo: business-solutions/hito/legacy-modernization.md.

2. **Contenido en Español Técnico:** Aunque la arquitectura de archivos esté en inglés, todo el contenido de los documentos debe redactarse en español, manteniendo los anglicismos técnicos propios de la industria de software.

3. **Principio de No Duplicidad (DRY):** El conocimiento se escribe una sola vez. Si un documento necesita información de otro, se debe enlazar utilizando referencias Markdown y dobles corchetes \[\[ruta-al-documento\]\].

4. **Contexto Explícito (AI-Ready):** Los documentos son consumidos por Inteligencia Artificial. No asuma conocimiento implícito. Defina los contextos, los acrónimos y escriba instrucciones directas y estructuradas.

5. **Atomicidad:** Cada archivo tiene un propósito único. Un documento de decisión documenta sólo la decisión; un documento de límite documenta sólo la restricción técnica.

# **2\. Arquitectura del árbol de directorios** {#2.-arquitectura-del-árbol-de-directorios}

La estructura del árbol de directorios refleja la separación de responsabilidades entre el negocio, la organización y la especialidad técnica.

```shell
pragma-knowledge/
├── manifest.json # Índice y metadatos dinámicos del repositorio
├── business-domain/ # Contexto y conocimiento propio de las industrias
│   ├── banking/ # Sector financiero, core bancario, medios de pago
│   ├── retail/ # Comercio, logística, fidelización, e-commerce
│   └── cross-cutting/ # Dominios transversales aplicables a cualquier sector
├── chapters/ # El núcleo del conocimiento dividido por especialidad
│   └── [chapter-name]/ # ej. architecture, backend, frontend, data, qa
│       ├── perspective.md # Identidad, mindset y principios del chapter
│       ├── decisions/ # ADRs (Architectural Decision Records)
│       ├── limits/ # Restricciones, fallos conocidos y cicatrices
│       ├── references/ # Guías procedimentales, estándares y checklists
│       └── business-solutions/ # El conocimiento aplicado a problemas reales
│           ├── standard/ # Flujos standard de negocio
│           │   ├── jtbd/ # JBTD: Capacidades que el chapter resuelve a los pragmáticos
│           │   └── outputs/ # Formatos y definiciones de los entregables
│           └── clients/ # Particularidades de clientes
│               └── [client-name]/ # ej. ficohsa, banco-mercantil, nequi
├── organizational-rules/ # Marco normativo y corporativo global
│   ├── constraints.md # Restricciones duras de la compañía (legales, financieras)
│   ├── decisions.md # Decisiones estratégicas tomadas a nivel gerencial o C-Level
│   └── principles.md # Pilares culturales y metodológicos globales
└── meta/ # Gobierno del propio estándar
    ├── schema.md # Reglas de contribución (este documento)
    └── changelog.md # Historial de cambios sobre la estructura de conocimiento
```

## 2.1. Datos propios del modelo {#2.1.-datos-propios-del-modelo}

Sobre la raíz del modelo, estarán el archivo **manifest.json** y el directorio **meta**, estos dos elementos constituyen información que el modelo mismo deberá manejar para el rastreo e indexación de la información. Estos serán de gobierno directo del **equipo de ingenieros de IA** en la definición y construcción.

## 2.2. Reglas y Contextos de Negocio {#2.2.-reglas-y-contextos-de-negocio}

### **2.2.1. Reglas Organizacionales (organizational-rules/)** {#2.2.1.-reglas-organizacionales-(organizational-rules/)}

* **constraints.md:** Documento vivo que lista barreras inamovibles. Ejemplo: "Por regulaciones de la Superintendencia, todo dato financiero debe residir físicamente en la región de Colombia".

* **decisions.md:** Decisiones directivas y de comité. Ejemplo: "La compañía ha decidido unificar todo el stack de repositorios bajo GitHub Enterprise".

* **principles.md:** Documentación profunda de la cultura de ingeniería y valores core de la organización.

### **2.2.2. Dominios de Negocio (business-domain/)** {#2.2.2.-dominios-de-negocio-(business-domain/)}

Carpeta destinada a almacenar la terminología, flujos y arquitecturas de referencia propias de industrias específicas. Por ejemplo, en banking/ se deben documentar conceptos como "Core Bancario", "Cámaras de Compensación" o "Normativas PCI DSS", para que tanto los equipos técnicos como la IA entiendan el contexto en el que se aplican los business-solutions.

## 2.3. Conocimiento específico de Pragma (chapters) {#2.3.-conocimiento-específico-de-pragma-(chapters)}

Cada chapter será responsable de la gestión y gobierno del conocimiento explícito reflejado en decisiones, límites y referencias, así como de gestionar las particularidades, entregables y flujos de valor de cara al negocio.

Para dicha gestión, cada chapter contará con un repositorio propio, en el que administrar los archivos relacionados a su área de conocimiento y que mediante procesos de despliegue llegarán al árbol principal de conocimiento y quedarán disponibles para el agente orquestador.

### **2.3.1. Arquetipo de Repositorio del Chapter** {#2.3.1.-arquetipo-de-repositorio-del-chapter}

Este repositorio es el **Arquetipo (Template Repository)** base para la gestión de conocimiento de cada Chapter en Pragma. Su objetivo es proporcionar una estructura estandarizada que cada equipo técnico debe clonar (hacer fork o usar como template) para gestionar de forma autónoma su documentación.

Este repositorio está diseñado para capturar la experiencia del chapter, estructurar dicha experiencia en un formato predecible y prepararla para ser el insumo principal de la **Base de Conocimiento Global** de la organización.

**Nomenclatura del Repositorio:** El nombre del repositorio de cada Chapter debe cumplir con el siguiente formato estricto: **sopp-chapter-\[nombre-chapter\]-doc-md-kb**. Por ejemplo, para el Chapter de Mobile, el nombre del repositorio debe ser: **sopp-chapter-mobile-doc-md-kb**.

#### **2.3.1.1. Estructura Raíz del Repositorio** {#2.3.1.1.-estructura-raíz-del-repositorio}

En la raíz del repositorio de cada chapter **deben** existir dos elementos principales para mantener la simplicidad cognitiva:

```shell
sopp-chapter-[nombre-chapter]-doc-md-kb/
├── README.md # Propósito inmutable y reglas de navegación del repo
└── kb/ # Knowledge Base: El conocimiento real del chapter
    ├── perspective.md # Identidad, mindset y principios del chapter
    ├── decisions/ # ADRs (Architectural Decision Records)
    ├── limits/ # Restricciones, fallos conocidos y cicatrices
    ├── references/ # Guías procedimentales, estándares y checklists
    └── business-solutions/ # El conocimiento aplicado a problemas reales
        ├── standard/ # Flujos standard de negocio
        │   ├── jtbd/ # JTBD: Capacidades que el chapter resuelve a los pragmáticos
        │   └── outputs/ # Formatos y definiciones de los entregables que se pueden generar
        └── clients/ # Particularidades de clientes
            └── [client-name]/ # ej. ficohsa, banco-mercantil, nequi
```

#### **2.3.1.2. El archivo README.md** {#2.3.1.2.-el-archivo-readme.md}

Es la puerta de entrada. Su contenido es **inmutable** **en estructura** y debe describir:

1. El nombre del chapter.
2. El propósito del repositorio (Estandarizar la práctica y centralizar el conocimiento).
3. Links directos a lineamientos específicos para enseñar a los nuevos miembros cómo contribuir.
4. El listado de "Maintainers" y del Encargado de Validación.

#### **2.3.1.3. El directorio kb/ (Knowledge Base)** {#2.3.1.3.-el-directorio-kb/-(knowledge-base)}

Es el único directorio donde los responsables del chapter deben crear y modificar archivos. Todo el contenido aquí debe seguir la nomenclatura estricta (ej. **limits/dynamodb-item-size.md**), redactado en español técnico y siguiendo obligatoriamente las plantillas de los lineamientos.

#### **2.3.1.4. Subdirectorios Tecnológicos (Organización Interna)** {#2.3.1.4.-subdirectorios-tecnológicos-(organización-interna)}

Para mejorar la navegabilidad y la organización del conocimiento de un Chapter que maneja **múltiples tecnologías** (ej. backend), se permite crear subdirectorios de agrupación únicamente dentro de las carpetas funcionales de más bajo nivel: **decisions/, limits/, references/, jtbd/, y outputs/**.

##### ***Requisitos de Cumplimiento:*** {#requisitos-de-cumplimiento:}

* Todos los nuevos subdirectorios deben nombrar la tecnología o el dominio que agrupan (ej. java, dot-net, mobile-ios).  
* Se debe adherir estrictamente a la **nomenclatura definida**.

##### 

##### ***Ejemplo de Estructura Aceptada*** {#ejemplo-de-estructura-aceptada}

La estructura aceptada se ve de la siguiente manera, tomando como ejemplo el *chapter* de backend:

```shell
sopp-chapter-backend-doc-md-kb/
├── README.md
└── kb/
    ├── perspective.md
    ├── decisions/
    │   ├── 001-api-first.md
    │   └── cloud-agnostic/ # Subcarpeta por Dominio/Contexto
    │       └── 002-multi-region.md
    ├── limits/
    │   ├── java/ # Subcarpeta por Tecnología (Ej. para límites de Java)
    │   │   └── jvm-memory-limit.md
    │   └── c-sharp/
    │       └── clr-thread-pool.md
    ├── references/
    │   └── dot-net/ # Subcarpeta por Tecnología (Ej. para referencias de .Net)
    │       └── entity-framework-guide.md
    └── business-solutions/
        ├── standard/
        │   ├── jtbd/
        │   │   └── legacy-modernization.md
        │   └── outputs/
        └── clients/
            └── [client-name]/
```

#### **2.3.1.5. Especificaciones o particularidades de clientes (directorio clients)** {#2.3.1.5.-especificaciones-o-particularidades-de-clientes-(directorio-clients)}

Dentro del directorio **business-solutions**, podrá existir un directorio clients en el que se alojen los archivos **.md** que contengan las especificaciones y personalizaciones o particularidades de los clientes, siempre y cuando cumplan lo siguiente:

* **No deben** existir subdirectorios dentro de la carpeta principal del cliente, en esta carpeta solo habrá documentos **.md** con especificaciones de las particularidades.
* Los nombres de los archivos **deben** respetar la nomenclatura definida para el directorio.
* Los nombres de los archivos **deben** reflejar su contenido (ej. nomenclatura-java.md)
* Los archivos **deben ser una extensión** de los que estén definidos a nivel global para el chapter.
* Los archivos **deben reflejar** particularidades y siempre deberá evaluarse si dichas particularidades son solo del cliente o deberían ser para todo Pragma.

### **2.3.2. Modelo de Trabajo y Gobernanza** {#2.3.2.-modelo-de-trabajo-y-gobernanza}
El ciclo de vida del conocimiento se rige bajo un modelo ágil y de revisiones estrictas.

#### **2.3.2.1. Flujo de Control de Versiones: Trunk-Based Development (TBD)** {#2.3.2.1.-flujo-de-control-de-versiones:-trunk-based-development-(tbd)}

* **Tronco Único (trunk):** La rama principal es sagrada y representa la "Verdad Actual" del chapter.

* **Ramas de Feature (feature/\*):** Para añadir un nuevo estándar, decisión o Hito, el contribuidor debe crear una rama efímera desde trunk (ej. feature/add-kafka-decision).

* **Commits Atómicos:** Los commits deben representar adiciones lógicas y completas.

#### **2.3.2.2. Proceso de Integración (Merge Requests)** {#2.3.2.2.-proceso-de-integración-(merge-requests)}

Toda integración a trunk requiere obligatoriamente:

1. **Apertura de Merge Request (MR / PR):** Nunca se hace push directo a trunk.

2. **Revisión de Pares (Peer Review):** Al menos un (1) miembro core del chapter debe revisar el MR, validando la veracidad técnica de la información (que lo que se documenta es correcto y aplicable).

3. **Validación de Formato y Contenido:** La revisión final para integrar el conocimiento a trunk está a cargo del **Encargado de Validación**. Esta persona verificará manualmente:

   * El uso estricto de kebab-case y nombres en inglés.  
   * La correcta utilización de los templates oficiales (sin omitir secciones).  
   * La claridad, completitud y correcta interconexión de los archivos (\[\[enlaces\]\]).

### **2.3.3. Future Journey (Evolución del Repositorio)** {#2.3.3.-future-journey-(evolución-del-repositorio)}

A medida que el modelo de adopción madure y los equipos dominen la estructura, el gobierno del repositorio evolucionará hacia la automatización total para escalar la operación. Los siguientes puntos están definidos en el roadmap futuro:

#### **2.3.3.1. Validación Automatizada (Sustitución del Encargado de Formato)** {#2.3.3.1.-validación-automatizada-(sustitución-del-encargado-de-formato)}

Implementación de un pipeline de CI (Continuous Integration) que ejecute un Linter sobre los Merge Requests. Este bot reemplazará la revisión manual de formato verificando automáticamente:

* Reglas de expresiones regulares para nombres de archivos (^\[a-z0-9-\]+.md$).
* Presencia de campos obligatorios en los Markdown.
* Validación de enlaces rotos en las referencias cruzadas.

#### **2.3.3.2. Sincronización Continua con S3 y la IA (Deploy Pipeline)** {#2.3.3.2.-sincronización-continua-con-s3-y-la-ia-(deploy-pipeline)}

Creación de un pipeline de CD (Continuous Deployment) disparado por cada merge a la rama trunk. Este proceso sincroniza automáticamente el contenido de la carpeta kb/ hacia un **Bucket S3** centralizado, manteniendo a los Agentes de IA corporativos actualizados en tiempo real sin intervención humana.

# **3\. Plantillas de archivos para el árbol de directorios** {#3.-plantillas-de-archivos-para-el-árbol-de-directorios}

## 3.1. Plantillas de los Chapters {#3.1.-plantillas-de-los-chapters}

Cada archivo dentro de un Chapter debe copiar de manera exacta estas plantillas. Ningún campo debe ser omitido; si un campo no aplica, debe indicarse explícitamente "No aplica para este caso" justificando el motivo.

### **3.1.1. La Perspectiva (perspective.md)**

Este archivo define el alma del chapter. Es el primer documento que un Agente de IA lee para entender cómo debe "actuar" o "pensar" cuando representa a esta especialidad.

```shell
# Perspectiva del Chapter: [NOMBRE_DEL_CHAPTER]

## Propósito
Describa la razón de ser de este chapter dentro de Pragma y su aporte central de valor.

## Mentalidad (Mindset)
¿Cuál es la filosofía principal con la que el chapter aborda los problemas? (ej. "Diseño API-First", "Resiliencia sobre eficiencia pura", "Automatización absoluta").

## Valores de Ingeniería
Liste las prioridades técnicas innegociables. (ej. 1. Seguridad, 2. Escalabilidad, 3. Mantenibilidad).

## Lo que defendemos (Dos)
Prácticas, metodologías o arquitecturas que el chapter promueve activamente en todos los proyectos.

## Lo que combatimos (Don'ts / Anti-patrones)
Prácticas, tecnologías, estilos de código o arquitecturas que están estrictamente desaconsejadas o prohibidas por el chapter.
```

### **3.1.2. Decisiones (decisions/000-decision-name.md)** {#3.1.2.-decisiones-(decisions/000-decision-name.md)}

Registro histórico de compromisos técnicos. Obligatorio usar el estándar ADR.

```shell
# [ID_NUMERICO_3_DIGITOS] - [TITULO_DE_LA_DECISION]

## Estado
[Propuesto | Aprobado | Depreciado | Superado por [[ID-nueva-decision]]]

## Contexto
Detalle de manera amplia el problema, la necesidad, el dolor tecnológico o la oportunidad de negocio que motivó la necesidad de tomar esta decisión.

## Opciones Consideradas
Liste de manera objetiva las alternativas que se evaluaron antes de tomar la decisión.
- **Opción 1:** [Nombre]. Pros: [lista]. Contras: [lista].
- **Opción 2:** [Nombre]. Pros: [lista]. Contras: [lista].

## Decisión Tomada
Describa explícitamente cuál fue la opción ganadora y el razonamiento técnico detallado del por qué se eligió por encima de las demás.

## Implicaciones
Detalle las consecuencias de haber tomado esta decisión.
- **Positivas:** Qué beneficios ganamos a corto y largo plazo.
- **Negativas / Riesgos:** Qué deuda técnica, esfuerzo de migración, curva de aprendizaje o vulnerabilidad asumimos.

## Referencias Relacionadas
- [[enlace-a-referencia-o-principio-relacionado]]
```

### **3.1.3. Límites (limits/limit-name.md)**

Capitalización de errores, "cicatrices" organizacionales y fronteras reales de las herramientas.

```shell
# Límite: [TITULO_DEL_LIMITE_O_RESTRICCION]

## Contexto de Identificación
Detalle en qué proyecto, versión específica de tecnología, entorno o volumen de carga se identificó que esta herramienta, arquitectura o proceso dejó de funcionar o alcanzó su límite.

## La Restricción
Descripción técnica exacta de la barrera. (Ej. "DynamoDB tiene un límite estricto de 400KB por item", "Este patrón de micro-frontends causa fugas de memoria en navegadores móviles").

## Evidencia e Impacto
Análisis de la causa raíz. ¿Qué le ocurre al sistema o al negocio si se intenta forzar o ignorar este límite?

## Señales de Alerta
Métricas, logs o síntomas tempranos que indican que un sistema se está acercando peligrosamente a este límite.

## Workarounds y Mitigaciones
Instrucciones detalladas de cómo sortear este límite de manera segura, o qué alternativas arquitectónicas usar cuando se sabe que se alcanzará este punto.
```

### **3.1.4. Referencias (references/reference-name.md)**

El "Cómo se hace". Procedimientos, guías, estándares de codificación y manuales operativos.

```shell
# Referencia: [TITULO_DEL_PROCEDIMIENTO_O_ESTANDAR]

## Propósito
¿Qué se logrará al terminar de leer o ejecutar esta referencia?

## Ámbito de Aplicación
¿En qué tipo de proyectos, lenguajes o infraestructuras es obligatorio aplicar esta guía?

## Paso a Paso / Lineamientos
Instrucciones secuenciales detalladas. Si es un estándar de código, incluya los fragmentos necesarios.
1. [Paso 1 detallado]
2. [Paso 2 detallado]

## Checklist de Verificación
Lista accionable para comprobar que el proceso se realizó con éxito.
- [ ] Validación 1
- [ ] Validación 2

## Herramientas y Recursos
Librerías, extensiones, comandos de consola o URLs necesarias para completar esta tarea.
```

### **3.1.5. Soluciones de Negocio (business-solutions/)** {#3.1.5.-soluciones-de-negocio-(business-solutions/)}

Esta carpeta es el corazón del repositorio para los Agentes de IA y el equipo comercial/arquitectura. Contiene la materialización del conocimiento en capacidades vendibles y ejecutables.

#### **3.1.5.1. Jobs To Be Done (standard/jtbd/jtbd-title.md)** {#3.1.5.1.-jobs-to-be-done-(standard/jtbd/jtbd-title.md)}

Capacidad completa que un chapter puede ejecutar para solucionar un problema real.

```shell
# JTBD: [TITULO_DEL_JTBD]

## Descripción y Dolor
¿Qué necesidad, dolor u oportunidad puntual atiende esta solución para el cliente?

## Contexto
Indique en qué escenario aplica esta solución y para qué tipo de cliente, industria, área o etapa de madurez tecnológica está pensada.

## Alcance
- **Incluye:** Describa exactamente qué cubre esta solución.
- **No incluye:** Describa explícitamente qué queda fuera del alcance y es responsabilidad de otro Job u otro proveedor.

## Cuándo aplica
Explique en qué escenarios esta solución es la ideal y generará el mayor valor o ROI.

## Cuándo no aplica
Explique en qué casos NO debería proponerse o implementarse porque es contraproducente, costosa, o existe una mejor alternativa (ej. "No usar para clientes que no tienen equipo in-house").

## Entradas (Inputs)
Liste de forma exhaustiva la información, accesos, artefactos, condiciones previas o dependencias necesarias por parte del cliente o de otros equipos antes de poder iniciar este Job.
- Entrada 1: [Descripción]
- Entrada 2: [Descripción]

## Flujo de alto nivel
Describa las fases o pasos principales de la solución, sin bajar a microtareas, pero dando claridad del proceso metodológico.
1. Paso o fase 1: [Descripción]
2. Paso o fase 2: [Descripción]
3. Paso o fase 3: [Descripción]

## Participación del chapter
Explique de manera concreta cuál es el rol específico de los integrantes de ESTE chapter dentro de esta solución (Ej. Lideran el diseño, ejecutan la migración, proveen consultoría técnica).

## Dependencias
Indique otros chapters, sistemas externos, herramientas licenciadas, decisiones arquitecturales o manuales necesarios para que esto funcione.
- Dependencia 1: [Descripción o link]
- Dependencia 2: [Descripción o link]

## Salidas esperadas (Outputs)
Describa qué resultados palpables, documentos, repositorios o capacidades produce esta solución. Deben enlazar a la carpeta `outputs/`.
- Salida 1: [[link-a-output]]
- Salida 2: [[link-a-output]]

## Riesgos y consideraciones
Liste limitaciones del mercado, puntos críticos técnicos, o aspectos organizacionales que deben vigilarse estrechamente durante la ejecución para evitar el fracaso del Job.
- Riesgo 1: [Descripción]
- Riesgo 2: [Descripción]

## Variantes comunes
Indique adaptaciones frecuentes de esta solución (Ej. "Versión acelerada para startups", "Versión de alto cumplimiento para entidades financieras").

## Relación con otros chapters
Explique con qué otros chapters (ej. QA, Cloud, Data) suele integrarse esta solución, por qué es necesaria la sinergia y en qué momento del flujo interactúan.

## Referencias relacionadas
- Decisión relacionada: [[link-a-decision]]
- Referencia relacionada: [[link-a-referencia]]
- Límite relacionado: [[link-a-limite]]
```

#### **3.1.5.2. Generables / Entregables (standard/outputs/output-title.md)** {#3.1.5.2.-generables-/-entregables-(standard/outputs/output-title.md)}

La definición exacta de lo que se le entrega en las manos al cliente o a otro equipo.

```shell
# Output: [NOMBRE_DEL_ENTREGABLE]

## Descripción General
¿Qué es exactamente este artefacto? (ej. Un documento de arquitectura de software, un repositorio configurado con CI/CD, un informe de vulnerabilidades).

## Formato y Herramienta de Entrega
Especifique el formato final (PDF, Repositorio Git, Tablero de Miro, Documento Markdown) y la herramienta institucional utilizada para su creación.

## Estructura Mandatoria del Artefacto
Desglose sección por sección lo que DEBE contener este entregable para ser considerado un producto oficial de Pragma.
- [Sección 1]: Descripción de lo que debe ir aquí.
- [Sección 2]: Descripción de lo que debe ir aquí.
- [Sección N]: Descripción de lo que debe ir aquí.

## Criterios de Calidad (Definition of Done)
Lista estricta de validaciones que deben pasar antes de que este output sea entregado.
- [ ] Criterio de validación 1
- [ ] Criterio de validación 2
```

