# Agente Inteligente de IA con RAG — Consultor de Documentación

> Agente de IA capaz de responder preguntas en lenguaje natural utilizando una base de conocimiento documental (RAG), simulando una aplicación en entorno real de producción para una empresa.

![Estado](https://img.shields.io/badge/estado-en%20desarrollo-yellow)
![n8n](https://img.shields.io/badge/n8n-workflow-orange)
![Ollama](https://img.shields.io/badge/LLM-Ollama-black)
![OCI](https://img.shields.io/badge/deploy-Oracle%20Cloud-red)

---

## 📋 Tabla de contenidos

- [Descripción del proyecto](#-descripción-del-proyecto)
- [Demo / URL pública](#-demo--url-pública)
- [Arquitectura](#-arquitectura)
- [Tecnologías utilizadas](#-tecnologías-utilizadas)
- [Base de conocimiento](#-base-de-conocimiento)
- [Instalación y despliegue](#-instalación-y-despliegue)
- [Ejemplos de preguntas y respuestas](#-ejemplos-de-preguntas-y-respuestas)
- [Evidencia del despliegue](#-evidencia-del-despliegue)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Historial de desarrollo](#-historial-de-desarrollo)

---

## 📖 Descripción del proyecto

Este proyecto implementa un **agente conversacional de IA** que responde preguntas de usuarios basándose exclusivamente en la documentación interna de una empresa (ficticia), utilizando la técnica de **RAG (Retrieval-Augmented Generation)**: en lugar de que el modelo "invente" respuestas, primero busca los fragmentos relevantes de la documentación y luego genera una respuesta fundamentada en esa información.

**Empresa ficticia utilizada:** `[PENDIENTE: nombre y breve descripción del rubro]`

**Problema que resuelve:** `[PENDIENTE: ej. "Reducir el tiempo de respuesta a consultas frecuentes de soporte, permitiendo que los usuarios obtengan respuestas inmediatas basadas en el manual/documentación oficial, sin esperar a un agente humano."]`

---

## 🌐 Demo / URL pública

- **Chat del agente (n8n en OCI):** `[PENDIENTE: URL pública del Chat Trigger]`
- **Repositorio GitHub:** `[PENDIENTE: URL de este repo]`

---

## 🏗 Arquitectura

```
┌──────────────────────────────────────────────────────┐
│                VM Oracle Cloud (Always Free)            │
│                                                          │
│   Usuario ──► Chat Trigger (n8n) ──► AI Agent           │
│                                        │                 │
│                          ┌─────────────┼──────────────┐ │
│                          ▼             ▼              ▼ │
│                    Ollama Chat   Simple Memory    Oracle │
│                      Model      (historial de    Vector │
│                    (LLM local)   conversación)    Store │
│                                                    (Tool)│
└──────────────────────────────────────────────────────┘
                                          │
                                          ▼
                          ┌───────────────────────────┐
                          │  Oracle Autonomous DB 23ai │
                          │  (documentos vectorizados) │
                          └───────────────────────────┘
```

**Flujo de datos:**
1. El usuario escribe una pregunta en la interfaz de chat pública (Chat Trigger de n8n).
2. El **AI Agent** recibe el mensaje y el historial de la sesión (Memory).
3. El agente consulta el **Vector Store** (documentos de la empresa embebidos como vectores) buscando los fragmentos más relevantes para la pregunta.
4. Esos fragmentos + la pregunta se envían al modelo **Ollama** (LLM local), que genera la respuesta final en lenguaje natural.
5. La respuesta se devuelve al usuario dentro del mismo hilo de chat.

---

## 🛠 Tecnologías utilizadas

| Tecnología | Rol en el proyecto |
|---|---|
| **n8n** (self-hosted) | Orquestador del agente: workflows visuales, Chat Trigger, AI Agent |
| **Ollama** | Modelo de lenguaje (LLM) local, gratuito, sin límites de uso, corre en la misma VM |
| **Oracle Cloud Infrastructure (OCI)** | Infraestructura del proyecto — VM Always Free (cómputo) |
| **Oracle Autonomous Database 23ai** | Base de datos vectorial — almacena los embeddings de la documentación |
| **Docker / Docker Compose** | Contenerización de n8n y Ollama en la VM |
| `[PENDIENTE: Redis, si se implementa]` | Memoria persistente de conversación con expiración por inactividad |

---

## 📚 Base de conocimiento

La documentación utilizada como base de conocimiento (RAG) corresponde a: `[PENDIENTE: describir los documentos — ej. "Manual de políticas internas y FAQ de producto de la empresa ficticia XYZ, en formato PDF/Markdown"]`.

Los documentos fuente se encuentran en [`/docs/documentacion-empresa`](./docs/documentacion-empresa).

---

## ⚙️ Instalación y despliegue

### Requisitos previos
- Cuenta Oracle Cloud (Always Free)
- Docker y Docker Compose instalados en la VM
- Dominio propio (o DDNS gratuito) para HTTPS

### Pasos

```bash
# 1. Clonar este repositorio en la VM
git clone [PENDIENTE: URL del repo]
cd agente-ia-rag

# 2. Levantar los servicios con Docker Compose
docker compose -f docker/docker-compose.yml up -d

# 3. Descargar el modelo de Ollama a utilizar
docker exec -it ollama ollama pull llama3.2:3b

# 4. Acceder a n8n
# https://tu-dominio-o-ip:5678

# 5. Importar el workflow del agente
# Settings → Import from File → workflows/agente-rag.json

# 6. Configurar credenciales en n8n
#    - Ollama (Base URL: http://ollama:11434)
#    - Oracle Database (usar wallet de conexión, ver /docker/oracle-wallet)

# 7. Ejecutar una vez el workflow de carga de documentos
#    (vectoriza los archivos de /docs/documentacion-empresa)

# 8. Activar (Publish) el workflow del agente
#    Copiar la Chat URL pública generada
```

`[PENDIENTE: completar/ajustar estos pasos según la configuración final real]`

---

## 💬 Ejemplos de preguntas y respuestas

| Pregunta del usuario | Respuesta del agente |
|---|---|
| `[PENDIENTE]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |

---

## 📸 Evidencia del despliegue

`[PENDIENTE: capturas de pantalla del agente respondiendo en producción, de la VM corriendo en OCI, del workflow en n8n, etc. — se agregarán en /screenshots]`

![Agente funcionando](./screenshots/agente-funcionando.png)
![Deploy en OCI](./screenshots/deploy-oci.png)

---

## 📂 Estructura del repositorio

```
agente-ia-rag/
├── README.md                          # Este archivo
├── docker/
│   └── docker-compose.yml             # Definición de servicios (n8n, Ollama)
├── workflows/
│   ├── agente-rag.json                # Workflow principal del agente (export de n8n)
│   └── carga-documentos.json          # Workflow de carga/vectorización de documentos
├── docs/
│   └── documentacion-empresa/         # Documentos fuente usados como base de conocimiento
└── screenshots/                       # Evidencia visual del proyecto funcionando
```

---

## 📅 Historial de desarrollo

Este proyecto se desarrolló de forma incremental, documentado a través de los commits de este repositorio. Resumen de hitos:

- [ ] Definición de arquitectura y stack tecnológico
- [ ] Aprovisionamiento de VM en OCI (Always Free)
- [ ] Despliegue de n8n + Ollama vía Docker Compose
- [ ] Creación de Oracle Autonomous Database (vector store)
- [ ] Carga y vectorización de la documentación
- [ ] Construcción del workflow del AI Agent
- [ ] Configuración de memoria de conversación
- [ ] Publicación del Chat Trigger con URL pública
- [ ] Pruebas de preguntas/respuestas
- [ ] Documentación final del README

---

## 👤 Autor

**Jorge Gómez** — Proyecto desarrollado como parte de un challenge de implementación de agentes de IA con RAG.
