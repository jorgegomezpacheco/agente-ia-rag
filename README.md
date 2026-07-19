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
- [Código fuente](#-código-fuente)
- [Base de conocimiento](#-base-de-conocimiento)
- [Instrucciones de implementación desde cero](#-instrucciones-de-implementación-desde-cero)
- [Ejecución local / mantenimiento](#-ejecución-local--mantenimiento)
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
- **Repositorio GitHub:** https://github.com/jorgegomezpacheco/agente-ia-rag

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

| Tecnología | Rol en el proyecto | Costo |
|---|---|---|
| **n8n** (self-hosted) | Orquestador del agente: workflows visuales, Chat Trigger, AI Agent | Gratis (self-hosted) |
| **Ollama** | Modelo de lenguaje (LLM) local, corre en la misma VM, sin límites de uso ni créditos | Gratis |
| **Oracle Cloud Infrastructure (OCI)** | Infraestructura del proyecto — VM Always Free (cómputo) | Gratis (Always Free) |
| **Oracle Autonomous Database 23ai** | Base de datos vectorial — almacena los embeddings de la documentación | Gratis (Always Free) |
| **Docker / Docker Compose** | Contenerización de n8n y Ollama en la VM | Gratis |
| `[PENDIENTE: Redis, si se implementa]` | Memoria persistente de conversación con expiración por inactividad | Gratis |

---

## 💻 Código fuente

Todo el código y la configuración del proyecto están versionados en este repositorio. No es una aplicación tradicional con líneas de código de programación general, sino una combinación de **infraestructura como código** (Docker) y **workflows visuales exportables** (n8n), de la siguiente manera:

| Archivo | Qué contiene | Lenguaje/Formato |
|---|---|---|
| [`docker/docker-compose.yml`](./docker/docker-compose.yml) | Definición de los servicios que corren en la VM (n8n y Ollama), puertos, volúmenes y variables de entorno | YAML |
| [`docker/.env.example`](./docker/.env.example) | Plantilla de variables de entorno necesarias (dominio, credenciales) | ENV |
| [`workflows/agente-rag.json`](./workflows/agente-rag.json) | Workflow principal: Chat Trigger + AI Agent + conexiones a Ollama, Memory y Vector Store | JSON (exportado de n8n) |
| [`workflows/carga-documentos.json`](./workflows/carga-documentos.json) | Workflow auxiliar que lee los documentos de `/docs`, los divide en fragmentos, genera embeddings y los guarda en el Vector Store | JSON (exportado de n8n) |
| [`docs/documentacion-empresa/`](./docs/documentacion-empresa) | Documentos fuente (base de conocimiento) que el agente utiliza para responder | PDF / Markdown / TXT |

> 💡 Los workflows de n8n son "código visual": cada nodo equivale a un bloque de lógica. El archivo `.json` exportado contiene toda esa lógica en formato texto, por lo que **sí es código versionable y revisable**, aunque no se escriba línea por línea como Python o JavaScript.

---

## 📚 Base de conocimiento

La documentación utilizada como base de conocimiento (RAG) corresponde a: `[PENDIENTE: describir los documentos — ej. "Manual de políticas internas y FAQ de producto de la empresa ficticia XYZ, en formato PDF/Markdown"]`.

Los documentos fuente se encuentran en [`/docs/documentacion-empresa`](./docs/documentacion-empresa).

---

## 🚀 Instrucciones de implementación desde cero

Esta sección permite que **cualquier persona, sin conocer el proyecto previamente**, pueda replicar la implementación completa: desde crear la infraestructura en la nube hasta tener el agente respondiendo por una URL pública.

### Requisitos previos

- Cuenta en [Oracle Cloud Infrastructure](https://www.oracle.com/cloud/free/) (nivel Always Free, sin costo)
- Un dominio propio o un servicio DNS dinámico gratuito (ej. [DuckDNS](https://www.duckdns.org/)) para tener HTTPS
- Cuenta de GitHub (para clonar este repositorio)
- Conocimientos básicos de terminal/línea de comandos

### Paso 1 — Crear la VM en Oracle Cloud

1. Ingresa a la [consola de OCI](https://cloud.oracle.com/) → **Compute → Instances → Create Instance**.
2. Selecciona la imagen **Ubuntu** (versión LTS más reciente disponible).
3. En **Shape**, elige **VM.Standard.A1.Flex** (Ampere ARM, incluida en Always Free) — asigna 2 OCPU / 12 GB RAM.
4. En **Networking**, asegúrate de que se asigne una **IP pública**.
5. Genera o sube un par de llaves SSH (guarda la llave privada, la necesitarás para conectarte).
6. Crea la instancia y espera a que su estado sea "Running".

### Paso 2 — Abrir los puertos necesarios (Security List)

En la consola de OCI, ve a la **VCN** de tu instancia → **Security Lists** → agrega reglas de ingreso (Ingress Rules) para permitir tráfico en los puertos:
- `22` (SSH)
- `80` y `443` (HTTP/HTTPS)
- `5678` (n8n)

### Paso 3 — Conectarte a la VM por SSH

```bash
ssh -i /ruta/a/tu-llave-privada.key ubuntu@IP_PUBLICA_DE_TU_VM
```

### Paso 4 — Instalar Docker y Docker Compose

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```
(Cierra sesión SSH y vuelve a entrar para que el cambio de grupo tenga efecto.)

### Paso 5 — Clonar este repositorio en la VM

```bash
git clone https://github.com/jorgegomezpacheco/agente-ia-rag.git
cd agente-ia-rag/docker
```

### Paso 6 — Configurar las variables de entorno

```bash
cp .env.example .env
nano .env
```
Completa `N8N_HOST` con tu dominio o IP pública.

### Paso 7 — Levantar los servicios

```bash
docker compose up -d
```

Verifica que ambos contenedores estén corriendo:
```bash
docker ps
```

### Paso 8 — Descargar el modelo de IA en Ollama

```bash
docker exec -it ollama ollama pull llama3.2:3b
```

### Paso 9 — Configurar HTTPS (obligatorio para el chat público)

`[PENDIENTE: detallar configuración de Caddy/Traefik/Nginx + certificado SSL una vez implementado]`

### Paso 10 — Acceder a n8n y configurar el agente

1. Abre `https://TU_DOMINIO_O_IP:5678` en el navegador.
2. Crea tu cuenta de administrador de n8n (primera vez).
3. Ve a **Workflows → Import from File** e importa `workflows/carga-documentos.json` y `workflows/agente-rag.json` (disponibles en este repositorio).
4. Configura las credenciales necesarias:
   - **Ollama**: Base URL `http://ollama:11434`
   - **Oracle Database** (Vector Store): usar los datos de conexión del Autonomous Database — `[PENDIENTE: detallar pasos exactos]`
5. Ejecuta manualmente el workflow `carga-documentos.json` **una sola vez** para vectorizar los archivos de `/docs/documentacion-empresa`.
6. Abre el workflow `agente-rag.json`, activa la opción **"Make Chat Publicly Available"** en el nodo Chat Trigger.
7. Haz clic en **Publish/Active** para dejar el workflow corriendo de forma permanente.
8. Copia la **Chat URL** generada — esa es la URL pública del agente.

---

## 🔧 Ejecución local / mantenimiento

Comandos útiles para administrar el proyecto una vez desplegado:

```bash
# Ver logs de n8n
docker logs -f n8n

# Ver logs de Ollama
docker logs -f ollama

# Reiniciar los servicios
docker compose restart

# Detener todo
docker compose down

# Actualizar a la última versión de las imágenes
docker compose pull
docker compose up -d
```

---

## 💬 Ejemplos de preguntas y respuestas

| Pregunta del usuario | Respuesta del agente |
|---|---|
| `[PENDIENTE]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |
| `[PENDIENTE]` | `[PENDIENTE]` |

---

## 📸 Evidencia del despliegue

`[PENDIENTE: capturas de pantalla del agente respondiendo en producción, de la VM corriendo en OCI, del workflow en n8n, etc.]`

| Evidencia | Captura |
|---|---|
| VM corriendo en OCI | `[PENDIENTE]` ![VM en OCI](./screenshots/vm-oci-creada.png) |
| Contenedores activos (`docker ps`) | `[PENDIENTE]` ![Contenedores activos](./screenshots/docker-ps.png) |
| Workflow del agente en n8n | `[PENDIENTE]` ![Workflow n8n](./screenshots/workflow-agente.png) |
| Chat del agente respondiendo | `[PENDIENTE]` ![Agente funcionando](./screenshots/agente-funcionando.png) |

---

## 📂 Estructura del repositorio

```
agente-ia-rag/
├── README.md                          # Este archivo
├── docker/
│   ├── docker-compose.yml             # Definición de servicios (n8n, Ollama)
│   └── .env.example                   # Plantilla de variables de entorno
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

- [x] Definición de arquitectura y stack tecnológico
- [x] Estructura inicial del repositorio en GitHub
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

**Jorge Gómez Pacheco** — Proyecto desarrollado como parte de un challenge de implementación de agentes de IA con RAG.
