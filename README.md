# n8n Docker Deployment

A Docker Compose setup for deploying [n8n](https://n8n.io/) workflow automation with PostgreSQL database and optional AI/automation services.

## 🔗 Part of the n8n Workflow Manager Project

This repository provides the **deployment infrastructure** for the main project:  
👉 [n8n-workflow-manager](https://github.com/Kumaravel-Arumugam/n8n-workflow-manager)

---

## Default Services Included

This deployment comes with **6 services** pre-configured. You can use it as-is or customize for your needs:

| Service | Port | Purpose | Required |
|---------|------|---------|----------|
| **postgres** | 5432 | TimescaleDB database | ✅ Mandatory |
| **n8n** | 5678 | Workflow automation | ✅ Mandatory |
| **task-runners** | - | Python/JS code execution | Optional |
| **ollama** | 11434 | Local LLM (GPU required) | Optional |
| **qdrant** | 6333 | Vector database for RAG | Optional |
| **selenium** | 4444 | Browser automation | Optional |

---

## Quick Start (Use Exact Configuration)

If you want to use the **exact same setup** as this deployment:

```bash
# 1. Clone this repo
git clone https://github.com/Kumaravel-Arumugam/n8n-docker-deploy.git
cd n8n-docker-deploy

# 2. Create your environment file from template
cp .env.example .env

# 3. Edit .env with YOUR credentials
# (database password, API keys, domain, etc.)

# 4. Build and start all services
docker compose up -d --build

# 5. Access n8n at http://localhost:5678
```

> ⚠️ **Note**: The Ollama service requires an NVIDIA GPU. If you don't have one, see the User Guide for how to remove it.

---

## Custom Setup (AI-Assisted)

For a **minimal installation** (just Postgres + n8n) or to **customize services**:

📖 **See: [USER_GUIDE.md](USER_GUIDE.md)**

> **Note**: The User Guide is designed to be read by **AI/Agentic IDE assistants** (like Cursor, Windsurf, or similar). It contains structured instructions that help AI understand and modify this deployment based on your hardware and requirements.

The guide covers:

- Minimal setup (only Postgres + n8n)
- How to add/remove optional services
- Environment variables configuration
- Using secrets in n8n workflows (`{{ $env.VARIABLE }}`)
- Resource limits based on your hardware
- Docker commands reference
- Troubleshooting

---

## Enabling MCP Server (For AI Workflow Creation)

To use this n8n instance with the [n8n-workflow-manager](https://github.com/Kumaravel-Arumugam/n8n-workflow-manager):

1. Access n8n at `http://localhost:5678`
2. Go to **Settings** → **MCP Server** (if available) or ensure API access is enabled
3. Enable MCP Server
4. Copy the access token
5. Use this token in your Agentic IDE's MCP configuration

---

## Credits

Created by **Kumaravel Arumugam** using AI-assisted development.

---

## Files

| File | Purpose |
|------|---------|
| `docker-compose.yml` | Full stack configuration (all 6 services) |
| `.env.example` | Environment template - copy to `.env` |
| `USER_GUIDE.md` | Detailed setup & customization guide |
| `Dockerfile.runners` | Python packages for code execution |
| `n8n-task-runners.json` | Task runner configuration |
