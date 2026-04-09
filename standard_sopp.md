# **Pragma Knowledge: Estándar Global de Gestión de Conocimiento**

# **1\. Propósito y Visión Estratégica**

El repositorio pragma-knowledge es la única fuente de verdad organizacional para el conocimiento técnico, metodológico y de negocio de Pragma. Su propósito dual es alinear a todos los colaboradores bajo un mismo marco de trabajo y servir como el **Grafo de Conocimiento Activo (Knowledge Base)** que alimenta a los Agentes de IA de Pragma.

A través de este repositorio, cualquier persona o agente debe ser capaz de responder con exactitud: **¿Cómo piensa Pragma?, ¿Qué decisiones ha tomado?, ¿Cuáles son sus límites? y ¿Qué capacidades tiene para resolver problemas de negocio?**.

## 1.1. Las 5 Reglas de Oro (Innegociables)

1. **Nomenclatura Estricta (English & Kebab-case & Single Word):** Todo directorio, subdirectorio y archivo markdown debe ser nombrado en inglés y utilizar el formato kebab-case (minúsculas separadas por guiones). Los subdirectorios de agrupación creados dentro de las carpetas funcionales (ej. java/, cloud/) deben consistir en una sola palabra para optimizar la generación de metadatos.

2. **Contenido en Español Técnico:** Aunque la arquitectura de archivos esté en inglés, todo el contenido de los documentos debe redactarse en español, manteniendo los anglicismos técnicos propios de la industria de software.

3. **Principio de No Duplicidad (DRY):** El conocimiento se escribe una sola vez. Si un documento necesita información de otro, se debe hacer mención contextual del documento relacionado (nombre y categoría) para que el lector o agente pueda ubicarlo.

4. **Contexto Explícito (AI-Ready):** Los documentos son consumidos por Inteligencia Artificial. No asuma conocimiento implícito. Defina los contextos, los acrónimos y escriba instrucciones directas y estructuradas.

5. **Atomicidad:** Cada archivo tiene un propósito único. Un documento de decisión documenta sólo la decisión; un documento de límite documenta sólo la restricción técnica.

# **2\. Arquitectura del árbol de directorios**

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
├── organizational-rules/ # Marco normativo y corporativo global
│   ├── constraints.md # Restricciones duras de la compañía (legales, financieras)
│   ├── decisions.md # Decisiones estratégicas tomadas a nivel gerencial o C-Level
│   └── principles.md # Pilares culturales y metodológicos globales
└── meta/ # Gobierno del propio estándar
    ├── schema.md # Reglas de contribución (este documento)
    └── changelog.md # Historial de cambios sobre la estructura de conocimiento
```

## 2.1. Datos propios del modelo

Sobre la raíz del modelo, estarán el archivo **manifest.json** y el directorio **meta**, estos dos elementos constituyen información que el modelo mismo deberá manejar para el rastreo e indexación de la información. Estos serán de gobierno directo del **equipo de ingenieros de IA** en la definición y construcción.

## 2.2. Reglas y Contextos de Negocio

### **2.2.1. Reglas Organizacionales (organizational-rules/)**

* **constraints.md:** Documento vivo que lista barreras inamovibles. Ejemplo: "Por regulaciones de la Superintendencia, todo dato financiero debe residir físicamente en la región de Colombia".

* **decisions.md:** Decisiones directivas y de comité. Ejemplo: "La compañía ha decidido unificar todo el stack de repositorios bajo GitHub Enterprise".

* **principles.md:** Documentación profunda de la cultura de ingeniería y valores core de la organización.

### **2.2.2. Dominios de Negocio (business-domain/)**

Carpeta destinada a almacenar la terminología, flujos y arquitecturas de referencia propias de industrias específicas. Por ejemplo, en banking/ se deben documentar conceptos como "Core Bancario", "Cámaras de Compensación" o "Normativas PCI DSS", para que tanto los equipos técnicos como la IA entiendan el contexto en el que se aplican los business-solutions.

## 2.3. Conocimiento específico de Pragma (chapters)

Cada chapter será responsable de la gestión y gobierno del conocimiento explícito reflejado en decisiones, límites y referencias, así como de gestionar las particularidades, entregables y flujos de valor de cara al negocio.

Para dicha gestión, cada chapter contará con un repositorio propio, en el que administrar los archivos relacionados a su área de conocimiento y que mediante procesos de despliegue llegarán al árbol principal de conocimiento y quedarán disponibles para el agente orquestador.

### **2.3.1. Arquetipo de Repositorio del Chapter**

Este repositorio es el **Arquetipo (Template Repository)** base para la gestión de conocimiento de cada Chapter en Pragma. Su objetivo es proporcionar una estructura estandarizada que cada equipo técnico debe clonar (hacer fork o usar como template) para gestionar de forma autónoma su documentación.

Este repositorio está diseñado para capturar la experiencia del chapter, estructurar dicha experiencia en un formato predecible y prepararla para ser el insumo principal de la **Base de Conocimiento Global** de la organización.

**Nomenclatura del Repositorio:** El nombre del repositorio de cada Chapter debe cumplir con el siguiente formato estricto: **sopp-chapter-\[nombre-chapter\]-doc-md-kb**. Por ejemplo, para el Chapter de Mobile, el nombre del repositorio debe ser: **sopp-chapter-mobile-doc-md-kb**.

#### **2.3.1.1. Estructura Raíz del Repositorio**

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
        │   └── outputs/ # Formatos/definiciones de entregables que se pueden generar
        └── clients/ # Particularidades de clientes
            └── [client-name]/ # ej. ficohsa, banco-mercantil, nequi
```

Adicionalmente, el arquetipo provee directorios de soporte operativo que **no contienen conocimiento del chapter** sino herramientas para su gestión (ver sección 2.3.1.6):

```shell
sopp-chapter-[nombre-chapter]-doc-md-kb/
├── README.md
├── kb/
├── docs/ # Documentación del estándar y guías de uso de las tools
└── tools/ # Scripts, templates y prompts para gestionar el repositorio
    ├── scaffold.sh # Inicialización y validación de la estructura
    ├── setup-workflow.sh # Instalación del workflow de deploy
    ├── new-doc.sh # Creación de documentos desde templates
    ├── templates/ # Plantillas para scripts y documentos
    └── prompts/ # Prompts para tareas asistidas por agentes de IA 
```

#### **2.3.1.2. El archivo README.md**

Es la puerta de entrada. Su contenido es **inmutable** **en estructura** y debe describir:

1. El nombre del chapter.

2. El propósito del repositorio (Estandarizar la práctica y centralizar el conocimiento).

3. Links directos a lineamientos específicos para enseñar a los nuevos miembros cómo contribuir.

4. El listado de "Maintainers" y del Encargado de Validación.

#### **2.3.1.3. El directorio kb/ (Knowledge Base)**

Es el único directorio donde los responsables del chapter deben crear y modificar archivos. Todo el contenido aquí debe seguir la nomenclatura estricta (ej. **limits/dynamodb-item-size.md**), redactado en español técnico y siguiendo obligatoriamente las plantillas de los lineamientos.

#### **2.3.1.4. Subdirectorios Tecnológicos (Organización Interna)**

Para mejorar la navegabilidad y la organización del conocimiento de un Chapter que maneja **múltiples tecnologías** (ej. backend), se permite crear subdirectorios de agrupación únicamente dentro de las carpetas funcionales de más bajo nivel: **decisions/, limits/, references/, jtbd/, y outputs/**.

##### ***Requisitos de Cumplimiento:***

* Todos los nuevos subdirectorios deben nombrar la tecnología o el dominio que agrupan (ej. java, dot-net, mobile-ios).  
* Se debe adherir estrictamente a la **nomenclatura definida**.

##### 

##### ***Ejemplo de Estructura Aceptada***

La estructura aceptada se ve de la siguiente manera, tomando como ejemplo el *chapter* de backend:

```shell
sopp-chapter-backend-doc-md-kb/
├── README.md
└── kb/
    ├── perspective.md
    ├── decisions/
    │   ├── 001-api-first.md
    │   └── cloud/ # Subcarpeta por Dominio/Contexto
    │       └── 002-multi-region.md
    ├── limits/
    │   ├── java/ # Subcarpeta por Tecnología (Ej. para límites de Java)
    │   │   └── jvm-memory-limit.md
    │   └── csharp/
    │       └── clr-thread-pool.md
    ├── references/
    │   └── dotnet/ # Subcarpeta por Tecnología (Ej. para referencias de .Net)
    │       └── entity-framework-guide.md
    └── business-solutions/
        ├── standard/
        │   ├── jtbd/
        │   │   └── legacy-modernization.md
        │   └── outputs/
        └── clients/
            └── [client-name]/
```

##### ***2.3.1.4.1. Regla de Versionado por Ruta (vX)***

Para la gestión de diferentes versiones pertenecientes a un archivo, se debe definir como último nivel una carpeta que indique el número de versión en el formato **vX** (siendo X un ordinal consecutivo: ej. **v1**, **v2**, etc.). Este directorio de versión constituye la única excepción y **puede ser ubicado, como máximo, en el sexto nivel de profundidad** en el árbol de directorios. Por defecto, la definición inicial de todo archivo debe incluir la carpeta **v1/**.

###### *Ejemplo de Estructura de Versionado Aceptada:*

kb/limits/java/jvm/v1/memory-limit.md (5 niveles de directorio, incluyendo el jvm/ y v1/).

##### ***2.3.1.4.2. Límite de Profundidad y Metadatos***

La estructura de subdirectorios aporta un nivel de definición de metadatos que sirven al Agente de IA para identificar la información relevante. Por definición, el **máximo número de niveles de profundidad permitido para las carpetas de agrupación es de 5 directorios**, siendo la carpeta **kb/** el primer nivel (L1). Se recomienda que las carpetas generadas sean altamente relevantes, pudiendo aplicarse a nivel de tecnología o de scope. Nota: La única excepción a este límite es el directorio de versionado **vX/** definido en la sección 2.3.1.4.1.

#### **2.3.1.5. Especificaciones o particularidades de clientes (directorio clients)**

Dentro del directorio **business-solutions**, podrá existir un directorio clients en el que se alojen los archivos **.md** que contengan las especificaciones y personalizaciones o particularidades de los clientes, siempre y cuando cumplan lo siguiente:

* **Se permite** el uso de subdirectorios funcionales **decisions/**, **limits/** y **references/** dentro de cada directorio de cliente (\[client-name\]/). Estos directorios sirven para categorizar las especializaciones de los documentos globales del Chapter, respetando la estructura N5/N6. Ejemplo: **kb/business-solutions/clients/ficohsa/limits/v1/storage-limit.md**

* Los nombres de los archivos **deben** respetar la nomenclatura definida para el directorio.

* Los nombres de los archivos **deben** reflejar su contenido (ej. nomenclatura-java.md)

* Los archivos **deben ser una extensión** de los que estén definidos a nivel global para el chapter.

* Los archivos **deben reflejar** particularidades y siempre deberá evaluarse si dichas particularidades son solo del cliente o deberían ser para todo Pragma.

#### **2.3.1.6. Directorios Auxiliares del Arquetipo (tools/ y docs/)**

Además de los elementos principales (**README.md** y **kb/**), el arquetipo incluye directorios de **soporte operativo** que facilitan la inicialización, mantenimiento y validación del repositorio. Estos directorios **no contienen conocimiento del chapter** y no son desplegados hacia la Base de Conocimiento Global ni consumidos por los Agentes de IA.

* **docs/:** Contiene la documentación del estándar de gestión de conocimiento y la guía de uso de las herramientas del arquetipo. Sirve como referencia local para los colaboradores del chapter.  
* **tools/:** Contiene scripts de automatización (**scaffold.sh**, **setup-workflow.sh**, **new-doc.sh**), templates oficiales para la generación de documentos y prompts predefinidos para tareas asistidas por agentes de IA. La guía completa de uso se encuentra en **docs/tools-guide.md**.

**Sobre los prompts (tools/prompts/):** Los prompts son instrucciones estructuradas diseñadas para ser ejecutadas por agentes de IA dentro del IDE, permitiendo automatizar tareas complejas como auditorías de cumplimiento sobre la **kb/**. Estos prompts han sido diseñados y probados únicamente con **Kiro**, por lo que su comportamiento puede variar en otros entornos o agentes de IA.

**Importante:** Los colaboradores del chapter **no deben modificar** el contenido de **tools/** ni **docs/**. Estos directorios son gestionados desde el arquetipo y se actualizan al sincronizar cambios desde el template repository.

### **2.3.2. Modelo de Trabajo y Gobernanza**

El ciclo de vida del conocimiento se rige bajo un modelo ágil y de revisiones estrictas.

#### **2.3.2.1. Flujo de Control de Versiones: Trunk-Based Development (TBD)**

* **Tronco Único (trunk):** La rama principal es sagrada y representa la "Verdad Actual" del chapter.

* **Ramas de Feature (feature/\*):** Para añadir un nuevo estándar, decisión o Hito, el contribuidor debe crear una rama efímera desde trunk (ej. feature/add-kafka-decision).

* **Commits Atómicos:** Los commits deben representar adiciones lógicas y completas.

#### **2.3.2.2. Proceso de Integración (Merge Requests)**

Toda integración a **trunk** requiere obligatoriamente la ejecución del siguiente Flujo de Contribución por parte del colaborador y su posterior validación:

##### ***Flujo de Contribución del Colaborador***

1. Cree una rama efímera para su contribución siguiendo el formato: **feature/\[nombre-descriptivo-del-aporte\]**.

2. **Copie la plantilla Markdown correspondiente desde la Sección 3 de este Estándar Global a la ubicación dentro de la carpeta kb/ de su Chapter.**

3. Llene **TODOS** los campos obligatorios del template (si un campo no aplica, debe indicarlo explícitamente justificando el motivo).

4. Abra un Merge Request (MR/PR) en su repositorio.

##### ***Proceso de Validación Obligatorio***

Una vez abierto el Merge Request, la integración a **trunk** solo procede si se cumplen los siguientes requisitos:

* **Revisión de Pares (Peer Review):** Al menos un (1) miembro core del chapter debe revisar el MR, validando la veracidad técnica de la información (que lo que se documenta es correcto y aplicable).

* **Validación de Formato y Contenido:** La revisión final está a cargo del **Encargado de Validación**. Esta persona verificará manualmente:

  * El uso estricto de kebab-case y nombres en inglés.

  * La correcta utilización de los templates oficiales (sin omitir secciones).

  * La claridad, completitud y correcta interconexión de los archivos.

#### **2.3.2.3. Sincronización Continua con S3 y la IA (Deploy Pipeline)**

El pipeline de CD (Continuous Deployment) sincroniza automáticamente el contenido de la carpeta **kb/** hacia los **Buckets S3** configurados, manteniendo a los Agentes de IA corporativos actualizados sin intervención humana. El deploy opera en dos ambientes:

* **dev:** Se dispara manualmente mediante un comentario \`/deploy-dev\` en un Pull Request. Requiere que el comentarista tenga permisos de escritura en el repositorio.  
* **prd:** Se dispara automáticamente al hacer merge a la rama **trunk**.

El workflow soporta despliegue a múltiples buckets y múltiples cuentas AWS en paralelo, con autenticación vía OIDC y role chaining. Después de cada deploy, se comenta un resumen consolidado en el PR con el estado de todos los destinos.

##### ***Configuración Centralizada de Destinos de Deploy***

Los destinos de deploy (cuentas AWS, roles y buckets) se gestionan de forma centralizada en el repositorio **sopp-chapters-kb-deploy-config**, que contiene un archivo JSON por environment (**config/dev.json**, **config/prd.json**). Cuando se agrega o quita un bucket o una cuenta, se edita el JSON en este repositorio y todos los chapters lo consumen automáticamente en el siguiente deploy.

El acceso al repositorio de configuración se realiza mediante una **GitHub App** con permiso de lectura, cuyos secretos se configuran a nivel de cada repositorio de chapter.

#### **2.3.2.4. Onboarding de un Nuevo Repositorio de Chapter**

Al crear un nuevo repositorio de chapter a partir del arquetipo, se deben completar los siguientes pasos:

1. Crear el repositorio usando el formato de nomenclatura: **sopp-chapter-\\\[nombre-chapter\\\]-doc-md-kb**.  
2. Ejecutar **\`./tools/scaffold.sh\`** para inicializar la estructura.  
3. Ejecutar **\`./tools/setup-workflow.sh\`** para instalar los workflows de deploy.  
4. Configurar los **secretos del repositorio** (Settings → Secrets and variables → Actions → Secrets):  
   * **DEPLOY\_CONFIG\_APP\_ID**: App ID de la GitHub App de acceso al repo de config.  
   * **DEPLOY\_CONFIG\_APP\_PRIVATE\_KEY**: Private key de la GitHub App.  
5. Configurar la **variable del repositorio** (Settings → Secrets and variables → Actions → Variables):  
   * **CHAPTER\_FOLDER**: Subdirectorio propio del chapter (ej. \`backend\`, \`mobile\`, \`data\`). Este valor determina la ruta final en S3.  
   * **USE\_DEBUG\_TARGETS**: Valor \`1\`. Temporal, para validar que el workflow funciona correctamente usando los targets de debug en lugar de los reales. Eliminar una vez validado.  
6. Crear los **environments** **\`dev\`** y **\`prd\`** en GitHub (Settings → Environments).  
7. (Opcional) Configurar notificaciones Slack en cada environment:  
   * Variable: \`SLACK\_CHANNEL\_ID\`  
   * Secret: \`SLACK\_BOT\_TOKEN\`

### **2.3.3. Future Journey (Evolución del Repositorio)**

A medida que el modelo de adopción madure y los equipos dominen la estructura, el gobierno del repositorio evolucionará hacia la automatización total para escalar la operación. Los siguientes puntos están definidos en el roadmap futuro:

#### **2.3.3.1. Validación Automatizada (Sustitución del Encargado de Formato)**

Implementación de un pipeline de CI (Continuous Integration) que ejecute un Linter sobre los Merge Requests. Este bot reemplazará la revisión manual de formato verificando automáticamente:

* Reglas de expresiones regulares para nombres de archivos (^\[a-z0-9-\]+.md$).

* Presencia de campos obligatorios en los Markdown.

* Validación de enlaces rotos en las referencias cruzadas.

# **3\. Plantillas de archivos para el árbol de directorios**

## 3.1. Plantillas de los Chapters

Cada archivo dentro de un Chapter debe copiar de manera exacta estas plantillas. Ningún campo debe ser omitido; si un campo no aplica, debe indicarse explícitamente "No aplica para este caso" justificando el motivo.

### **3.1.1. El archivo README.md**

Este archivo es la puerta de entrada y el punto de referencia rápido para el repositorio del Chapter. Su contenido es **inmutable en estructura** y debe resumir las reglas críticas de la arquitectura, nomenclatura y flujo de trabajo para garantizar el cumplimiento del estándar y facilitar la incorporación de nuevos colaboradores.

```shell
# Pragma Knowledge: Chapter [NOMBRE_DEL_CHAPTER]

## Propósito
Este repositorio es la fuente única de verdad para el conocimiento del Chapter de [NOMBRE]. Aquí almacenamos nuestras decisiones técnicas, límites, referencias, procedimientos y capacidades de negocio. Todo el contenido está diseñado para alimentar el Agente de IA corporativo.

## Estructura
- `kb/`: Knowledge Base. Carpeta de trabajo donde se escribe el conocimiento del chapter, siguiendo la **Regla de 5 Niveles** y el **Versionado vX/**.

## Gobernanza y Flujo de Trabajo
1. Trabajamos bajo **Trunk-Based Development** (TBD).
2. Todos los aportes se hacen mediante ramas `feature/*` y Merge Requests (MRs).
3. Cada MR requiere: a) Aprobación técnica por un par. b) Aprobación estructural por el **Encargado de Validación**.

---

## 🔥 **RESUMEN DEL ESTÁNDAR PARA EL CHAPTER**

Este resumen condensa las reglas innegociables para contribuir al `kb/` del Chapter. La consulta exhaustiva se encuentra en el Enlace al Estándar Global Completo.

### 1. Nomenclatura Estricta
*   **Archivos y Carpetas:** Todo debe estar en **inglés** y **kebab-case** (minúsculas separadas por guiones).
*   **Subdirectorios de Agrupación:** Deben ser de **una sola palabra** (ej. `java/`, `cloud/`).
*   **Límite de Profundidad:** Máximo **5 directorios** de agrupación a partir de `kb/`. El directorio de versión `vX/` es la única excepción y puede ser Nivel 6.

### 2. Contenido y Contribución
*   **Idioma:** Todo el texto debe ser en **español técnico**.
*   **Principio DRY:** No duplique información. Haga mención contextual del documento relacionado (nombre y categoría) cuando necesite referenciar conocimiento existente.
*   **Templates y Flujo:** Cree rama `feature/*`, **copie la plantilla correspondiente de la Sección 3 del Estándar** y llene **TODOS** sus campos obligatorios.

---

## Maintainers
- [Nombre del Lead]
- [Nombre del Encargado de Validación de Formato]

## Enlace al Estándar Global Completo
[Consulta Exhaustiva del Estándar de Gestión de Conocimiento Pragma](https://docs.google.com/document/d/1j5uZabIknVMYRQWalqMnicOoQ8yZMYMOVB8kLUmFGMk/edit?usp=sharing)
```

### **3.1.2. La Perspectiva (perspective.md)**

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

### **3.1.3. Decisiones (decisions/000-decision-name.md)**

Registro histórico de compromisos técnicos. Obligatorio usar el estándar ADR.

```shell
# [ID_NUMERICO_3_DIGITOS] - [TITULO_DE_LA_DECISION]

## Estado
[Propuesto | Aprobado | Depreciado | Superado por ID-nueva-decision]

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
```

### **3.1.4. Límites (limits/limit-name.md)**

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

### **3.1.5. Referencias (references/reference-name.md)**

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

### **3.1.6. Soluciones de Negocio (business-solutions/)**

Esta carpeta es el corazón del repositorio para los Agentes de IA y el equipo comercial/arquitectura. Contiene la materialización del conocimiento en capacidades vendibles y ejecutables.

#### **3.1.6.1. Jobs To Be Done (standard/jtbd/jtbd-title.md)**

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
Liste de forma exhaustiva la información, accesos, artefactos, confdiciones previas o dependencias necesarias por parte del cliente o de otros equipos antes de poder iniciar este Job.
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
- Dependencia 1: [Descripción]
- Dependencia 2: [Descripción]

## Salidas esperadas (Outputs)
Describa qué resultados palpables, documentos, repositorios o capacidades produce esta solución.
- Salida 1: [Descripción contextual del output]
- Salida 2: [Descripción contextual del output]

## Riesgos y consideraciones
Liste limitaciones del mercado, puntos críticos técnicos, o aspectos organizacionales que deben vigilarse estrechamente durante la ejecución para evitar el fracaso del Job.
- Riesgo 1: [Descripción]
- Riesgo 2: [Descripción]

## Variantes comunes
Indique adaptaciones frecuentes de esta solución (Ej. "Versión acelerada para startups", "Versión de alto cumplimiento para entidades financieras").

## Relación con otros chapters
Explique con qué otros chapters (ej. QA, Cloud, Data) suele integrarse esta solución, por qué es necesaria la sinergia y en qué momento del flujo interactúan.
```

#### **3.1.6.2. Generables / Entregables (standard/outputs/output-title.md)**

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

### **3.1.7. Particularidad de Cliente (clients/\[client-name\]/\[category\]/vX/file-name.md)**

Esta plantilla se utiliza para documentar extensiones, modificaciones o restricciones que aplican únicamente en el contexto de un cliente específico. Su uso es obligatorio dentro de las carpetas funcionales (**decisions/**, **limits/**, **references/**) de cada cliente.

```shell
# Particularidad: [TITULO_DESCRIPTIVO_DEL_TEMA]

## Cliente
[Nombre completo del Cliente, ID, o nombre de proyecto en kebab-case]

## Tipo de Documento Global Especializado
[Decisión | Límite | Referencia]

## Documento Global Base
Indique el nombre y la categoría del documento en el repositorio global del Chapter que esta particularidad está extendiendo, modificando o anulando para este cliente. (Principio DRY).

- Relación: Nombre y categoría del documento global a especializar

## Contexto de la Especialización
Describa la razón específica de esta diferencia. ¿Es un requisito legal, una limitación de infraestructura/tecnología del cliente, o una decisión particular de negocio?

## Detalle de la Particularidad
Describa con exactitud la especificación única que aplica para este cliente y cómo difiere del estándar global. 

## Mantenedores
[Nombre de la persona o equipo responsable de mantener esta documentación de cliente]
```

