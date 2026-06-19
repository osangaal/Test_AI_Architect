# Bloque 1: Diseño de Arquitectura (40%)

**Modalidad:** en casa · **Entrega:** antes de la entrevista (en el plazo que se te indique)

Lee este documento completo antes de empezar.

---

## Instrucciones importantes

Esta prueba evalúa tu capacidad como **Arquitecto de Azure e IA**. **No** te pedimos que implementes nada: queremos tu **arquitectura objetivo**, tu razonamiento sobre **trade-offs** y un **diagrama claro**.

Tienes libertad para usar herramientas de otros clouds (AWS, GCP) u open-source, **PERO** deberás:

1. Explicar **por qué** elegiste esa herramienta sobre la opción Azure equivalente.
2. Mostrar que **entiendes cuál sería el equivalente Azure** de lo que elegiste.
3. **Justificar TODAS** tus decisiones técnicas.

> Todo lo que no esté especificado en este documento es una **decisión de diseño deliberada** que tú debes tomar y justificar.

> Preferencia de la firma: **Azure-first**. Usa servicios Azure-nativos donde tenga sentido y justifica cualquier elección de terceros.

---

## 1. Contexto de negocio

La firma es un fondo de inversión privado que evalúa proyectos mineros. Ha construido **una plataforma de inteligencia documental para *due diligence* minera**: un sistema de IA que ayuda a sus analistas a hacer due diligence técnico de grado inversión sobre **reportes técnicos mineros** (estándar **NI 43-101**: PDFs grandes y densos, a menudo de 100 a más de 1.000 páginas, sobre geología, recursos/reservas, metalurgia, economía y riesgo).

> 📂 En la carpeta [Insumos/](./Insumos) hay **PDFs de ejemplo** de este tipo de reporte (NI 43-101) para que dimensiones el reto: tamaño, estructura y tipo de contenido. Son material de referencia; no tienes que procesarlos.

Hoy la plataforma permite a un analista:

- **Subir un reporte técnico (PDF)** y obtener extracción estructurada: tablas de recursos/reservas, métricas clave (ley, tonelaje, NPV/IRR, AISC, CapEx/OpEx), un SWOT y un resumen del proyecto.
- **Conversar con el documento** (chat): preguntas expertas con respuestas fundamentadas y con citas (opcionalmente apoyadas en búsqueda web).
- **Analizar varios reportes juntos** (un "grupo de proyecto") para comparación cruzada.
- Un **pipeline de back-office** que ya parseó y clasificó un corpus de **varios miles de reportes históricos** (resolución de entidades: qué reportes pertenecen al mismo proyecto físico pese a cambios de nombre de la empresa; etapa del ciclo de vida de cada reporte).

Los usuarios son un **equipo experto reducido** (evaluadores internos y algunos socios externos). El uso es **profundo, no masivo**: unos pocos *power users* hacen preguntas muy técnicas.

## 2. El problema de negocio

El *due diligence* minero es lento y el talento experto es escaso. Un evaluador senior lee cientos de páginas para responder lo que un junior no puede. La firma quiere **escalar esa experticia**:

- El **conocimiento institucional** (cómo un analista senior evalúa un proyecto: qué verificar, qué desconfiar) hoy vive solo en la cabeza de las personas.
- El **gran corpus histórico** (objetivo: decenas de miles de reportes) está subutilizado porque no es consultable como base de conocimiento.

## 3. Objetivo técnico (lo que la arquitectura debe habilitar)

El objetivo es **evolucionar de un chat sobre un solo documento a un asistente experto respaldado por una base de conocimiento (RAG)**, con una **capa de reglas/metodología** y un **mecanismo de evaluación de calidad**. En términos técnicos, esto se traduce en tres capacidades:

1. **Capa de reglas / metodología** — la lógica de evaluación del dominio (categorías de recursos, lógica por etapa de estudio, chequeos financieros, heurísticas de verificación) codificada como **flujos y reglas estructuradas**, no como un prompt plano.
2. **Recuperación / RAG** — una base de conocimiento sobre el corpus histórico + análisis previos + criterios codificados, para que el asistente razone con contexto acumulado.
3. **Loop de calidad / evaluación** — calidad de respuesta **medible** (fundamentación, honestidad ante vacíos de datos, corrección), de modo que los cambios se puedan validar.

Más un **pipeline de ingesta escalado** para procesar el corpus histórico completo (decenas de miles de PDFs grandes) y mantenerlo al día.

## 4. Punto de partida del caso (estado actual y limitaciones)

El punto de partida del caso es un **MVP funcional en producción sobre Azure** que **no se construyó para escala enterprise**. Su forma actual:

- **Frontend:** SPA web (hosting estático).
- **Backend:** un único servicio API Node/Express + base de datos relacional (PostgreSQL).
- **IA:** llamadas a un **LLM gestionado de un proveedor externo (no-Azure)**; abierto a migrar a Azure OpenAI / Azure AI. Los PDFs se suben a una File API del LLM; los documentos largos se trocean y procesan con paralelismo acotado.
- **Auth:** sesión por cookie (una sola organización hoy; modelo "contenido compartido, conversaciones privadas").

Limitaciones que la arquitectura objetivo debe resolver:

- **Instancia única con estado:** la cola de jobs en background y las sesiones de chat están **en memoria** → no escala horizontalmente y pierde trabajo en vuelo al reiniciar/desplegar.
- **Sin pipeline asíncrono durable** para análisis largos (algunos jobs tardan minutos; la plataforma pelea contra el timeout HTTP de 230s).
- **Sin observabilidad centralizada** (logs/métricas/tracing), sin tracking de costo de uso de LLM.
- **Higiene de esquema/despliegue débil** (sin migraciones de BD gobernadas).
- **Sin capa de recuperación/vectorial** (la capacidad RAG / memoria institucional no existe aún).
- **Supuestos single-tenant:** multi-org / aislamiento de datos no está diseñado.

## 5. Escala y requisitos no funcionales (objetivos de diseño)

**Diseña para el estado objetivo, no para los números de hoy:**

| Dimensión | Hoy | Objetivo a diseñar |
|---|---|---|
| Usuarios concurrentes | equipo experto reducido | **100–300** (expandible), con power users en ráfagas |
| Corpus de documentos | varios miles | **decenas de miles (50.000–100.000+)** PDFs técnicos |
| Tamaño de documento | hasta ~150 MB / 1.000+ páginas | igual; debe manejar docs grandes con elegancia |
| Latencia | minutos (semi-síncrono) | análisis **asíncrono con progreso**; **chat: p95 < 5 s** |
| Calidad de respuesta | sin medición formal | **relevancia / fundamentación > 85%**, medible con un *golden set* |
| Ingesta | manual / batch | **pipeline de alto throughput** (extracción, clasificación, resolución de entidades, embedding) |
| Tenencia | una sola org | **multi-org / basada en roles, con aislamiento de datos** |

Prioridades no funcionales: **seguridad y aislamiento de datos**; **fiabilidad/durabilidad** (sin jobs perdidos); **observabilidad** (logs, métricas, tracing, costo/uso de LLM); **eficiencia de costo** (el gasto de LLM es un driver principal); **reproducibilidad**; y un **camino claro para añadir las capacidades de RAG / reglas / evaluación**.

## 6. Restricciones y preferencias

- **Cloud:** **Azure-first** (la firma está comprometida con Azure). Usa servicios Azure-nativos donde tenga sentido y justifica cualquier elección de terceros.
- **LLM:** flexible en proveedor, pero **Azure OpenAI / Azure AI Foundry** es preferido para una postura enterprise; explica cómo mantenerte **portable**.
- **Residencia de datos / seguridad:** los documentos son **comercialmente sensibles**; asume que la firma exige control de acceso, auditabilidad y **mantener el contenido propietario dentro de su tenant**.
- **Equipo:** equipo de ingeniería **pequeño** → favorece **servicios gestionados** y simplicidad operativa sobre infraestructura a medida.

---

## 7. El reto de arquitectura (qué debes diseñar)

Diseña la **arquitectura enterprise objetivo** sobre Azure, cubriendo estas **8 áreas**. Cada una debe aparecer en el diagrama y estar justificada en tu documento de decisiones (qué elegiste, por qué, qué rechazaste).

| # | Área | Qué debe resolver |
|---|------|-------------------|
| **1** | **Ingesta y procesamiento de documentos** | Pipeline durable, escalable y asíncrono: upload → almacenamiento → extracción/OCR/parsing → clasificación y resolución de entidades → chunking → embedding/indexado. Manejar PDFs muy grandes y alto volumen. |
| **2** | **Capa de IA** | Orquestación de LLM (extracción, resumen, chat), **RAG / recuperación vectorial** para la memoria institucional, un lugar para la **capa de reglas/metodología** y un **mecanismo de evaluación/calidad**. Controles de costo y caching. |
| **3** | **Aplicación y API** | Servicios **stateless** y escalables horizontalmente; manejo de **jobs/colas durable** (sin estado en memoria); **progreso en tiempo real** para jobs largos. |
| **4** | **Capa de datos** | Relacional + vectorial + blob/objeto; cómo encajan; **aislamiento multi-tenant**. |
| **5** | **Seguridad e identidad** | AuthN/Z, secretos, aislamiento de tenant, auditabilidad, protección de datos. |
| **6** | **Observabilidad y FinOps** | Logging, métricas, tracing y **tracking de costo/uso de LLM**; alertas. |
| **7** | **DevOps / entrega** | CI/CD, entornos, **migraciones de BD gobernadas**, IaC. |
| **8** | **Escalabilidad, fiabilidad y costo** | Cómo escala a los objetivos del §5 y cómo **controla el gasto de LLM**. |

---

## 8. Qué entregas

### 1. Diagrama de arquitectura objetivo

Servicios Azure, flujos de datos y **el camino de IA/RAG** explícito. En la herramienta que prefieras (draw.io, Excalidraw, Lucidchart, PowerPoint, incluso foto de un boceto legible).

> **OBLIGATORIO:** incluye una **leyenda que distinga claramente los componentes Azure vs. no-Azure.**

Entrégalo como `docs/arquitectura.png` / `.pdf` o como enlace embebido en tu documento de decisiones.

### 2. Documento de decisiones (`docs/DECISIONES.md`, conciso)

- **Visión general:** resumen de la solución y del flujo de datos de extremo a extremo.
- **Decisiones por área:** para **CADA una de las 8 áreas del §7**, indica **qué elegiste, por qué, y qué rechazaste** (la alternativa descartada y la razón). Aborda explícitamente los trade-offs clave: **Azure OpenAI vs. LLM externo**, **elección de vector store**, **síncrono vs. asíncrono**, **enfoque de multi-tenancy**, **build vs. buy**.
- **Calidad y latencia medibles:** cómo cumples y cómo **medirías** los objetivos no funcionales del §5 — en concreto **chat p95 < 5 s** y **relevancia/fundamentación > 85%** (más honestidad ante vacíos de datos y corrección). **Nombra métricas concretas** (cómo se calculan y con qué herramienta, p. ej. un *golden set*); "los usuarios quedan satisfechos" no cuenta.

### 3. Boceto de migración (`docs/MIGRACION.md` o sección)

Cómo pasar del **MVP actual** (servicio único con estado, jobs en memoria, sin recuperación) a la **arquitectura objetivo**, **en fases**. Qué harías primero, qué después y por qué.

### 4. Riesgos y costos

Los riesgos concretos de TU diseño y cómo los mitigas, **especialmente alrededor del uso de LLM a escala** (FinOps). Incluye: qué haces si **el modelo o un servicio crítico cae** (estrategia de fallback) y cómo garantizas el **aislamiento de datos entre organizaciones** y que el **contenido propietario no se exponga** (ni entre tenants ni hacia el proveedor de LLM).

---

## 9. Cómo se evalúa este bloque (peso: 40% del total)

Compartimos los pesos para que priorices tu esfuerzo:

| Criterio | Peso (dentro del Bloque 1) |
|---|---|
| Cobertura y solidez de las 8 áreas de diseño | 30% |
| Diseño de la capacidad **RAG + reglas + evaluación** | 15% |
| Justificación de decisiones (qué elegiste / por qué / qué rechazaste) | 20% |
| Escalabilidad, fiabilidad y **control de costo de LLM** (FinOps) | 15% |
| Seguridad, aislamiento multi-tenant y auditabilidad | 10% |
| Boceto de migración por fases (MVP → objetivo) | 5% |
| Claridad del diagrama y del documento (comunicación) | 5% |

> No buscamos "la única respuesta correcta", sino **juicio de arquitectura Azure/IA**, conciencia de trade-offs y comunicación clara de un diseño **defendible**.

---

## 10. Entrega

1. Crea **tu propio repositorio privado** en GitHub.
2. Estructura sugerida:
   ```
   tu-repo/
   └── docs/
       ├── DECISIONES.md        # decisiones por área + trade-offs + calidad medible
       ├── MIGRACION.md         # boceto de migración por fases (o sección en DECISIONES.md)
       └── arquitectura.png     # diagrama objetivo (o enlace en DECISIONES.md)
   ```
3. Invita como colaborador al usuario de GitHub **`osangaal`** (Settings → Collaborators → Add people).
4. Envía el enlace a **ogaspar@oceloteminerals.com** **dentro del plazo indicado**. El *timestamp* de tus commits valida el cumplimiento.
