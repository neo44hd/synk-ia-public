<p align="center">
  <img src="https://img.shields.io/badge/Node.js-v20+-339933?logo=node.js&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=white" alt="React" />
  <img src="https://img.shields.io/badge/Ollama-local_LLM-000?logo=ollama&logoColor=white" alt="Ollama" />
  <img src="https://img.shields.io/badge/Docker-compose-2496ED?logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/License-MIT-blue" alt="License" />
</p>

# SYNK-IA

**Plataforma de IA local para gestión inteligente de PYMES.**

Chat multi-modelo, agentes autónomos, procesamiento de documentos, automatizaciones y dashboards ejecutivos — todo corriendo en tu propia máquina sin dependencia de APIs de pago.

---

## Tabla de contenidos

- [Arquitectura](#arquitectura)
- [Stack tecnológico](#stack-tecnológico)
- [Requisitos](#requisitos)
- [Instalación rápida](#instalación-rápida)
- [Variables de entorno](#variables-de-entorno)
- [Docker — servicios auxiliares](#docker--servicios-auxiliares)
- [Estructura del proyecto](#estructura-del-proyecto)
- [API — endpoints principales](#api--endpoints-principales)
- [Paneles de chat](#paneles-de-chat)
- [Acceso remoto con Cloudflare Tunnel](#acceso-remoto-con-cloudflare-tunnel)
- [Gestión de procesos con PM2](#gestión-de-procesos-con-pm2)
- [Problemas conocidos y fixes](#problemas-conocidos-y-fixes)
- [Troubleshooting](#troubleshooting)
- [Contribuir](#contribuir)
- [Licencia](#licencia)

---

## Arquitectura

```
                    ┌─────────────────────────────────────────┐
                    │           Cloudflare Tunnel              │
                    │   tudominio.com → localhost:3001         │
                    │   chat.tudominio.com → localhost:3030    │
                    └──────────────┬──────────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────────┐
                    │          PM2 (Process Manager)           │
                    │                                          │
                    │  sinkia-api (:3001)    Backend Node.js   │
                    │  ollama-proxy (:11435) Proxy Ollama      │
                    │  cloudflared          Túnel Cloudflare   │
                    └──────────────┬──────────────────────────┘
                                   │
          ┌────────────────────────┼─────────────────────────┐
          │                        │                         │
   ┌──────▼──────┐   ┌────────────▼──────────┐   ┌──────────▼──────┐
   │   Frontend   │   │    Backend (Express)   │   │  Docker Stack   │
   │  React + Vite│   │    API REST + WS       │   │                 │
   │  Tailwind    │   │    21 rutas            │   │  SearXNG :8888  │
   │  Radix UI    │   │    3 servicios         │   │  Qdrant  :6333  │
   │  Recharts    │   │    2 agentes           │   │  OpenWebUI:3030 │
   │  80+ páginas │   │                        │   │  n8n     :5678  │
   └──────────────┘   └────────────┬───────────┘   └─────────────────┘
                                   │
                      ┌────────────▼───────────┐
                      │   Ollama (:11434)       │
                      │   Modelos locales       │
                      │   qwen3.5, codegemma,   │
                      │   qwen2.5-coder, etc.   │
                      └─────────────────────────┘
```

---

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Frontend | React 18, Vite, Tailwind CSS, Radix UI, Recharts, Framer Motion |
| Backend | Node.js, Express, WebSockets (ws) |
| IA local | Ollama (qwen3.5, codegemma:7b, qwen2.5-coder:14b, functiongemma) |
| Vector DB | Qdrant |
| Búsqueda web | SearXNG (self-hosted) |
| Interfaz IA alt. | Open WebUI |
| Automatización | n8n |
| Procesos | PM2 |
| Túnel | Cloudflare Tunnel (cloudflared) |
| Contenedores | Docker Compose (OrbStack recomendado en macOS) |

---

## Requisitos

| Requisito | Mínimo | Recomendado |
|-----------|--------|-------------|
| RAM | 16 GB | 24 GB+ |
| CPU | 4 cores | Apple Silicon M-series / 8+ cores |
| Disco | 20 GB libres | 50 GB+ (modelos IA) |
| Node.js | v20 | v22+ |
| Ollama | Última versión | Última versión |
| Docker | Docker Desktop / OrbStack | OrbStack (macOS) |
| SO | macOS / Linux | macOS (Apple Silicon) |

---

## Instalación rápida

### 1. Clonar e instalar dependencias

```bash
git clone https://github.com/neo44hd/synk-ia-public.git
cd synk-ia-public
npm install
cd server && npm install && cd ..
```

### 2. Instalar Ollama y descargar modelos

```bash
# Instalar Ollama (macOS)
brew install ollama
ollama serve &

# Descargar modelos necesarios
ollama pull qwen3.5          # Chat general
ollama pull codegemma:7b     # Clasificación de documentos
ollama pull qwen2.5-coder:14b  # Programación (Aider + Claude Code)
```

### 3. Configurar variables de entorno

```bash
cp server/.env.example server/.env
# Edita server/.env con tus valores reales
```

### 4. Levantar servicios Docker

```bash
cd docker
docker compose up -d
cd ..
```

### 5. Arrancar el servidor

```bash
# Desarrollo
node server/index.js

# Producción con PM2
pm2 start server/index.js --name sinkia-api
pm2 save
```

### 6. Arrancar el frontend (desarrollo)

```bash
npm run dev
# Abre http://localhost:5173
```

### 7. Build para producción

```bash
npm run build
# Los archivos estáticos quedan en dist/
# El servidor Express ya sirve dist/ automáticamente
```

---

## Variables de entorno

Copia `server/.env.example` a `server/.env` y configura:

### Servidor

| Variable | Descripción | Valor por defecto |
|----------|-------------|-------------------|
| `PORT` | Puerto del backend | `3001` |
| `NODE_ENV` | Entorno | `production` |
| `CORS_ORIGINS` | Orígenes CORS permitidos (separados por coma) | `http://localhost:5173` |
| `ADMIN_TOKEN` | Token de autenticación admin | **obligatorio** |
| `JWT_SECRET` | Secreto para firmar tokens JWT | Auto-generado si no se define |

### Ollama (IA local)

| Variable | Descripción | Valor por defecto |
|----------|-------------|-------------------|
| `OLLAMA_URL` | URL de Ollama | `http://localhost:11434` |
| `OLLAMA_MODEL` | Modelo principal | `qwen3.5:latest` |
| `OLLAMA_CHAT_MODEL` | Modelo para chat | Usa `OLLAMA_MODEL` |
| `OLLAMA_CLASSIFY_MODEL` | Modelo para clasificación | `codegemma:7b` |

### Claude Code (proxy local)

| Variable | Descripción | Valor por defecto |
|----------|-------------|-------------------|
| `LOCAL_LLM_URL` | URL del LLM local | `http://localhost:11434` |
| `LOCAL_LLM_MODEL` | Modelo para Claude Code | `qwen2.5-coder:14b` |

### Aider

| Variable | Descripción | Valor por defecto |
|----------|-------------|-------------------|
| `AIDER_MODEL` | Modelo Aider | `ollama/qwen2.5-coder:14b` |
| `AIDER_PROJECT_DIR` | Directorio del proyecto | `/path/to/your/project` |

### Integraciones (opcionales)

| Variable | Descripción |
|----------|-------------|
| `EMAIL_USER` | Cuenta IMAP (Gmail) |
| `EMAIL_APP_PASSWORD` | App Password de Gmail |
| `REVO_TOKEN_LARGO` | Token largo de Revo XEF (POS) |
| `REVO_TOKEN_CORTO` | Token corto de Revo XEF |
| `BILOOP_CIF` | CIF para integración Biloop |
| `ASSEMPSA_BILOOP_API_KEY` | API key de Biloop |
| `SEARXNG_URL` | URL de SearXNG local |

---

## Docker — servicios auxiliares

El archivo `docker/docker-compose.yml` levanta 4 servicios:

| Servicio | Puerto | Descripción |
|----------|--------|-------------|
| **SearXNG** | 8888 | Motor de búsqueda web privado |
| **Qdrant** | 6333 | Base de datos vectorial (RAG / memoria semántica) |
| **Open WebUI** | 3030 | Interfaz alternativa de chat con IA |
| **n8n** | 5678 | Automatizaciones y workflows |

```bash
cd docker
docker compose up -d        # Arrancar todo
docker compose logs -f       # Ver logs
docker compose down          # Parar todo
```

> **Importante:** Edita `docker-compose.yml` y cambia los valores `CHANGE_ME_*` por tus propias claves secretas antes de arrancar.

---

## Estructura del proyecto

```
synk-ia/
├── server/                      # Backend Node.js + Express
│   ├── index.js                 # Punto de entrada
│   ├── .env.example             # Plantilla de configuración
│   ├── middleware/
│   │   └── auth.js              # JWT + admin token
│   ├── routes/
│   │   ├── chat.js              # Chat IA multi-modelo
│   │   ├── claude-proxy.js      # Proxy Anthropic → Ollama
│   │   ├── aider.js             # Integración Aider
│   │   ├── aiden.js             # Agente OpenClaw
│   │   ├── shell-terminal.js    # Terminal WebSocket (node-pty)
│   │   ├── documents.js         # Gestión documental
│   │   ├── trabajadores.js      # CRUD trabajadores
│   │   ├── email.js             # IMAP / envío email
│   │   ├── revo.js              # POS Revo XEF
│   │   ├── biloop.js            # Integración Biloop
│   │   ├── filebrain.js         # RAG sobre documentos
│   │   ├── health.js            # Health checks
│   │   └── ...                  # +8 rutas más
│   ├── services/
│   │   ├── brain.js             # Motor IA (SynkiaBrain)
│   │   ├── documentProcessor.js # OCR + clasificación
│   │   └── fileContext.js       # Contexto de archivos compartido
│   └── agents/
│       ├── emailAgent.js        # Agente autónomo de email
│       └── revoAgent.js         # Agente autónomo de POS
│
├── src/                         # Frontend React
│   ├── pages/                   # 80+ páginas (dashboard, staff, billing...)
│   ├── components/              # UI components (Radix, custom)
│   ├── services/                # Clientes API
│   └── data/                    # Datos estáticos (productos, mappings)
│
├── public/                      # HTML estático (paneles de chat)
│   ├── chat.html                # Chat multi-tab principal
│   ├── admin.html               # Panel de administración
│   ├── documents.html           # Gestor documental
│   ├── terminal.html            # Terminal web
│   └── trabajadores.html        # Portal empleados
│
├── docker/
│   ├── docker-compose.yml       # SearXNG + Qdrant + OpenWebUI + n8n
│   ├── config/searxng/          # Config SearXNG
│   └── setup.sh                 # Script de setup inicial
│
├── scripts/
│   ├── startup.sh               # Arranque completo
│   ├── claude-code.sh           # Launcher Claude Code
│   └── download-model.js        # Descarga de modelos
│
├── .github/workflows/
│   └── deploy.yml               # CI/CD con GitHub Actions
│
└── package.json                 # Dependencias frontend + scripts
```

---

## API — endpoints principales

Todos los endpoints se sirven desde `http://localhost:3001`.

### Chat IA

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/api/chat` | Chat general (soporta `contextFiles` y `autoContext`) |
| POST | `/api/chat/brain` | SynkiaBrain — asistente con contexto de negocio |
| POST | `/claude/chat` | Proxy Claude Code → Ollama local |
| WS | `/ws/openclaw` | Agente OpenClaw en tiempo real |

### Negocio

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET/POST | `/api/trabajadores` | CRUD de empleados |
| GET/POST | `/api/documents` | Gestión documental |
| GET/POST | `/api/email` | Bandeja de email IMAP |
| GET/POST | `/api/revo/*` | Integración POS (Revo XEF) |
| GET/POST | `/api/biloop/*` | Integración contabilidad (Biloop) |
| GET/POST | `/api/filebrain/*` | RAG sobre documentos subidos |

### Sistema

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/health` | Health check completo de todos los servicios |
| GET | `/api/files/tree` | Árbol de archivos del proyecto |
| GET | `/api/files/read` | Leer contenido de archivos |
| GET | `/api/files/search` | Buscar en archivos |
| POST | `/api/auth/login` | Autenticación JWT |

---

## Paneles de chat

El archivo `public/chat.html` incluye 5 pestañas de chat, cada una con un modelo y propósito diferente:

| Pestaña | Modelo | Uso |
|---------|--------|-----|
| **Chat IA** | qwen3.5 | Conversación general, preguntas, brainstorming |
| **Brain** | codegemma:7b | Asistente con contexto de negocio (clasificación, análisis) |
| **OpenClaw** | functiongemma | Agente con herramientas (function calling) |
| **Aider** | qwen2.5-coder:14b | Edición de código asistida |
| **Claude Code** | qwen2.5-coder:14b | Proxy Claude Code con context picker y terminal |

Todos los paneles comparten una **barra de contexto global** que permite:
- Seleccionar archivos manualmente para inyectar en el prompt
- Activar auto-context (RAG) que busca archivos relevantes automáticamente

---

## Acceso remoto con Cloudflare Tunnel

Para exponer la plataforma a internet sin abrir puertos:

```bash
# Instalar cloudflared
brew install cloudflare/cloudflare/cloudflared

# Crear túnel (una sola vez)
cloudflared tunnel create synkia
cloudflared tunnel route dns synkia tudominio.com
cloudflared tunnel route dns synkia chat.tudominio.com

# Configurar ~/.cloudflared/config.yml
tunnel: YOUR_TUNNEL_ID
credentials-file: ~/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: tudominio.com
    service: http://localhost:3001
  - hostname: chat.tudominio.com
    service: http://localhost:3030
  - service: http_status:404

# Ejecutar
cloudflared tunnel run synkia

# Registrar en PM2 para auto-arranque
pm2 start cloudflared -- tunnel run synkia
pm2 save
```

---

## Gestión de procesos con PM2

```bash
# Arrancar todo
pm2 start server/index.js --name sinkia-api
pm2 start cloudflared -- tunnel run synkia
pm2 save

# Auto-arranque en reboot
pm2 startup
# (ejecuta el comando sudo que te indique)
pm2 save

# Monitorizar
pm2 monit
pm2 logs sinkia-api --lines 50

# Reiniciar
pm2 restart sinkia-api
pm2 restart all
```

---

## Problemas conocidos y fixes

### URLs de LM Studio hardcodeadas (resuelto)

> **Issue:** [#1](https://github.com/neo44hd/synk-ia-public/issues/1) | **Severidad:** Alta | **Estado:** Resuelto

**Problema:** Múltiples archivos tenían la URL de LM Studio (`localhost:12345`) hardcodeada. Al migrar a Ollama (`localhost:11434`), los servicios de IA fallaban con `ECONNREFUSED` porque seguían apuntando al puerto antiguo, ignorando `OLLAMA_URL`.

**Archivos afectados:**

| Archivo | Qué tenía mal |
|---------|---------------|
| `docker/docker-compose.yml` | `OPENAI_API_BASE_URL` apuntaba a `:12345` |
| `docker/setup.sh` | Health check usaba `:12345` en vez de `$OLLAMA_URL` |
| `scripts/start-litellm.sh` | `--api_base` fijo a `:12345` |
| Docs (README, SOUL.md, etc.) | Referenciaban LM Studio como dependencia |

**Solución:** Todas las URLs ahora leen de variables de entorno con fallback a Ollama:

```javascript
// Patrón correcto
const OLLAMA_URL = process.env.OLLAMA_URL || 'http://localhost:11434';

// Patrón incorrecto (nunca hardcodear)
const LLM_URL = 'http://localhost:12345';
```

**Verificación:** `curl http://localhost:3001/api/health` debe devolver todos los servicios `ok`.

**Lección:** Si migras de un LLM provider a otro, haz `grep -rn "puerto_antiguo"` para encontrar todas las referencias residuales.

---

## Troubleshooting

### El servidor arranca pero el chat no responde

```bash
# 1. Verificar que Ollama está corriendo
curl http://localhost:11434/api/tags

# 2. Verificar que el modelo está descargado
ollama list

# 3. Comprobar health check completo
curl http://localhost:3001/api/health | python3 -m json.tool
```

### `ECONNREFUSED` en los logs

Significa que un servicio intenta conectar a un puerto cerrado. Comprueba:

```bash
# ¿Ollama corre en el puerto esperado?
lsof -i :11434

# ¿Las variables de entorno están bien?
pm2 env <ID_PROCESO> | grep OLLAMA
```

### `posix_spawnp failed` al abrir el terminal

El módulo `node-pty` necesita compilarse para tu plataforma:

```bash
npm install node-pty --build-from-source
```

### Docker no conecta con Ollama

Ollama corre en el host, no en Docker. Los contenedores deben usar `host.docker.internal`:

```yaml
# docker-compose.yml
environment:
  - OPENAI_API_BASE_URL=http://host.docker.internal:11434/v1
extra_hosts:
  - "host.docker.internal:host-gateway"
```

### Cloudflare Tunnel no arranca

Verifica el orden de los argumentos — `run` va antes del nombre:

```bash
# Correcto
cloudflared tunnel run synkia

# Incorrecto (da error de --config)
cloudflared tunnel synkia run
```

---

## Contribuir

Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para la guía completa. En resumen:

1. Haz fork del repositorio
2. Crea tu rama: `git checkout -b feat/mi-mejora`
3. Haz commit siguiendo [Conventional Commits](https://www.conventionalcommits.org/es/): `git commit -m 'feat: descripción'`
4. Push: `git push origin feat/mi-mejora`
5. Abre un Pull Request con la [checklist del PR](CONTRIBUTING.md#checklist-del-pr)

Para reportar bugs o proponer mejoras, usa las [plantillas de issues](https://github.com/neo44hd/synk-ia-public/issues/new/choose).

---

## Licencia

MIT — consulta el archivo [LICENSE](LICENSE) para más detalles.

---

<p align="center">
  Hecho con ❤️ en Valencia, España<br/>
  <strong>SYNK-IA</strong> — IA local para PYMES que necesitan orden, no magia
</p>
