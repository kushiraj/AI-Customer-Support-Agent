# AI-Customer-Support-Agent

# RecallDesk — AI Customer Support Agent with Hindsight

RecallDesk is an AI-powered customer support application that uses long-term memory to provide more contextual support, preserve conversation history, manage tickets, and reuse successful troubleshooting knowledge.

The central idea is to keep **customer-specific memory separate from shared troubleshooting knowledge**. This lets the agent recall a customer's own history while drawing on solutions that have helped other customers—without retrieving another customer's private memory bank.

## Table of Contents

- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Seed Demo Data](#seed-demo-data)
- [Run the Application](#run-the-application)
- [API and Interactive Docs](#api-and-interactive-docs)
- [Testing and Evaluation](#testing-and-evaluation)
- [Privacy and Limitations](#privacy-and-limitations)
- [Future Improvements](#future-improvements)
- [Learn More](#learn-more)

## Key Features

- **Two-layer memory:** A private Hindsight memory bank per customer, plus a shared troubleshooting playbook.
- **Context-aware responses:** Retrieve relevant customer history and shared fixes before generating an answer.
- **Memory toggle:** Compare responses with memory enabled and disabled.
- **Conversation history:** Persist chats in SQLite and reopen previous conversations.
- **Ticket management:** Create and track support tickets with statuses and inferred priorities.
- **Human handoff:** Support direct human requests and escalation after repeated unresolved feedback.
- **Memory hygiene:** Configurable PII-pattern redaction and near-duplicate memory checks.
- **Customer memory controls:** Inspect memories and request that customer memory be forgotten.
- **Evaluation support:** Offline tests and live-instance evaluation endpoints/scripts.

## How It Works

```text
Customer
   |
   v
RecallDesk Web UI
   |
   v
FastAPI Chat Endpoint
   |
   +--> Recall customer-specific memory
   |
   +--> Recall shared playbook
   |
   v
Build context and generate response
   |
   +--> Save conversation to SQLite
   |
   +--> Create/update support ticket when needed
   |
   v
Customer feedback / human handoff
   |
   v
Retain outcome in customer memory
   |
   +--> Add reusable, privacy-processed learning to shared playbook
```

### Memory design

- `cust-<customer_id>` — customer-specific context, history, environment, preferences, and outcomes.
- `recalldesk-playbook` — shared troubleshooting knowledge and resolution outcomes.

The application uses Hindsight for memory operations such as bank creation, recall, retention, reflection, and memory listing. Memory failures are handled so that a failed memory operation does not necessarily stop the chat.

## Technology Stack

| Technology | Purpose |
|---|---|
| Python | Application logic |
| FastAPI | Backend API |
| Hindsight | Long-term memory |
| OpenAI-compatible API | LLM access |
| SQLite | Conversation and ticket persistence |
| HTML, CSS, JavaScript | Web interface |
| Pytest | Automated tests |

## Project Structure

```text
.
├── main.py                  # FastAPI application and core logic
├── seed.py                  # Seeds demo customer histories and playbook entries
├── static/
│   └── index.html           # Web UI
├── tests/
│   └── test_recalldesk.py   # Offline test suite
├── eval.py                  # Evaluation against a running instance
├── requirements.txt         # Runtime dependencies
├── requirements-dev.txt     # Development/test dependencies
├── .env.example             # Environment configuration template
├── Dockerfile               # Container build
└── .github/workflows/
    └── tests.yml            # CI test workflow
```

## Getting Started

### Prerequisites

- Python 3.12 recommended
- A Hindsight service/API key
- An API key for an OpenAI-compatible LLM endpoint (the example configuration uses Groq)

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

Replace the placeholders with your repository URL and folder name.

### 2. Create and activate a virtual environment

**Windows (PowerShell):**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Copy `.env.example` to `.env` and fill in your own credentials.

**Windows (PowerShell):**

```powershell
Copy-Item .env.example .env
```

**macOS / Linux:**

```bash
cp .env.example .env
```

Load the values into your environment using your preferred method or a dotenv tool. Do not commit `.env` or publish API keys.

## Configuration

The application reads configuration from environment variables.

| Variable | Purpose |
|---|---|
| `HINDSIGHT_URL` | Hindsight API base URL |
| `HINDSIGHT_API_KEY` | Hindsight API key |
| `LLM_API_KEY` | API key for the configured LLM provider |
| `LLM_BASE_URL` | OpenAI-compatible API base URL |
| `LLM_MODELS` | Comma-separated model fallback list |
| `ADMIN_KEY` | Secret for protected admin/evaluation endpoints |
| `COMPANY_NAME` | Company name shown/configured by the app |
| `RATE_LIMIT_PER_MIN` | Per-minute rate limit setting |
| `DB_PATH` | SQLite database path |
| `DEDUPE_THRESHOLD` | Similarity threshold used for duplicate-memory checks (default: `0.82`) |
| `FAIL_STREAK_ESCALATE` | Consecutive unresolved feedback count before automatic escalation (default: `2`) |

See `.env.example` for the complete template. Use strong, private values for secrets.

## Seed Demo Data

With Hindsight configured, run:

```bash
python seed.py
```

The script creates demo memory banks and adds sample support history and reusable playbook entries for `northwind-logistics` and `brightpath-health`.

The seed script may report that a bank already exists if you run it more than once. Allow Hindsight time to consolidate newly retained memories before testing recall.

## Run the Application

Start the API server:

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

Then open:

```text
http://localhost:8000
```

Create a demo API key using the admin key configured in your environment:

```bash
curl -X POST http://localhost:8000/admin/keys \
  -H "X-Admin-Key: $ADMIN_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"demo"}'
```

Use the returned API key in the UI. For the seeded walkthrough, use customer ID `northwind-logistics`.

## API and Interactive Docs

When the server is running, FastAPI's interactive API documentation is available at:

- `http://localhost:8000/docs`
- `http://localhost:8000/health`

The application includes endpoints for chat, feedback, conversations, tickets, customer memory inspection/briefs, human handoff, admin key management, and evaluation. See `/docs` for the exact request and response schemas.

## Testing and Evaluation

Install development dependencies:

```bash
pip install -r requirements-dev.txt
```

Run the offline test suite:

```bash
pytest tests/ -v
```

The offline tests use fakes for external memory/LLM dependencies and cover behaviors such as redaction, deduplication, conversation persistence, ticket creation and priority, escalation, and customer isolation.

For evaluation against a running instance:

```bash
python eval.py --base-url http://localhost:8000 --admin-key "$ADMIN_KEY"
```

The app also provides admin-key-protected evaluation endpoints for memory, isolation, and hygiene. Refer to the API docs for exact paths and headers.

## Suggested Demo Walkthrough

1. Send a support issue with memory **OFF** and observe the general response.
2. Enable memory and repeat the issue for `northwind-logistics`; inspect the recalled customer context and playbook knowledge.
3. Mark the issue resolved to demonstrate how the outcome is retained.
4. Switch to `brightpath-health` and test a similar issue to demonstrate shared-playbook retrieval without using the first customer's private bank.
5. Open a new chat, then revisit a previous conversation to show persistence.
6. Submit unresolved feedback twice, or explicitly request a human, to demonstrate escalation.
7. Demonstrate the customer-memory forget control.

## Privacy and Limitations

- Customer-specific memory banks are separated from the shared playbook.
- PII redaction uses configured patterns; it is a safeguard, **not a guarantee** that all sensitive information will be detected.
- Shared playbook entries should be reviewed and validated before production use. Removing obvious identifiers does not guarantee complete anonymization.
- The forget flow depends on the capabilities of the installed Hindsight client. If direct deletion is unavailable, a tombstone fallback is not the same as physically deleting stored data.
- Memory can be incomplete, outdated, or irrelevant. The agent should verify current details rather than blindly trust prior outcomes.
- Automated tests and demo evaluations do not establish production readiness or guarantee response correctness.

## Future Improvements

- Stronger validation and review of shared playbook entries.
- More comprehensive privacy, deletion, and cross-customer isolation tests.
- Broader evaluation using realistic support scenarios.
- Measurement of whether memory reduces repeated troubleshooting and improves resolution outcomes.

## Learn More

- [Hindsight on GitHub](https://github.com/vectorize-io/hindsight)
- [Hindsight Documentation](https://hindsight.vectorize.io/)
- [What Is Agent Memory? — Vectorize](https://vectorize.io/what-is-agent-memory)

---

Built as a project exploring long-term memory for AI-powered customer support.
