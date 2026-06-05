# AI-Powered Customer Support Agent

An end-to-end customer support copilot that generates context-aware draft replies using LLMs, long-term customer memory, retrieval augmented generation, and tool calling. The system combines a FastAPI backend, Streamlit dashboard, SQLite persistence, ChromaDB knowledge search, Mem0 memory, and Docker-based deployment.

## Features

- **AI Draft Generation**: Generate support-ready draft replies for customer tickets using Groq-hosted LLMs.
- **Long-Term Customer Memory**: Save accepted resolutions and retrieve customer-specific context with Mem0.
- **Knowledge Base RAG**: Ingest support documents and retrieve relevant policy/FAQ snippets through ChromaDB.
- **Tool Calling**: Enrich responses with structured tools for customer plan lookup and open-ticket load.
- **Ticket & Draft Workflow**: Create tickets, generate drafts, edit drafts, accept/discard drafts, and resolve tickets.
- **Streamlit Dashboard**: Agent-friendly UI for ticket management, draft review, context inspection, and memory probing.
- **FastAPI Backend**: REST API with typed Pydantic schemas and interactive Swagger docs.
- **Docker Ready**: Run the API and dashboard together with Docker Compose.
- **CI/CD Ready**: GitHub Actions workflows for tests and EC2 deployment.

## Tech Stack

- **Backend**: FastAPI, Uvicorn
- **Frontend**: Streamlit
- **LLM**: Groq via LangChain
- **Agent Framework**: LangChain agents with tool calling
- **Memory**: Mem0
- **Vector Database**: ChromaDB
- **Embeddings**: Gemini, OpenAI, or local Ollama embeddings
- **Database**: SQLite
- **Dependency Management**: uv
- **Deployment**: Docker, Docker Compose, GitHub Actions, EC2
- **Language**: Python 3.12+

## Project Structure

```text
Customer_Support_Agent/
|-- app.py                              # Streamlit dashboard
|-- main.py                             # FastAPI app entrypoint
|-- Dockerfile                          # Container image definition
|-- docker-compose.yml                  # API + dashboard services
|-- pyproject.toml                      # Project dependencies
|-- uv.lock                             # Locked dependency graph
|-- knowledge_base/                     # Support knowledge documents
|-- docs/                               # Deployment and project notes
|-- tests/                              # Automated tests
|-- .github/workflows/                  # CI/CD workflows
`-- customer_support_agent/
    |-- api/                            # FastAPI routers and dependencies
    |-- core/                           # Settings and app configuration
    |-- integrations/
    |   |-- memory/                     # Mem0 customer memory store
    |   |-- rag/                        # Chroma knowledge-base retrieval
    |   `-- tools/                      # LangChain support tools
    |-- repositories/sqlite/            # SQLite repositories
    |-- schemas/                        # Pydantic API schemas
    `-- services/                       # Copilot, draft, and knowledge services
```

## Getting Started

### Prerequisites

- Python 3.12 or higher
- uv package manager
- Groq API key
- Optional: Google API key, OpenAI API key, or local Ollama setup for embeddings
- Optional: Docker Desktop for containerized execution

### Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/deep1305/Customer_Support_Agent_live.git
   cd Customer_Support_Agent_live
   ```

2. **Install dependencies**

   ```bash
   uv sync --dev
   ```

3. **Create a local environment file**

   Create `.env` in the project root:

   ```env
   GROQ_API_KEY=your_groq_api_key_here
   GOOGLE_API_KEY=your_google_api_key_here
   API_BASE_URL=http://localhost:8000
   ENABLE_LOCAL_EMBEDDINGS=false
   ```

   Keep `.env` local only. Do not commit API keys.

4. **Run the FastAPI backend**

   ```bash
   uv run python main.py
   ```

   API docs will be available at:

   ```text
   http://localhost:8000/docs
   ```

5. **Run the Streamlit dashboard**

   In a second terminal:

   ```bash
   uv run streamlit run app.py
   ```

   Dashboard will be available at:

   ```text
   http://localhost:8501
   ```

## Usage

1. Open the Streamlit dashboard.
2. Create a customer ticket with subject, description, priority, and customer details.
3. Generate an AI draft response.
4. Review the generated draft and inspect the context used.
5. Accept the draft to resolve the ticket and save useful resolution memory.
6. Use Memory Probe to search previous customer memories.

Example support scenarios:

- ATM withdrawal issues
- Savings account rules
- Minimum balance and banking charges
- KYC and account update questions
- Recurring customer support history

## Knowledge Base Ingestion

The app can ingest `.md` and `.txt` files from `knowledge_base/` into ChromaDB.

Through Swagger:

```http
POST /api/knowledge/ingest
```

Example body:

```json
{
  "clear_existing": true
}
```

Through the dashboard, use the **Ingest Knowledge Base** button.

## API Overview

Common endpoints:

```text
GET  /health
POST /api/tickets
GET  /api/tickets
GET  /api/tickets/{ticket_id}
POST /api/tickets/{ticket_id}/generate-draft
GET  /api/drafts/{ticket_id}
PATCH /api/drafts/{draft_id}
POST /api/knowledge/ingest
GET  /api/customers/{customer_id}/memories
GET  /api/customers/{customer_id}/memory-search
```

Interactive API documentation:

```text
http://localhost:8000/docs
```

## Docker Deployment

Run both API and dashboard with Docker Compose:

```bash
docker compose up --build
```

Services:

- API: `http://localhost:8000`
- API Docs: `http://localhost:8000/docs`
- Dashboard: `http://localhost:8501`

Stop services:

```bash
docker compose down
```

## GitHub Actions and EC2 Deployment

This project includes GitHub Actions workflows for CI and EC2 deployment:

```text
.github/workflows/ci.yml
.github/workflows/deploy-ec2.yml
```

Recommended GitHub secrets:

```text
EC2_HOST
EC2_USER
EC2_PORT
EC2_APP_DIR
EC2_SSH_KEY
EC2_ENV_FILE
```

For Ubuntu EC2, a typical app directory is:

```text
/home/ubuntu/customer_support_agent
```

The deployment workflow packages the repository, uploads it to EC2, installs Docker if needed, runs Docker Compose, and checks the API health endpoint.

## Environment Variables

Common local `.env` variables:

```env
GROQ_API_KEY=your_groq_api_key_here
GROQ_MODEL=qwen/qwen3-32b
MEMORY_GROQ_MODEL=llama-3.1-8b-instant
GOOGLE_API_KEY=your_google_api_key_here
GOOGLE_EMBEDDING_MODEL=gemini-embedding-2
OPENAI_API_KEY=your_openai_api_key_here
ENABLE_LOCAL_EMBEDDINGS=false
API_BASE_URL=http://localhost:8000
```

Only set the keys/providers you use. Store production values as GitHub Secrets or server-side environment variables, not in Git.

## Testing

Run tests locally:

```bash
uv run pytest -q
```

The CI workflow runs the same test command on GitHub Actions.

## How It Works

1. **Ticket Creation**: Customer and ticket records are stored in SQLite.
2. **Knowledge Retrieval**: The ticket query is searched against ChromaDB knowledge-base chunks.
3. **Memory Retrieval**: Mem0 retrieves prior customer and company memories.
4. **Tool Calling**: Support tools provide structured customer plan and ticket-load context.
5. **Draft Generation**: LangChain routes the prompt, retrieved context, and tool output to the LLM.
6. **Draft Review**: Agents can edit, accept, or discard generated drafts.
7. **Memory Update**: Accepted resolutions are stored for future customer interactions.

## Security Notes

- Do not commit `.env`, API keys, private SSH keys, or generated database files.
- Rotate any API key that has been pasted into chat, logs, screenshots, or public repositories.
- Keep EC2 security groups restricted to required ports only.
- Store deployment credentials in GitHub Secrets.

## Acknowledgments

- Built with FastAPI, Streamlit, LangChain, ChromaDB, and Mem0.
- Powered by Groq LLM inference and configurable embedding providers.
- Designed as a production-style AI support copilot with memory and tool calling.
