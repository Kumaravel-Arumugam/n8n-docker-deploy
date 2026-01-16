# n8n Docker Deployment - User Guide

> **For AI/Agentic IDE**: This document is structured to help you understand and modify this deployment. Key sections marked with 🤖 contain instructions specifically for AI-assisted editing. When helping users, dynamically adjust resource limits based on their hardware specifications.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Quick Start (Minimal Setup)](#quick-start-minimal-setup)
4. [Full Setup (All Services)](#full-setup-all-services)
5. [Environment Variables](#environment-variables)
6. [Using Variables in n8n Nodes](#using-variables-in-n8n-nodes)
7. [Optional Services](#optional-services)
8. [Docker Commands Reference](#docker-commands-reference)
9. [Resource Limits Configuration](#resource-limits-configuration)
10. [Troubleshooting](#troubleshooting)

---

## Overview

This project deploys n8n (workflow automation) with Docker Compose. It includes:

| Service | Purpose | Required? |
|---------|---------|-----------|
| **postgres** | Database for n8n | ✅ Mandatory |
| **n8n** | Workflow automation platform | ✅ Mandatory |
| **task-runners** | Python/JS code execution in workflows | ⚪ Optional |
| **ollama** | Local LLM inference (requires GPU) | ⚪ Optional |
| **qdrant** | Vector database for AI/RAG workflows | ⚪ Optional |
| **selenium** | Browser automation for web scraping | ⚪ Optional |

---

## Prerequisites

- Docker Desktop installed and running
- At least 4GB RAM available for containers
- (Optional) NVIDIA GPU with drivers for Ollama

---

## Quick Start (Minimal Setup)

This deploys only **Postgres + n8n** - the minimum required services.

### Step 1: Create Your Environment File

```bash
cp .env.example .env
```

Edit `.env` with your values:

```env
POSTGRES_USER=your_username
POSTGRES_PASSWORD=your_secure_password
POSTGRES_DB=n8n_database
N8N_RUNNERS_AUTH_TOKEN=any-random-string
DOMAIN_NAME=localhost:5678
N8N_EDITOR_BASE_URL=http://localhost:5678/
WEBHOOK_URL=http://localhost:5678/
```

### Step 2: Create Minimal docker-compose.yml

Create a new file `docker-compose.minimal.yml` or modify `docker-compose.yml`:

```yaml
services:

  postgres:
    image: timescale/timescaledb:latest-pg16
    container_name: postgres_db
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: "${POSTGRES_USER}"
      POSTGRES_PASSWORD: "${POSTGRES_PASSWORD}"
      POSTGRES_DB: "${POSTGRES_DB}"
    volumes:
      - ./database_postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 1G

  n8n:
    image: n8nio/n8n
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    volumes:
      - ./n8n_data_postgres:/home/node/.n8n
    env_file:
      - .env
    environment:
      # Database
      DB_TYPE: "postgresdb"
      DB_POSTGRESDB_HOST: "postgres"
      DB_POSTGRESDB_PORT: "5432"
      DB_POSTGRESDB_DATABASE: "${POSTGRES_DB}"
      DB_POSTGRESDB_USER: "${POSTGRES_USER}"
      DB_POSTGRESDB_PASSWORD: "${POSTGRES_PASSWORD}"
      
      # URLs
      N8N_EDITOR_BASE_URL: "${N8N_EDITOR_BASE_URL}"
      WEBHOOK_URL: "${WEBHOOK_URL}"
      
      # Settings
      GENERIC_TIMEZONE: "UTC"
      N8N_COMMUNITY_PACKAGES_ENABLED: "true"
    depends_on:
      postgres:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: 2G
```

### Step 3: Start Services

```bash
docker compose up -d
```

Access n8n at: `http://localhost:5678`

---

## Full Setup (All Services)

Use the provided `docker-compose.yml` for the complete stack with all optional services.

```bash
# 1. Create environment file
cp .env.example .env

# 2. Edit .env with your values
# (see Environment Variables section)

# 3. Build and start all services
docker compose up -d --build
```

---

## Environment Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `POSTGRES_USER` | Database username | `n8n_user` |
| `POSTGRES_PASSWORD` | Database password | `secure_password_123` |
| `POSTGRES_DB` | Database name | `n8n_database` |
| `N8N_RUNNERS_AUTH_TOKEN` | Auth token for task runners | `random-secure-token` |
| `DOMAIN_NAME` | Your domain or localhost | `localhost:5678` |
| `N8N_EDITOR_BASE_URL` | Full URL to n8n editor | `http://localhost:5678/` |
| `WEBHOOK_URL` | Webhook callback URL | `http://localhost:5678/` |

### Optional API Keys

Add any API keys your workflows need:

```env
TELEGRAM_BOT_TOKEN=your_bot_token
OPENAI_API_KEY=sk-your-key
# Add more as needed
```

---

## Using Variables in n8n Nodes

> **🤖 AI Note**: When users ask how to use environment variables in workflows, direct them to use expressions, not hardcoded values.

### Adding a New Variable

**Example**: Adding `TESTING_KEY = 12345678910`

1. **Add to `.env` file**:
   ```env
   TESTING_KEY=12345678910
   ```

2. **Add to allowlist in `docker-compose.yml`**:
   ```yaml
   N8N_ENV_ALLOWLIST: "TELEGRAM_BOT_TOKEN,TESTING_KEY"
   ```

3. **Restart containers**:
   ```bash
   docker compose down
   docker compose up -d
   ```

4. **Use in n8n node** (Expression mode, not Fixed):
   - In Expression: `{{ $env.TESTING_KEY }}`
   - In Code Node: `process.env.TESTING_KEY`

### Important Rules

- ❌ **Never** hardcode secrets in nodes
- ✅ **Always** use `{{ $env.VARIABLE_NAME }}` expression
- ✅ Set field to "Expression" mode (not "Fixed")
- ✅ Add variable name to `N8N_ENV_ALLOWLIST`

---

## Optional Services

### Task Runners (Python/JavaScript Code Execution)

Required files: `Dockerfile.runners`, `n8n-task-runners.json`

Enables running Python and JavaScript code in n8n workflows with custom packages:
- **Python**: numpy, pandas, matplotlib, requests, selenium, etc.
- **JavaScript**: moment, uuid, crypto

To add more Python packages, edit `Dockerfile.runners`:
```dockerfile
RUN cd /opt/runners/task-runner-python && \
    uv pip install your_package_name
```

---

### Ollama (Local LLM)

**Requirements**: NVIDIA GPU with CUDA drivers

Provides local AI model inference. Remove if you don't have a GPU or prefer cloud AI APIs.

To remove:
1. Delete the `ollama` service block from `docker-compose.yml`
2. Remove `ollama` from n8n's `depends_on` section

---

### Qdrant (Vector Database)

Used for AI/RAG (Retrieval Augmented Generation) workflows.

To remove:
1. Delete the `qdrant` service block from `docker-compose.yml`
2. Remove `qdrant` from n8n's `depends_on` section

---

### Selenium (Browser Automation)

Used for web scraping and browser automation in workflows.

To remove:
1. Delete the `selenium` service block from `docker-compose.yml`

---

## Docker Commands Reference

### Basic Operations

```bash
# Start all services
docker compose up -d

# Start with rebuild (after changing Dockerfile)
docker compose up -d --build

# Stop all services
docker compose down

# Stop and remove volumes (DELETES DATA!)
docker compose down -v

# View logs
docker compose logs -f

# View logs for specific service
docker compose logs -f n8n

# Restart a single service
docker compose restart n8n
```

### Maintenance

```bash
# Check service status
docker compose ps

# Enter container shell
docker compose exec n8n sh
docker compose exec postgres psql -U ${POSTGRES_USER} -d ${POSTGRES_DB}

# Pull latest images
docker compose pull
```

---

## Resource Limits Configuration

> **🤖 AI Note**: When users share their hardware specs, calculate appropriate memory limits. Use these guidelines:
> - Reserve 2-4GB for host OS
> - Postgres: 10-15% of available RAM
> - n8n: 15-25% of available RAM  
> - Other services: Scale based on usage

### Memory Limits by System RAM

| System RAM | Postgres | n8n | Ollama | Qdrant | Total |
|------------|----------|-----|--------|--------|-------|
| 8GB | 1G | 2G | - | - | ~3G |
| 16GB | 2G | 3G | 4G | 1G | ~10G |
| 32GB | 2G | 4G | 8G | 2G | ~16G |
| 64GB+ | 4G | 6G | 16G | 4G | ~30G |

### Modifying Resource Limits

In `docker-compose.yml`, adjust the `deploy.resources.limits.memory` value:

```yaml
deploy:
  resources:
    limits:
      memory: 2G  # Change this value
```

### GPU Configuration (Ollama)

The Ollama service requires NVIDIA GPU. If you don't have one, either:
1. Remove the Ollama service entirely
2. Remove the `runtime: nvidia` and GPU-related lines

---

## Troubleshooting

### Port Already in Use

```bash
# Find what's using the port
netstat -ano | findstr :5678

# Kill the process or change port in docker-compose.yml
```

### Network/Firewall Issues (Windows)

```bash
netsh int ipv4 reset
netsh advfirewall reset
```

### Container Won't Start

```bash
# Check logs for errors
docker compose logs n8n

# Rebuild from scratch
docker compose down
docker compose build --no-cache
docker compose up -d
```

### Database Connection Issues

Ensure Postgres is healthy before n8n starts:
```bash
docker compose ps  # Check postgres_db status
docker compose logs postgres
```

---

## File Structure

```
├── .env                    # Your secrets (DO NOT COMMIT)
├── .env.example            # Template for .env
├── .gitignore              # Git exclusions
├── docker-compose.yml      # Full stack configuration
├── Dockerfile.runners      # Python packages for task runners
├── n8n-task-runners.json   # Task runner configuration
├── README.md               # Project overview
├── USER_GUIDE.md           # This file
├── database_postgres/      # Postgres data (auto-created)
├── n8n_data_postgres/      # n8n data (auto-created)
├── ollama_data/            # Ollama models (auto-created, optional)
└── qdrant/                 # Qdrant storage (auto-created, optional)
```

---

## For AI/Agentic IDE Assistants

🤖 **When helping users set up this project:**

1. **Ask about their hardware** (RAM, GPU) to configure resource limits
2. **Ask which services they need** (minimal: postgres+n8n, or full stack)
3. **Generate `.env`** with appropriate placeholder values
4. **Adjust `docker-compose.yml`** resource limits based on their system
5. **Remove unused services** (ollama if no GPU, selenium if no scraping needed)
6. **Remind users** to never commit `.env` or credentials

**Key files to modify:**
- `.env` - User's secrets and configuration
- `docker-compose.yml` - Service definitions and resource limits
- `Dockerfile.runners` - Python packages for code execution
