# Agente Inteligente de IA con RAG — BimBam Buy

> Agente de IA capaz de responder preguntas en lenguaje natural utilizando la documentación oficial de **BimBam Buy** como base de conocimiento, mediante la técnica de RAG (Retrieval-Augmented Generation).

![Estado](https://img.shields.io/badge/estado-en%20desarrollo-yellow)
![n8n](https://img.shields.io/badge/n8n-cloud-orange)
![Vector Store](https://img.shields.io/badge/vector%20store-Pinecone-blueviolet)
![Embeddings](https://img.shields.io/badge/embeddings-Cohere-purple)

---

## 📋 Tabla de contenidos

- [Descripción del proyecto](#-descripción-del-proyecto)
- [Checklist del Challenge](#-checklist-del-challenge)
- [Demo / URL pública](#-demo--url-pública)
- [Arquitectura](#-arquitectura)
- [Tecnologías utilizadas](#-tecnologías-utilizadas)
- [Código fuente](#-código-fuente)
- [Base de conocimiento](#-base-de-conocimiento)
- [Instrucciones de instalación](#-instrucciones-de-instalación-desde-cero)
- [Instrucciones de ejecución](#-instrucciones-de-ejecución-uso-del-agente)
- [Notas técnicas y soluciones a problemas encontrados](#-notas-técnicas-y-soluciones-a-problemas-encontrados)
- [Ejemplos de preguntas y respuestas](#-ejemplos-de-preguntas-y-respuestas)
- [Evidencia del despliegue](#-evidencia-del-despliegue)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Decisiones de arquitectura](#-decisiones-de-arquitectura)
- [Trabajo futuro](#-trabajo-futuro)
- [Historial de desarrollo](#-historial-de-desarrollo)

---

## 📖 Descripción del proyecto

**BimBam Buy** es un e-commerce multiplataforma (ficticio) enfocado en la experiencia de compra digital ágil y segura, con políticas robustas de reembolso, un programa de afiliados dinámico y una infraestructura logística optimizada para entregas rápidas y soporte constante al usuario final.

Este proyecto implementa un **agente conversacional de IA** que responde preguntas de los usuarios/clientes de BimBam Buy basándose exclusivamente en su documentación oficial (políticas, garantías, medios de pago, envíos, afiliados), utilizando **RAG (Retrieval-Augmented Generation)**: el agente primero busca los fragmentos relevantes de la documentación real y luego genera una respuesta fundamentada en esa información, evitando alucinaciones o respuestas inventadas. Simula una aplicación en entorno real de producción para una empresa.

**Problema que resuelve:** reduce el tiempo de respuesta a consultas frecuentes de soporte al cliente (reembolsos, envíos, garantías, medios de pago, afiliados), permitiendo respuestas inmediatas y consistentes basadas en la documentación oficial vigente, sin depender de un agente humano disponible en todo momento.

---

## ✅ Checklist del Challenge

| Requisito | Estado |
|---|---|
| Agente funcional que responda preguntas en lenguaje natural usando documentos | ✅ Completo y probado — ver [Ejemplos de preguntas y respuestas](#-ejemplos-de-preguntas-y-respuestas) |
| Documentación utilizada para alimentar el RAG | ✅ Completo — 5 PDFs de BimBam Buy, 124 fragmentos vectorizados |
| Código del proyecto en repositorio GitHub organizado, con URL pública y acceso público | ✅ Completo |
| README con descripción, arquitectura, tecnologías, código fuente, instrucciones de instalación y ejecución, ejemplos de preguntas/respuestas | ✅ Completo |
| Evidencia del deploy (capturas/video) dentro del README | ✅ Completo — capturas de las 5 categorías + video del workflow en ejecución |
| Historial de commits | ✅ En curso, ver [Historial de desarrollo](#-historial-de-desarrollo) |
| Deploy disponible mediante URL pública | ⏳ Pendiente — falta publicar el Chat Trigger de n8n con "Make Chat Publicly Available" |

---

## 🌐 Demo / URL pública

- **Chat del agente (n8n Cloud):** `[PENDIENTE: URL del Chat Trigger publicado]`
- **Instancia de n8n:** https://jorgegomezpacheco.app.n8n.cloud/
- **Repositorio GitHub:** https://github.com/jorgegomezpacheco/agente-ia-rag

---

## 🏗 Arquitectura

```
                    Usuario (navegador, sin registro)
                              │
                              ▼
        ┌──────────────────────────────────────────────────┐
        │              n8n Cloud (jorgegomezpacheco)          │
        │                                                      │
        │   Chat Trigger                                       │
        │       │                                               │
        │       ▼                                               │
        │   Chat Memory Manager ──► Redis Chat Memory            │
        │   (lee historial reciente, no escribe)  (misma sesión) │
        │       │                                               │
        │       ▼                                               │
        │   Basic LLM Chain (clasificador, CON contexto)         │
        │       │                                               │
        │       ▼                                               │
        │   IF ¿relacionado con BimBam Buy?                     │
        │       │                        │                       │
        │      SÍ                       NO                      │
        │       ▼                        ▼                       │
        │   AI Agent                 Set (mensaje fijo)          │
        │    ├── Ollama Chat Model                               │
        │    ├── Redis Chat Memory (TTL 40 min, ventana 18)       │
        │    └── 5 Tools Pinecone (uno por categoría)             │
        └──────────────────────────────────────────────────┘
                                          │
                                          ▼
                          ┌───────────────────────────┐
                          │  Workflow de carga (aparte)  │
                          │  HTTP Request → Code (fix     │
                          │  mimeType) → Pinecone Vector  │
                          │  Store (Insert)               │
                          └───────────────────────────┘
```

**Flujo de consulta (usuario final):**
1. El usuario abre la URL pública del chat y escribe su pregunta.
2. El **Chat Memory Manager** recupera el historial reciente de la sesión (sin modificarlo), dándole contexto al clasificador.
3. El **Basic LLM Chain** clasifica si la pregunta actual —considerando ese historial— está relacionada con alguno de los 5 temas de BimBam Buy, respondiendo únicamente "SI" o "NO".
4. Si es "NO", un nodo **Set** responde con un mensaje fijo indicando el alcance del agente, sin consumir tokens del AI Agent principal.
5. Si es "SI", el **AI Agent** procesa la pregunta con su propia memoria de conversación (Redis, TTL 40 min) y elige, según el tema, cuál de los 5 Tools de Pinecone consultar.
6. Los fragmentos recuperados + la pregunta se envían a **Ollama Cloud** (temperatura 0.3, para minimizar alucinaciones), que genera la respuesta final.
7. La respuesta se muestra al usuario en la misma ventana de chat.

**Flujo de carga de documentos (administrador, proceso aparte, no público):**
1. Se ejecuta manualmente (Manual Trigger) el workflow **"Carga de Documentos - BimBam Buy"**.
2. Recorre en loop los 5 PDFs, descarga cada uno, corrige su mime type, extrae el texto, lo fragmenta, lo vectoriza (Cohere) y lo inserta en Pinecone con su metadata (`doc_id`, `categoria`, `source`).
3. ✅ **Verificado y funcional**: 124 fragmentos insertados correctamente en Pinecone a partir de los 5 documentos.

---

## 🛠 Tecnologías utilizadas

| Tecnología | Rol en el proyecto | Costo |
|---|---|---|
| **n8n Cloud** | Orquestador del agente: workflows visuales, Chat Trigger (interfaz + URL pública), AI Agent | Free trial / plan básico |
| **Ollama Cloud** (`gpt-oss:20b`) | Modelo de lenguaje (LLM) vía API — genera respuestas (temperatura 0.3) y clasifica preguntas | Gratis (con límites de uso) |
| **Cohere** (`embed-multilingual-v3.0`) | Genera los embeddings (vectores) tanto de los documentos como de las preguntas del usuario | Gratis (con límites de uso) |
| **Pinecone** | Vector Store — almacena y busca los fragmentos de documentación por similitud semántica, con soporte de metadata | Gratis (plan Starter) |
| **Redis Cloud** | Memoria de conversación persistente, con expiración automática por inactividad (TTL) | Gratis (plan de 30 MB) |

---

## 💻 Código fuente

| Archivo | Qué contiene | Formato |
|---|---|---|
| [`workflows/carga-documentos.json`](./workflows/carga-documentos.json) | Workflow de carga inicial: recorre los 5 PDFs de BimBam Buy, corrige su mime type, los vectoriza e inserta en Pinecone con metadata. Verificado: 124 fragmentos insertados. | JSON (exportado de n8n) |
| [`workflows/agente-rag.json`](./workflows/agente-rag.json) | Workflow principal: Chat Trigger, guardrail (Chat Memory Manager + Basic LLM Chain + IF), AI Agent con Ollama, Redis Chat Memory y 5 Tools de Pinecone. Probado con 6 casos reales. | JSON (exportado de n8n) |
| [`docs/documentacion-empresa/`](./docs/documentacion-empresa) | Los 5 PDFs oficiales de BimBam Buy usados como base de conocimiento | PDF |

> 💡 Los workflows de n8n son "código visual": cada nodo equivale a un bloque de lógica, y el archivo `.json` exportado contiene toda esa lógica en formato texto — por lo tanto, es código versionable y revisable, aunque no se escriba línea por línea como Python o JavaScript.

> 🔧 Un tercer workflow de mantenimiento (actualización/incorporación de nuevos documentos) está diseñado pero pendiente de implementación — ver [Trabajo futuro](#-trabajo-futuro).

---

## 📚 Base de conocimiento

La base de conocimiento del agente está compuesta por 5 documentos oficiales (ficticios) de BimBam Buy:

| Documento | `doc_id` | Categoría | Contenido |
|---|---|---|---|
| Política de Reembolsos y Devoluciones de BimBam Buy | `politica-reembolsos-devoluciones` | `reembolsos-devoluciones` | Condiciones, plazos y proceso para solicitar reembolsos y devoluciones |
| Programa de Afiliados de BimBam Buy | `programa-afiliados` | `afiliados` | Funcionamiento del programa de afiliados, comisiones y requisitos |
| Guía de Tiempos y Costos de Envío de BimBam Buy | `guia-tiempos-costos-envio` | `envios` | Tiempos estimados de entrega y costos según destino/modalidad |
| Preguntas Frecuentes sobre Métodos de Pago de BimBam Buy | `faq-metodos-pago` | `pagos` | Medios de pago aceptados y resolución de problemas comunes |
| Manual de Garantía de Productos de BimBam Buy | `manual-garantia-productos` | `garantias` | Cobertura, plazos y proceso de garantía de productos |

Los documentos fuente se encuentran en [`/docs/documentacion-empresa`](./docs/documentacion-empresa). En conjunto, generaron **124 fragmentos vectorizados** en el índice de Pinecone.

---

## 🚀 Instrucciones de instalación (desde cero)

Esta sección permite que **cualquier persona, sin conocer el proyecto previamente**, pueda replicar la implementación completa desde cero.

### Requisitos previos
- Cuenta de [n8n Cloud](https://n8n.io/cloud/)
- Cuenta de [Ollama](https://ollama.com/) (API Key para Ollama Cloud)
- Cuenta de [Cohere](https://cohere.com/) (API Key)
- Cuenta de [Pinecone](https://www.pinecone.io/) (API Key)
- Cuenta de [Redis Cloud](https://redis.io/try-free/) (host, puerto, password)
- Cuenta de GitHub (para acceder a los documentos fuente vía URL raw)

### Pasos de instalación

1. **Crear el índice en Pinecone**: nombre `agente-ia-rag`, dimensión `1024` (compatible con `embed-multilingual-v3.0` de Cohere), métrica `cosine`, tipo de vector `Denso`.

2. **Crear una base de datos en Redis Cloud** (plan gratuito, 30 MB es suficiente).

3. **Crear cuenta en n8n Cloud** y configurar las credenciales:
   - *Ollama*: Base URL `https://ollama.com` + API Key.
   - *Cohere*: API Key.
   - *Pinecone*: API Key.
   - *Redis*: host, puerto, password (SSL deshabilitado según la configuración de este proyecto).

4. **Clonar/consultar este repositorio** para obtener los workflows y documentos:
   ```bash
   git clone https://github.com/jorgegomezpacheco/agente-ia-rag.git
   ```

5. **Importar los workflows en n8n** (`Workflows → Import from File`):
   - `workflows/carga-documentos.json`
   - `workflows/agente-rag.json`

6. **Cargar la base de conocimiento**: abrir `carga-documentos.json` y ejecutar manualmente (Manual Trigger) — procesa los 5 PDFs de BimBam Buy y los inserta en Pinecone con su metadata correspondiente.

7. **Publicar el agente**: abrir `agente-rag.json` → nodo Chat Trigger → activar **"Make Chat Publicly Available"** → modo **"Hosted Chat"** → **Publish/Active** → copiar la Chat URL generada.

Con esto, la instalación queda completa y el sistema listo para usarse (ver sección siguiente).

---

## ▶️ Instrucciones de ejecución (uso del agente)

Esta sección explica cómo **usar** el proyecto ya instalado y desplegado — a diferencia de la sección anterior, que explica cómo instalarlo desde cero.

### Para el usuario final (consultar al agente)
1. Abrir la **Chat URL pública** del agente (ver [Demo / URL pública](#-demo--url-pública)).
2. Escribir una pregunta en lenguaje natural relacionada con BimBam Buy — por ejemplo:
   - *"¿Cuánto tiempo tarda un reembolso?"*
   - *"¿Cómo me uno al programa de afiliados?"*
   - *"¿Qué garantía tienen los productos?"*
3. El agente responde en la misma ventana, basándose exclusivamente en la documentación oficial cargada.
4. La conversación mantiene contexto mientras la sesión esté activa (hasta 40 minutos de inactividad) — se pueden hacer preguntas de seguimiento sin repetir el contexto completo, incluso con referencias implícitas (ej. "¿y qué pasa si ya pasó ese plazo?").
5. Si se pregunta algo fuera del alcance de BimBam Buy, el agente lo indica amablemente y sugiere contactar a soporte humano, sin intentar inventar una respuesta.

### Para el administrador (mantener la base de conocimiento)
- **Cargar documentos por primera vez**: ejecutar manualmente `carga-documentos.json` desde el editor de n8n (botón "Execute workflow").
- **Actualizar o agregar nuevos documentos**: funcionalidad planeada, ver [Trabajo futuro](#-trabajo-futuro).
- **Monitorear ejecuciones**: en n8n, pestaña "Executions" de cada workflow, para verificar que las ejecuciones terminen sin errores.

---

## 🔧 Notas técnicas y soluciones a problemas encontrados

### 1. Mime type incorrecto al descargar PDFs desde GitHub raw

**Problema:** los PDFs descargados desde `raw.githubusercontent.com` llegan con mime type genérico `application/octet-stream` en lugar de `application/pdf`, provocando que el **Default Data Loader** los rechazara.

**Solución:** nodo **Code (JavaScript)** entre `HTTP Request` y `Pinecone Vector Store` que sobrescribe el mime type manualmente:
```javascript
for (const item of $input.all()) {
  item.binary.data.mimeType = 'application/pdf';
}
return $input.all();
```

### 2. Filtrado dinámico de metadata no soportado en Tools de AI Agent

**Problema:** n8n no permite usar `$fromAI()` en el filtro de metadata de un Vector Store conectado como Tool de un AI Agent.

**Solución:** se implementaron **5 herramientas Pinecone independientes** (una por categoría), cada una con un filtro de metadata fijo y una descripción clara, permitiendo que el AI Agent elija automáticamente la herramienta correcta según el tema de la pregunta.

### 3. Clasificador sin memoria de conversación

**Problema:** un guardrail (clasificador de tema) que evalúa cada mensaje de forma aislada rechaza incorrectamente preguntas de seguimiento con referencias implícitas (ej. "¿y qué pasa si ya pasó ese plazo?" tras haber hablado de reembolsos).

**Solución:** se agregó un nodo **Chat Memory Manager** (modo "Get Many Messages") antes del clasificador, apuntando a la misma sesión de **Redis Chat Memory** que usa el AI Agent — de forma que el clasificador *lee* el historial reciente sin escribir en él (evitando contaminar la memoria real de la conversación). El prompt del clasificador fue ajustado para usar ese historial como contexto, pero sin asumir que toda la sesión es válida solo porque comenzó siendo relevante:
```javascript
{{ $json.messages.map(m => (m.human ? 'Usuario: ' + m.human + '\n' : '') + (m.ai ? 'Asistente: ' + (typeof m.ai === 'string' ? m.ai : JSON.stringify(m.ai)) : '')).join('\n') }}
```
(Se corrigió también un bug inicial donde la expresión asumía una estructura `{type, text}` que no correspondía al formato real devuelto por Redis Chat Memory, que usa claves `human`/`ai`/`tool` por turno.)

### 4. El nodo Pinecone Vector Store de n8n no soporta borrar por metadata

**Hallazgo:** a diferencia de lo asumido inicialmente, el nodo visual **Pinecone Vector Store** de n8n no expone una operación de borrado por filtro de metadata — es una limitación reportada por la comunidad de n8n desde 2024, aún sin resolver de forma nativa. Pinecone como servicio sí soporta este borrado vía su API REST, pero el nodo de n8n no lo implementa en su interfaz visual.

**Implicancia:** el workflow de mantenimiento (actualizar o agregar documentos con borrado limpio) requiere un nodo **HTTP Request** llamando directamente al endpoint de borrado de la API de Pinecone, en vez de únicamente el nodo Pinecone Vector Store estándar. Ver diseño planeado en [Trabajo futuro](#-trabajo-futuro).

---

## 💬 Ejemplos de preguntas y respuestas

Pruebas reales realizadas sobre el agente en ejecución, cubriendo las 5 categorías y el rechazo de preguntas fuera de alcance:

| # | Pregunta del usuario | Categoría / Tool | Resumen de la respuesta del agente |
|---|---|---|---|
| 1 | ¿Qué cubre la garantía de productos? | `buscar_garantias` | Lista de fallas cubiertas (defectos de fabricación, ensamblaje, problemas de encendido) y exclusiones (daños por golpes, agua, manipulación, desgaste normal), con resumen final. |
| 2 | ¿Qué métodos de pago aceptan? | `buscar_pagos` | Tarjeta de crédito/débito, transferencia bancaria, efectivo en puntos habilitados, billeteras digitales y financiamiento en cuotas (según país y disponibilidad). |
| 3 | ¿Cuánto cuesta y demora el envío? | `buscar_envios` | Tabla con tiempos y costos por zona (urbana, secundaria, cobertura extendida), más detalles de preparación del pedido y métodos de envío disponibles. |
| 4 | ¿Cómo me hago afiliado? | `buscar_afiliados` | Requisitos, proceso de postulación paso a paso, tiempos de revisión (1-3 días hábiles), qué incluye la aprobación y cómo se generan y liquidan las comisiones. |
| 5 | ¿Cuánto tarda un reembolso? | `buscar_reembolsos` | Plazo estándar de 5-10 días hábiles desde la aprobación, y factores que pueden extenderlo (método de pago, país, validaciones adicionales). |
| 6 | ¿Quién ganó el mundial? | *(guardrail — sin Tool)* | *"Lo siento, solo puedo responder preguntas relacionadas con reembolsos, programa de afiliados, envíos, métodos de pago o garantías de productos de BimBam Buy. Para otras consultas, contacta a soporte humano."* — rechazada correctamente sin consumir tokens del AI Agent. |

> ✅ Adicionalmente se verificó que preguntas de seguimiento con referencias implícitas (ej. *"¿y qué pasa si ya pasó ese plazo?"* tras una pregunta sobre reembolsos) son correctamente interpretadas gracias al guardrail con memoria de contexto (Chat Memory Manager + Redis), sin perder continuidad de la conversación.

Capturas de cada ejecución disponibles en la sección de [Evidencia del despliegue](#-evidencia-del-despliegue).

---

## 📸 Evidencia del despliegue

### Carga de documentos (base de conocimiento)

| Evidencia | Estado |
|---|---|
| Carga de documentos ejecutada con éxito en n8n (124/124 items, sin errores) | ✅ |
| Registros verificados en Pinecone (124 records, metadata correcta por `doc_id`) | ✅ |

![Carga de documentos exitosa en n8n](./screenshots/pinecone-carga-exitosa.png)

**Video demostrativo (carga de documentos funcionando):**

https://github.com/jorgegomezpacheco/agente-ia-rag/raw/main/screenshots/demo-carga-documentos.mp4

### Agente RAG en funcionamiento (5 categorías probadas)

Cada captura muestra la ejecución real del workflow `agente-rag.json` en n8n, con la Tool de Pinecone activada y la respuesta generada por el agente:

| Categoría | Captura |
|---|---|
| Garantías | ![Prueba garantías](./screenshots/agente-prueba-garantias.png) |
| Métodos de pago | ![Prueba pagos](./screenshots/agente-prueba-pagos.png) |
| Envíos | ![Prueba envíos](./screenshots/agente-prueba-envios.png) |
| Programa de afiliados | ![Prueba afiliados](./screenshots/agente-prueba-afiliados.png) |
| Reembolsos | ![Prueba reembolsos](./screenshots/agente-prueba-reembolsos.png) |

**Video demostrativo (Agente RAG completo en funcionamiento):**

https://github.com/jorgegomezpacheco/agente-ia-rag/raw/main/screenshots/demo-agente-rag.mp4

> Si los videos no se reproducen embebidos directamente en GitHub, descárgalos desde los enlaces de arriba o desde la carpeta [`/screenshots`](./screenshots).

`[PENDIENTE: captura del chat público respondiendo, una vez publicado el Chat Trigger con URL pública]`

---

## 📂 Estructura del repositorio

```
agente-ia-rag/
├── README.md                          # Este archivo
├── workflows/
│   ├── agente-rag.json                # Workflow principal del agente — probado
│   └── carga-documentos.json          # Workflow de carga inicial (5 PDFs) — verificado
├── docs/
│   └── documentacion-empresa/         # 5 PDFs oficiales de BimBam Buy
├── screenshots/                       # Evidencia visual y videos del proyecto funcionando
└── docker/                            # ⚠️ Referencia histórica — ver nota abajo
```

> ⚠️ **Nota sobre la carpeta `/docker`**: contiene el `docker-compose.yml` del intento inicial de despliegue en Oracle Cloud Infrastructure (OCI), descartado por indisponibilidad de capacidad de servidores ARM (ver [Decisiones de arquitectura](#-decisiones-de-arquitectura)). Se conserva únicamente como evidencia del proceso de desarrollo.

---

## 🧭 Decisiones de arquitectura

**OCI → n8n Cloud:** el proyecto se planificó originalmente para OCI (VM con n8n y Ollama autoalojados). Se presentó indisponibilidad reiterada de capacidad ("Out of host capacity") para instancias Ampere A1 en la región Chile Central (Santiago). El challenge permite explícitamente elegir la plataforma de despliegue, priorizando que el agente funcione correctamente — por lo que se migró a n8n Cloud. La configuración original se conserva en `/docker` como evidencia del proceso.

**Simple Vector Store → Pinecone:** se buscaba una base vectorial con mejor soporte de metadata y mayor robustez que el Simple Vector Store nativo de n8n (que no persiste datos entre reinicios), por lo que se optó por Pinecone.

**Un Tool de Pinecone por categoría:** n8n no permite usar `$fromAI()` en el filtro de metadata de un Vector Store conectado como Tool de un AI Agent. Como alternativa robusta y más realista a un entorno de producción departamentalizado, se implementaron 5 herramientas Pinecone independientes, cada una con su propio filtro fijo y una descripción clara que permite al AI Agent elegir automáticamente cuál usar.

**Guardrail (clasificador) antes del AI Agent:** para evitar que preguntas ajenas a BimBam Buy consuman tokens innecesariamente en el AI Agent principal (con sus 5 Tools y system prompt extenso), se agregó una etapa previa de clasificación (Basic LLM Chain) con temperatura muy baja, que decide si la pregunta amerita pasar al agente completo o recibir una respuesta fija indicando el alcance del asistente.

**Chat Memory Manager como "memoria de solo lectura" para el clasificador:** para que el guardrail entienda referencias implícitas en preguntas de seguimiento (ej. "¿y qué pasa si ya pasó ese plazo?") sin perder precisión ante temas realmente ajenos, se usa un Chat Memory Manager que *lee* el historial reciente de la misma sesión de Redis que usa el AI Agent, sin escribir en él — evitando así contaminar la memoria real de la conversación con las clasificaciones internas del guardrail.

**Redis Chat Memory con TTL:** se eligió Redis (en vez de Simple Memory) para la memoria del AI Agent porque soporta expiración automática por inactividad (Session TTL de 40 minutos), liberando recursos de sesiones abandonadas sin intervención manual — un comportamiento estándar en sistemas de chat de producción.

---

## 🔮 Trabajo futuro

**Workflow de mantenimiento (`actualizar-documento.json`)** — diseñado pero pendiente de implementación. No es un requisito explícito del challenge, se plantea como mejora de robustez del proyecto.

Diseño planeado:
1. **Trigger manual** (administrador), con los campos `url`, `doc_id` y `categoria` del documento a cargar o actualizar — por ejemplo, un nuevo documento **`contacto-soporte.pdf`** (`doc_id: contacto-soporte`, `categoria: contacto-soporte`), que ampliaría el alcance del agente a una sexta categoría de atención al cliente.
2. **Nodo HTTP Request** llamando directamente al endpoint de borrado de la API de Pinecone (`DELETE` con filtro por `doc_id` en el body), ya que el nodo visual de n8n no soporta esta operación de forma nativa (ver [Notas técnicas #4](#-notas-técnicas-y-soluciones-a-problemas-encontrados)). Este paso es seguro de ejecutar tanto para documentos existentes (borra los fragmentos obsoletos) como para documentos completamente nuevos (no encuentra nada que borrar, sin generar error).
3. **Descarga, corrección de mime type, fragmentación, vectorización e inserción** del documento — reutilizando el mismo patrón ya construido y verificado en `carga-documentos.json`.

Con este diseño, un mismo workflow serviría tanto para **actualizar un documento existente** como para **incorporar documentos completamente nuevos** a la base de conocimiento, sin necesidad de workflows separados. De implementarse, agregar un sexto Tool de Pinecone (`buscar_contacto_soporte`) al AI Agent para que pueda responder también sobre este nuevo tema.

---

## 📅 Historial de desarrollo

- [x] Definición de arquitectura y stack tecnológico inicial (OCI)
- [x] Estructura inicial del repositorio en GitHub
- [x] Intento de despliegue en OCI (bloqueado por indisponibilidad de capacidad)
- [x] Pivote de arquitectura a n8n Cloud + Ollama Cloud
- [x] Definición de la empresa ficticia (BimBam Buy) y sus 5 documentos base
- [x] Selección de Pinecone + Cohere como stack de vectorización
- [x] Creación del índice en Pinecone (1024 dim, cosine, denso)
- [x] Configuración de credenciales (Ollama, Cohere, Pinecone, Redis) en n8n
- [x] Carga y vectorización de los 5 documentos de BimBam Buy — 124 fragmentos, verificado en Pinecone
- [x] Resolución de incompatibilidad de mime type (PDF vía GitHub raw)
- [x] Construcción del AI Agent con 5 Tools de Pinecone por categoría
- [x] Implementación de guardrail (clasificador) para filtrar preguntas fuera de tema
- [x] Configuración de Redis Chat Memory con TTL (40 min) para el AI Agent
- [x] Corrección del guardrail para mantener contexto en preguntas de seguimiento (Chat Memory Manager)
- [x] Pruebas de preguntas/respuestas — 5 categorías + rechazo de tema ajeno, verificado
- [x] Evidencia visual completa (capturas por categoría + videos de ambos workflows)
- [ ] Publicación del Chat Trigger con URL pública
- [ ] Workflow de mantenimiento/actualización de documentos, incluyendo `contacto-soporte.pdf` (ver Trabajo futuro)
- [ ] Documentación final del README

---

## 👤 Autor

**Jorge Gómez Pacheco** — Proyecto desarrollado como parte de un challenge de implementación de agentes de IA con RAG.
