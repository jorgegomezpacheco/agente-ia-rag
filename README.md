# Agente Inteligente de IA con RAG — Consultor de Documentación

> Agente de IA capaz de responder preguntas en lenguaje natural utilizando una base de conocimiento documental (RAG), simulando una aplicación en entorno real de producción para una empresa.

![Estado](https://img.shields.io/badge/estado-en%20desarrollo-yellow)
![n8n](https://img.shields.io/badge/n8n-cloud-orange)
![LLM](https://img.shields.io/badge/LLM-Ollama%20Cloud-black)

---

## 📋 Tabla de contenidos

- [Descripción del proyecto](#-descripción-del-proyecto)
- [Demo / URL pública](#-demo--url-pública)
- [Arquitectura](#-arquitectura)
- [Tecnologías utilizadas](#-tecnologías-utilizadas)
- [Código fuente](#-código-fuente)
- [Base de conocimiento](#-base-de-conocimiento)
- [Instrucciones de implementación desde cero](#-instrucciones-de-implementación-desde-cero)
- [Ejemplos de preguntas y respuestas](#-ejemplos-de-preguntas-y-respuestas)
- [Evidencia del despliegue](#-evidencia-del-despliegue)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Decisiones de arquitectura](#-decisiones-de-arquitectura)
- [Historial de desarrollo](#-historial-de-desarrollo)

---

## 📖 Descripción del proyecto

Este proyecto implementa un **agente conversacional de IA** que responde preguntas de usuarios basándose exclusivamente en la documentación interna de una empresa (ficticia), utilizando la técnica de **RAG (Retrieval-Augmented Generation)**: en lugar de que el modelo "invente" respuestas, primero busca los fragmentos relevantes de la documentación y luego genera una respuesta fundamentada en esa información.

**Empresa ficticia utilizada:** `[PENDIENTE: nombre y breve descripción del rubro]`

**Problema que resuelve:** `[PENDIENTE: ej. "Reducir el tiempo de respuesta a consultas frecuentes de soporte, permitiendo que los usuarios obtengan respuestas inmediatas basadas en el manual/documentación oficial, sin esperar a un agente humano."]`

---

## 🌐 Demo / URL pública

- **Chat del agente (n8n Cloud):** `[PENDIENTE: URL del Chat Trigger, formato https://jorgegomezpacheco.app.n8n.cloud/webhook/xxxx/chat]`
- **Instancia de n8n:** https://jorgegomezpacheco.app.n8n.cloud/
- **Repositorio GitHub:** https://github.com/jorgegomezpacheco/agente-ia-rag

---

## 🏗 Arquitectura

```
                    Usuario (navegador, sin registro)
                              │
                              ▼
        ┌──────────────────────────────────────────┐
        │        n8n Cloud (jorgegomezpacheco)        │
        │                                              │
        │   Chat Trigger ──► AI Agent                  │
        │  (URL pública)         │                      │
        │                        ├── Ollama Cloud       │
        │                        │   (LLM, vía API)     │
        │                        │                      │
        │                        ├── Simple Memory      │
        │                        │   (historial de      │
        │                        │    conversación)     │
        │                        │                      │
        │                        └── Simple Vector Store│
        │                            (documentos, Tool) │
        └──────────────────────────────────────────┘
```

**Flujo de datos:**
1. El usuario abre la URL pública del chat (sin necesidad de registro ni instalación).
2. Escribe su pregunta en lenguaje natural.
3. El **AI Agent** recibe el mensaje junto con el historial de la conversación (Memory).
4. El agente consulta el **Vector Store** buscando los fragmentos de documentación más relevantes para la pregunta.
5. Esos fragmentos + la pregunta se envían al modelo de lenguaje (**Ollama Cloud**), que genera la respuesta final.
6. La respuesta se muestra al usuario en la misma ventana de chat.

---

## 🛠 Tecnologías utilizadas

| Tecnología | Rol en el proyecto | Costo |
|---|---|---|
| **n8n Cloud** | Orquestador del agente: workflows visuales, Chat Trigger (interfaz + URL pública), AI Agent | Free trial / plan básico |
| **Ollama Cloud** | Modelo de lenguaje (LLM) vía API, sin necesidad de infraestructura propia | Gratis (con límites de uso) |
| **Simple Vector Store** (nodo nativo de n8n) | Almacena los embeddings de la documentación para búsqueda semántica | Gratis, sin configuración adicional |
| **Simple Memory** (nodo nativo de n8n) | Mantiene el historial de conversación durante la sesión del usuario | Gratis, sin configuración adicional |

---

## 💻 Código fuente

Este proyecto se implementa mediante **workflows visuales de n8n**, exportables como archivos `.json` — el equivalente a "código fuente" en una plataforma de automatización low-code/no-code.

| Archivo | Qué contiene | Formato |
|---|---|---|
| [`workflows/agente-rag.json`](./workflows/agente-rag.json) | Workflow principal: Chat Trigger + AI Agent + conexiones a Ollama Cloud, Memory y Vector Store | JSON (exportado de n8n) |
| [`workflows/carga-documentos.json`](./workflows/carga-documentos.json) | Workflow auxiliar que lee los documentos de `/docs`, los divide en fragmentos, genera embeddings y los guarda en el Vector Store | JSON (exportado de n8n) |
| [`docs/documentacion-empresa/`](./docs/documentacion-empresa) | Documentos fuente (base de conocimiento) que el agente utiliza para responder | PDF / Markdown / TXT |

> 💡 Los workflows de n8n son "código visual": cada nodo equivale a un bloque de lógica, y el archivo `.json` exportado contiene toda esa lógica en formato texto — por lo tanto, es código versionable y revisable, aunque no se escriba línea por línea como Python o JavaScript.

---

## 📚 Base de conocimiento

La documentación utilizada como base de conocimiento (RAG) corresponde a: `[PENDIENTE: describir los documentos — ej. "Manual de políticas internas y FAQ de producto de la empresa ficticia XYZ, en formato PDF/Markdown"]`.

Los documentos fuente se encuentran en [`/docs/documentacion-empresa`](./docs/documentacion-empresa).

---

## 🚀 Instrucciones de implementación desde cero

Esta sección permite que **cualquier persona, sin conocer el proyecto previamente**, pueda replicar la implementación completa usando n8n Cloud.

### Requisitos previos
- Cuenta de [n8n Cloud](https://n8n.io/cloud/) (free trial disponible)
- Cuenta de [Ollama](https://ollama.com/) para obtener una API Key de Ollama Cloud
- Cuenta de GitHub (para clonar/consultar este repositorio)

### Pasos

1. **Crear cuenta en n8n Cloud** en `n8n.io/cloud` y esperar a que se aprovisione la instancia (unos minutos).

2. **Obtener API Key de Ollama Cloud**:
   - Ingresar a `ollama.com` → crear cuenta.
   - Generar una API Key desde el panel de usuario.

3. **Configurar la credencial de Ollama en n8n**:
   - Dentro de n8n, ir a *Credentials → New → Ollama*.
   - Base URL: `https://ollama.com`
   - Pegar la API Key generada.

4. **Importar los workflows**:
   - *Workflows → Import from File*.
   - Importar `workflows/carga-documentos.json` (disponible en este repositorio).
   - Importar `workflows/agente-rag.json` (disponible en este repositorio).

5. **Cargar la documentación**:
   - Subir los archivos de `docs/documentacion-empresa` al workflow de carga (según el nodo de entrada configurado: manual upload o lectura desde una fuente conectada).
   - Ejecutar manualmente el workflow `carga-documentos.json` **una sola vez** para vectorizar los documentos en el Simple Vector Store.

6. **Configurar y publicar el agente**:
   - Abrir el workflow `agente-rag.json`.
   - Verificar que el nodo **AI Agent** tenga conectados: Ollama Chat Model, Simple Memory y Simple Vector Store (como Tool).
   - Abrir el nodo **Chat Trigger** → activar **"Make Chat Publicly Available"** → modo **"Hosted Chat"**.
   - Hacer clic en **Publish/Active** para dejarlo corriendo de forma permanente.
   - Copiar la **Chat URL** generada — esa es la URL pública del agente.

7. **Probar el agente**: abrir la Chat URL en el navegador y realizar preguntas relacionadas con la documentación cargada.

---

## 💬 Ejemplos de preguntas y respuestas

| Pregunta del usuario | Respuesta del agente |
|---|---|
| `[PENDIENTE]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |

---

## 📸 Evidencia del despliegue

`[PENDIENTE: capturas de pantalla del agente respondiendo en producción, del workflow en n8n, de la instancia activa]`

| Evidencia | Captura |
|---|---|
| Workflow del agente en n8n Cloud | `[PENDIENTE]` ![Workflow n8n](./screenshots/workflow-agente.png) |
| Chat público del agente respondiendo | `[PENDIENTE]` ![Agente funcionando](./screenshots/agente-funcionando.png) |
| Documentos cargados en el Vector Store | `[PENDIENTE]` ![Vector store](./screenshots/vector-store.png) |

---

## 📂 Estructura del repositorio

```
agente-ia-rag/
├── README.md                          # Este archivo
├── workflows/
│   ├── agente-rag.json                # Workflow principal del agente (export de n8n)
│   └── carga-documentos.json          # Workflow de carga/vectorización de documentos
├── docs/
│   └── documentacion-empresa/         # Documentos fuente usados como base de conocimiento
├── screenshots/                       # Evidencia visual del proyecto funcionando
└── docker/                            # ⚠️ Referencia histórica — ver nota abajo
```

> ⚠️ **Nota sobre la carpeta `/docker`**: contiene el `docker-compose.yml` y configuración del intento inicial de despliegue en Oracle Cloud Infrastructure (OCI), que fue descartado por indisponibilidad de capacidad de servidores ARM (ver sección [Decisiones de arquitectura](#-decisiones-de-arquitectura)). Se conserva en el repositorio **únicamente como evidencia del proceso de desarrollo**, no forma parte de la implementación final activa, que corre íntegramente en n8n Cloud.

---

## 🧭 Decisiones de arquitectura

Este proyecto originalmente se planificó para desplegarse en **Oracle Cloud Infrastructure (OCI)**, con n8n y Ollama autoalojados en una VM, y Oracle Autonomous Database como vector store. Durante la implementación se presentó indisponibilidad reiterada de capacidad ("Out of host capacity") para instancias Ampere A1 en la región Chile Central (Santiago), lo cual impidió aprovisionar la VM en un tiempo razonable.

Dado que el challenge indica explícitamente que **el uso de OCI no es obligatorio** y que la prioridad es que **el agente funcione correctamente**, se decidió migrar la implementación a **n8n Cloud**, simplificando además la arquitectura (Simple Memory y Simple Vector Store nativos de n8n en lugar de Redis y Oracle Database) para reducir puntos de falla y tiempo de configuración, sin afectar la funcionalidad ni la calidad de las respuestas del agente.

La configuración del intento original en OCI (`docker-compose.yml`, variables de entorno) se conserva en la carpeta [`/docker`](./docker) como evidencia documentada del proceso de desarrollo y de la decisión técnica tomada, no como parte de la implementación activa del proyecto.

---

## 📅 Historial de desarrollo

Este proyecto se desarrolló de forma incremental, documentado a través de los commits de este repositorio. Resumen de hitos:

- [x] Definición de arquitectura y stack tecnológico inicial (OCI)
- [x] Estructura inicial del repositorio en GitHub
- [x] Intento de despliegue en OCI (bloqueado por indisponibilidad de capacidad)
- [x] Pivote de arquitectura a n8n Cloud + Ollama Cloud
- [ ] Configuración de credencial Ollama Cloud en n8n
- [ ] Carga y vectorización de la documentación
- [ ] Construcción del workflow del AI Agent
- [ ] Publicación del Chat Trigger con URL pública
- [ ] Pruebas de preguntas/respuestas
- [ ] Documentación final del README

---

## 👤 Autor

**Jorge Gómez Pacheco** — Proyecto desarrollado como parte de un challenge de implementación de agentes de IA con RAG.
