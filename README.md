# Agente Inteligente de IA con RAG — BimBam Buy

> Agente de IA capaz de responder preguntas en lenguaje natural utilizando la documentación oficial de **BimBam Buy** como base de conocimiento, mediante la técnica de RAG (Retrieval-Augmented Generation).

![Estado](https://img.shields.io/badge/estado-en%20desarrollo-yellow)
![n8n](https://img.shields.io/badge/n8n-cloud-orange)
![Vector Store](https://img.shields.io/badge/vector%20store-Pinecone-blueviolet)
![Embeddings](https://img.shields.io/badge/embeddings-Cohere-purple)

---

## 📋 Tabla de contenidos

- [Descripción del proyecto](#-descripción-del-proyecto)
- [Demo / URL pública](#-demo--url-pública)
- [Arquitectura](#-arquitectura)
- [Tecnologías utilizadas](#-tecnologías-utilizadas)
- [Código fuente](#-código-fuente)
- [Base de conocimiento](#-base-de-conocimiento)
- [Instrucciones de implementación desde cero](#-instrucciones-de-implementación-desde-cero)
- [Mantenimiento y actualización de documentos](#-mantenimiento-y-actualización-de-documentos)
- [Ejemplos de preguntas y respuestas](#-ejemplos-de-preguntas-y-respuestas)
- [Evidencia del despliegue](#-evidencia-del-despliegue)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Decisiones de arquitectura](#-decisiones-de-arquitectura)
- [Historial de desarrollo](#-historial-de-desarrollo)

---

## 📖 Descripción del proyecto

**BimBam Buy** es un e-commerce multiplataforma (ficticio) enfocado en la experiencia de compra digital ágil y segura, con políticas robustas de reembolso, un programa de afiliados dinámico y una infraestructura logística optimizada para entregas rápidas y soporte constante al usuario final.

Este proyecto implementa un **agente conversacional de IA** que responde preguntas de los usuarios/clientes de BimBam Buy basándose exclusivamente en su documentación oficial (políticas, garantías, medios de pago, envíos, afiliados), utilizando **RAG (Retrieval-Augmented Generation)**: el agente primero busca los fragmentos relevantes de la documentación real y luego genera una respuesta fundamentada en esa información, evitando alucinaciones o respuestas inventadas.

**Problema que resuelve:** reduce el tiempo de respuesta a consultas frecuentes de soporte al cliente (reembolsos, envíos, garantías, medios de pago, afiliados), permitiendo respuestas inmediatas y consistentes basadas en la documentación oficial vigente, sin depender de un agente humano disponible en todo momento.

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
        │                        └── Pinecone (Tool)    │
        │                            (vector store)     │
        └──────────────────────────────────────────┘
                                          │
                                          ▼
                          ┌───────────────────────────┐
                          │  Workflow de carga (aparte)  │
                          │  HTTP Request → Data Loader  │
                          │  → Text Splitter →           │
                          │  Embeddings Cohere →          │
                          │  Pinecone (Insert)            │
                          └───────────────────────────┘
```

**Flujo de consulta (usuario final):**
1. El usuario abre la URL pública del chat.
2. Escribe su pregunta en lenguaje natural (ej. "¿Cuánto tarda un reembolso?").
3. El **AI Agent** recibe el mensaje junto con el historial de la conversación (Memory).
4. El agente consulta **Pinecone** buscando los fragmentos de documentación más relevantes.
5. Esos fragmentos + la pregunta se envían a **Ollama Cloud**, que genera la respuesta final.
6. La respuesta se muestra al usuario en la misma ventana de chat.

**Flujo de carga de documentos (administrador, proceso aparte, no público):**
1. Se ejecuta manualmente (Manual Trigger, solo accesible desde la cuenta de n8n) un workflow que recorre la lista de PDFs de BimBam Buy.
2. Cada PDF se descarga, se le extrae el texto (Data Loader), se fragmenta (Text Splitter), se vectoriza (Embeddings Cohere) y se inserta en Pinecone con su metadata (`doc_id`, `categoria`, `source`).

---

## 🛠 Tecnologías utilizadas

| Tecnología | Rol en el proyecto | Costo |
|---|---|---|
| **n8n Cloud** | Orquestador del agente: workflows visuales, Chat Trigger (interfaz + URL pública), AI Agent | Free trial / plan básico |
| **Ollama Cloud** | Modelo de lenguaje (LLM) vía API, genera las respuestas finales del agente | Gratis (con límites de uso) |
| **Cohere** (`embed-multilingual-v3.0`) | Genera los embeddings (vectores) tanto de los documentos como de las preguntas del usuario | Gratis (con límites de uso) |
| **Pinecone** | Vector Store — almacena y busca los fragmentos de documentación por similitud semántica, con soporte de metadata y borrado selectivo | Gratis (plan Starter) |
| **Simple Memory** (nodo nativo de n8n) | Mantiene el historial de conversación durante la sesión del usuario | Gratis, sin configuración adicional |

---

## 💻 Código fuente

| Archivo | Qué contiene | Formato |
|---|---|---|
| [`workflows/agente-rag.json`](./workflows/agente-rag.json) | Workflow principal: Chat Trigger + AI Agent + conexiones a Ollama Cloud, Memory y Pinecone | JSON (exportado de n8n) |
| [`workflows/carga-documentos.json`](./workflows/carga-documentos.json) | Workflow de carga inicial: recorre los 5 PDFs de BimBam Buy, los vectoriza e inserta en Pinecone con metadata | JSON (exportado de n8n) |
| [`workflows/actualizar-documento.json`](./workflows/actualizar-documento.json) | Workflow de mantenimiento: borra los vectores de un `doc_id` específico e inserta la versión actualizada del documento | JSON (exportado de n8n) |
| [`docs/documentacion-empresa/`](./docs/documentacion-empresa) | Los 5 PDFs oficiales de BimBam Buy usados como base de conocimiento | PDF |

> 💡 Los workflows de n8n son "código visual": cada nodo equivale a un bloque de lógica, y el archivo `.json` exportado contiene toda esa lógica en formato texto — por lo tanto, es código versionable y revisable, aunque no se escriba línea por línea como Python o JavaScript.

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

Los documentos fuente se encuentran en [`/docs/documentacion-empresa`](./docs/documentacion-empresa).

---

## 🚀 Instrucciones de implementación desde cero

### Requisitos previos
- Cuenta de [n8n Cloud](https://n8n.io/cloud/)
- Cuenta de [Ollama](https://ollama.com/) (API Key para Ollama Cloud)
- Cuenta de [Cohere](https://cohere.com/) (API Key)
- Cuenta de [Pinecone](https://www.pinecone.io/) (API Key)
- Cuenta de GitHub (para acceder a los documentos fuente vía URL raw)

### Pasos

1. **Crear el índice en Pinecone**: nombre `agente-ia-rag`, dimensión `1024` (compatible con `embed-multilingual-v3.0` de Cohere), métrica `cosine`.

2. **Crear cuenta en n8n Cloud** y configurar las credenciales:
   - *Ollama*: Base URL `https://ollama.com` + API Key.
   - *Cohere*: API Key.
   - *Pinecone*: API Key.

3. **Importar los workflows** (`Workflows → Import from File`):
   - `workflows/carga-documentos.json`
   - `workflows/actualizar-documento.json`
   - `workflows/agente-rag.json`

4. **Ejecutar la carga inicial**: abrir `carga-documentos.json` y ejecutar manualmente (Manual Trigger) — procesa los 5 PDFs de BimBam Buy y los inserta en Pinecone con su metadata correspondiente.

5. **Publicar el agente**: abrir `agente-rag.json` → nodo Chat Trigger → activar **"Make Chat Publicly Available"** → modo **"Hosted Chat"** → **Publish/Active** → copiar la Chat URL generada.

6. **Probar el agente**: abrir la Chat URL y realizar preguntas relacionadas con reembolsos, envíos, pagos, garantías o el programa de afiliados.

---

## 🔄 Mantenimiento y actualización de documentos

Cuando un documento de BimBam Buy se actualiza (ej. cambia la política de reembolsos), el proceso es:

1. Reemplazar el PDF correspondiente en `/docs/documentacion-empresa` y subir el cambio a GitHub.
2. Ejecutar manualmente el workflow `workflows/actualizar-documento.json`, indicando el `doc_id` del documento a actualizar.
3. El workflow primero **borra** todos los vectores asociados a ese `doc_id` en Pinecone (`Delete` filtrado por metadata), y luego **inserta** los vectores del documento actualizado — garantizando que no queden fragmentos obsoletos mezclados con los nuevos.

Este mecanismo de borrado + reinserción por `doc_id` es posible gracias a que Pinecone soporta filtrado y borrado nativo por metadata, a diferencia de alternativas más básicas (como el Simple Vector Store nativo de n8n) que no ofrecen esta capacidad de forma directa.

---

## 💬 Ejemplos de preguntas y respuestas

| Pregunta del usuario | Respuesta del agente |
|---|---|
| `[PENDIENTE: probar tras publicar el agente]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |

---

## 📸 Evidencia del despliegue

| Evidencia | Captura |
|---|---|
| Índice de Pinecone con vectores cargados | `[PENDIENTE]` ![Pinecone](./screenshots/pinecone-index.png) |
| Workflow de carga de documentos en n8n | `[PENDIENTE]` ![Workflow carga](./screenshots/workflow-carga.png) |
| Workflow del agente en n8n Cloud | `[PENDIENTE]` ![Workflow agente](./screenshots/workflow-agente.png) |
| Chat público del agente respondiendo | `[PENDIENTE]` ![Agente funcionando](./screenshots/agente-funcionando.png) |

---

## 📂 Estructura del repositorio

```
agente-ia-rag/
├── README.md                          # Este archivo
├── workflows/
│   ├── agente-rag.json                # Workflow principal del agente
│   ├── carga-documentos.json          # Workflow de carga inicial (5 PDFs)
│   └── actualizar-documento.json      # Workflow de actualización (borrar + recargar)
├── docs/
│   └── documentacion-empresa/         # 5 PDFs oficiales de BimBam Buy
├── screenshots/                       # Evidencia visual del proyecto funcionando
└── docker/                            # ⚠️ Referencia histórica — ver nota abajo
```

> ⚠️ **Nota sobre la carpeta `/docker`**: contiene el `docker-compose.yml` del intento inicial de despliegue en Oracle Cloud Infrastructure (OCI), descartado por indisponibilidad de capacidad de servidores ARM (ver [Decisiones de arquitectura](#-decisiones-de-arquitectura)). Se conserva únicamente como evidencia del proceso de desarrollo.

---

## 🧭 Decisiones de arquitectura

**OCI → n8n Cloud:** el proyecto se planificó originalmente para OCI (VM con n8n y Ollama autoalojados). Se presentó indisponibilidad reiterada de capacidad ("Out of host capacity") para instancias Ampere A1 en la región Chile Central (Santiago). Dado que el challenge no exige OCI y prioriza que el agente funcione correctamente, se migró a n8n Cloud. La configuración original se conserva en `/docker` como evidencia del proceso.

**Simple Vector Store → Pinecone:** se requería la capacidad de actualizar documentos de forma limpia (borrar los fragmentos obsoletos de un documento específico e insertar la versión nueva, sin afectar el resto de la base de conocimiento). El Simple Vector Store nativo de n8n no ofrece borrado selectivo por metadata de forma directa, por lo que se optó por Pinecone, que soporta esta operación de forma nativa y cuenta con una capa gratuita suficiente para el volumen de este proyecto (5 documentos, ~500-900 fragmentos vectorizados estimados).

---

## 📅 Historial de desarrollo

- [x] Definición de arquitectura y stack tecnológico inicial (OCI)
- [x] Estructura inicial del repositorio en GitHub
- [x] Intento de despliegue en OCI (bloqueado por indisponibilidad de capacidad)
- [x] Pivote de arquitectura a n8n Cloud + Ollama Cloud
- [x] Definición de la empresa ficticia (BimBam Buy) y sus 5 documentos base
- [x] Selección de Pinecone + Cohere como stack de vectorización (soporte de actualización por metadata)
- [ ] Creación del índice en Pinecone
- [ ] Configuración de credenciales (Ollama, Cohere, Pinecone) en n8n
- [ ] Carga y vectorización de los 5 documentos de BimBam Buy
- [ ] Construcción del workflow del AI Agent
- [ ] Publicación del Chat Trigger con URL pública
- [ ] Pruebas de preguntas/respuestas
- [ ] Documentación final del README

---

## 👤 Autor

**Jorge Gómez Pacheco** — Proyecto desarrollado como parte de un challenge de implementación de agentes de IA con RAG.
