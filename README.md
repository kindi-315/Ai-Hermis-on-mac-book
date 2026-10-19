# Ai-Hermis-on-mac-book
# Local AI Agent Environment: Hermes Agent + Ollama (Dockerized)

A fully localized, private, and containerized AI agent environment orchestrated using Docker Compose. This setup pairs **Hermes Agent** (frontend/agentic framework) with **Ollama** (backend inference engine) running entirely on small-scale local models.

---

## 💻 System Specifications
* **Machine:** MacBook Air (Apple Silicon / M-series or compatible)
* **Unified Memory / RAM:** 3.8 GB / 4 GB
* **Containerization:** Docker & Docker Compose
* **Orchestrated Models:**
  1. `llama3.2:1b` (Lightweight general reasoning)
  2. `qwen2.5:1.5b` (Precise coding and direct instruction following)
  3. `phi3:mini` (Alternative compact model)

---

## 🛠️ Architecture Overview
* **Ollama Container (`ollama:latest`):** Acts as the local LLM inference provider, serving models via an OpenAI-compatible API (`http://ollama:11434/v1`).
* **Hermes Agent Container (`nousresearch/hermes-agent:latest`):** Acts as the agentic interface equipped with toolsets, running inside a shared Docker bridge network.

---

## 📋 Step-by-Step Installation Guide

### 1. Create `docker-compose.yml`
Create a project directory and set up the following configuration to ensure the containers remain stable and persistent:

```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    restart: unless-stopped

  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes_agent
    ports:
      - "8642:8642"
      - "9119:9119"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    command: ["tail", "-f", "/dev/null"] # Keeps the container alive as a stable daemon
    volumes:
      - hermes_data:/opt/data
    depends_on:
      - ollama
    restart: unless-stopped

volumes:
  ollama_data:
  hermes_data:

```

### 2. Run and Initialize

Deploy the containers in the background:

```bash
docker compose up -d

```

### 3. Pull Local Models

Access the Ollama container and pull the desired small-scale models:

```bash
docker exec -it ollama ollama pull llama3.2:1b
docker exec -it ollama ollama pull qwen2.5:1.5b
docker exec -it ollama ollama pull phi3:mini

```

### 4. Configure Hermes to Use Ollama

Configure Hermes config settings to route requests properly to the local backend:

```bash
docker exec -it hermes_agent hermes config set providers.default ollama
docker exec -it hermes_agent hermes config set providers.ollama.base_url http://ollama:11434/v1

```

---

## ⚠️ Encountered Hurdles & Solutions

1. **Container Auto-Shutdown (Health/Restart Loop):**
* *Problem:* Hermes container kept shutting down immediately due to non-interactive terminal triggers (`Warning: Input is not a terminal`).
* *Solution:* Added `command: ["tail", "-f", "/dev/null"]` in `docker-compose.yml` to force the container to remain persistently active as a background daemon.


2. **Dashboard Security Lockout:**
* *Problem:* The web dashboard refused binds to `0.0.0.0` due to unconfigured authentication providers.
* *Solution:* Bypassed unnecessary public dashboard exposure and utilized the CLI chat interface tied directly to the local backend.


3. **API 404 Endpoint Errors:**
* *Problem:* Initial requests failed with `HTTP 404: page not found`.
* *Solution:* Appended `/v1` to the Ollama base URL configuration (`http://ollama:11434/v1`) to match OpenAI-compatible endpoint routes expected by Hermes.



---

## 🔌 How to Add Custom Skills

Hermes Agent loads bundled and custom skills directly into the persistent volume directory. To add new skills:

1. Place your custom skill folders or scripts inside your local mapped volume directory (or directly inside the container at `~/.hermes/skills/` or `/opt/data/skills/`).
2. Inside the container, reload or sync skills using the built-in CLI command:
```bash
docker exec -it hermes_agent hermes skills sync

```


3. Alternatively, author custom skills using Hermes's built-in authoring tools or place Markdown/Python-based skill definitions into the skills directory.

---

## 📦 How to Migrate This Project to Another Laptop

To clone and run this exact environment on a brand-new machine:

1. **Install Prerequisites:** Ensure **Docker Desktop** and **Git** are installed on the target laptop.
2. **Transfer Project Files:** Copy your project folder containing `docker-compose.yml` and any configuration files to the new machine (via Git or USB).
3. **Build & Spin Up:**
```bash
docker compose up -d

```


4. **Re-download Models (Ollama Data is Local):** Since Docker volumes are local to each machine, pull the models again on the new laptop:
```bash
docker exec -it ollama ollama pull llama3.2:1b
docker exec -it ollama ollama pull qwen2.5:1.5b

```


5. **Apply Configuration:** Run the configuration setup commands once:
```bash
docker exec -it hermes_agent hermes config set providers.default ollama
docker exec -it hermes_agent hermes config set providers.ollama.base_url http://ollama:11434/v1

```


6. **Start Chatting:**
```bash
docker exec -it hermes_agent hermes chat --model ollama/llama3.2:1b

```
