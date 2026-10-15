# Project Design: AI Ticket Triage Agent

A bounded agentic system for support-ticket triage.

**Status:** In development. Phase 1 (foundation) is in progress. The phases and feature lists below describe the planned scope.

## Overview

A support-ticket triage system where an AI agent reasons about incoming tickets, retrieves relevant knowledge, decides whether it can resolve a ticket autonomously or must escalate to a human, and improves over time from human corrections, all inside a governed, auditable, tool-scoped structure.

**Classification:** an end-to-end, AI-integrated, bounded workflow agent: autonomous reasoning, tool use and decision-making within fixed governance rails, rather than an open-ended, free-planning agent.

## Architecture

```
[Ticket Source] -> [Ingestion API] -> [Governance Gateway: PII scrub, injection check]
        -> [Classification Agent: intent, urgency, sentiment]
        -> [Retrieval Layer: vector search over knowledge base]
        -> [Reasoning/Verification: confidence scoring, step-by-step check]
        -> [Decision: auto-resolve | draft + human review | escalate]
        -> [Action: response sent | ticket routed | alert raised]
        -> [Feedback Loop: human corrections logged -> future accuracy improves]
        -> [Evaluation & Drift Monitoring: continuous accuracy/cost tracking]
```

## Build Phases

Each phase ends in a working, demoable state.

| Phase | Focus | Description |
|---|---|---|
| 1 | Foundation | Ticket data model, ingestion API, keyword-based classification as a measured baseline |
| 2 | LLM classification | Replace keyword logic with LLM-based intent, urgency and sentiment classification over a small taxonomy (5 to 10 categories) |
| 3 | RAG pipeline | Vector database, embeddings, knowledge base ingestion and retrieval-augmented response drafting |
| 4 | Confidence and decisions | Confidence scoring, auto-resolve versus escalate logic, React dashboard for human review |
| 5 | Feedback loop | Capture human corrections and feed them back into classification (few-shot examples or lightweight retraining) |
| 6 | Reasoning-first upgrade | Step-by-step verified reasoning instead of plain RAG, ticket clustering and deduplication, model routing by complexity |
| 7 | Governance and security | Input and output guardrails (PII redaction, injection defense), least-privilege tool scoping, sandboxed action execution |
| 8 | Evaluation and verification | Adversarial test suite, labeled evaluation set, accuracy, cost and drift tracking, CI-integrated regression testing |
| 9 | Deployment and observability | Docker, CI/CD, cloud deployment, logging and monitoring, circuit-breaker fallback if accuracy drops |
| 10 | Documentation | Architecture diagram, governance and evaluation write-up, demo script |

## Planned Features

### Core

- Ticket ingestion and storage
- LLM-based classification (intent, urgency, sentiment)
- RAG-based knowledge retrieval
- Confidence scoring with auto-resolve and escalate logic
- Human review dashboard

### Differentiators

- Reasoning-first verification: step-by-step checking against sources, not just summarization
- Ticket clustering and deduplication to detect many tickets with the same root cause
- Model routing by complexity (cost and accuracy optimization)
- Auto-drafted knowledge base articles from novel human resolutions
- Root-cause trend detection surfaced to a product-team view

### Governance and Security

- Input and output guardrail gateway (PII redaction, prompt injection detection)
- Least-privilege tool scoping: the agent can only call explicitly permitted functions
- Sandboxed execution for any agent-proposed action
- Full audit trail: every decision is reconstructable (what was retrieved, reasoning steps, confidence)
- Circuit breaker: automatic fallback to full human routing if accuracy drops

### Evaluation and Verification

- Adversarial prompt-injection test suite with pass/fail tracking
- PII leak testing (input redaction and log scrubbing)
- Tool-scoping penetration test confirming restricted actions are structurally impossible, not just discouraged
- Labeled evaluation set with accuracy tracked over time, starting from the keyword baseline
- Token cost instrumentation with before and after optimization comparison
- Drift detection: simulate a new ticket category and confirm monitoring flags it
- All of the above wired into CI/CD as automated regression tests

## Tech Stack

The system is split into two cooperating services: a **Node.js core** for tickets, authentication, the dashboard and orchestration, and a **Python (FastAPI) AI service** for reasoning, retrieval and guardrails.

| Layer | Tool | Service |
|---|---|---|
| Core backend (tickets, users, routing) | Node.js, Express, TypeScript | Node |
| Structured database | PostgreSQL | Shared |
| Knowledge base storage | PostgreSQL (pgvector) or MongoDB | Shared |
| Authentication | JWT with role-based access control | Node |
| Queue (async ticket processing) | Redis, BullMQ | Node |
| Frontend dashboard | React | Node |
| AI classification and reasoning | FastAPI, Pydantic | Python |
| RAG pipeline | LangChain or LlamaIndex | Python |
| Vector store | pgvector, Pinecone or Chroma | Python |
| Embeddings | Hosted embeddings API or sentence-transformers | Python |
| LLM calls | Claude or OpenAI API, or a local model via Ollama | Python |
| Agent orchestration | LangGraph | Python |
| Guardrails (PII, injection defense) | Custom FastAPI middleware | Python |
| Inter-service communication | Internal REST calls | Both |
| Deployment | Docker and Docker Compose (Node, FastAPI, Postgres, Redis), then cloud | Both |
| CI/CD | GitHub Actions running evaluation and security tests on every change | Both |
| Observability | Structured logging, with LLM tracing added later | Python |

**Why the split:** the Node service handles the conventional product logic. The Python service isolates the AI ecosystem's strongest tooling (LangChain, LangGraph, native vector database clients) so AI logic stays out of the Express app. The two communicate over a simple internal API, so each can be developed, tested and scaled independently.

### Self-Hosted and Free-Tier Option

The same architecture can run end to end without paid services:

| Component | Paid default | Free alternative |
|---|---|---|
| LLM | Claude or OpenAI API | Ollama running Llama 3 or Mistral locally |
| Embeddings | Hosted embeddings API | sentence-transformers (local) |
| Vector store | Pinecone | Chroma, Qdrant free tier, or pgvector |
| Structured database | Managed Postgres | Supabase or MongoDB Atlas free tier |
| Queue | Managed Redis | Upstash free tier |
| Hosting | Paid cloud | Render or Railway free tier, or local |
| CI/CD | n/a | GitHub Actions free minutes |

Running a local model also makes the system fully self-hosted, with no external API dependency.

### Ticket Data Sources

1. **Public datasets:** labeled customer-support ticket datasets from Kaggle and Hugging Face for bulk testing data.
2. **The project's own `POST /api/tickets` endpoint:** the live ticket-submission feature.
3. **Synthetic edge cases:** LLM-generated ambiguous, angry and injection-attempt tickets for testing confidence thresholds and guardrails.
4. **GitHub Issues API:** optionally anchoring the project to real, messy open-source issues instead of generic tickets.
