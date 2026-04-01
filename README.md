# Incident Response Copilot

An AI-powered incident response tool that helps engineers diagnose outages,
track actions, and generate postmortems — built entirely on Cloudflare.

![Architecture Diagram](./docs/architecture.png)

## How It Works

1. Paste an error log, describe symptoms, or share a system alert.
2. The copilot analyzes the input and returns a diagnosis, severity,
   and prioritized action plan.
3. As you investigate, the copilot remembers prior context and refines
   its hypothesis.
4. When the incident is resolved, generate a structured postmortem
   with one click.

## Architecture

| Component | Cloudflare Service | Purpose |
|---|---|---|
| LLM | Workers AI (Llama 3.3 70B) | Diagnosis, analysis, postmortem writing |
| Coordination | Durable Objects | Per-incident stateful session management |
| Workflows | Cloudflare Workflows | Multi-step postmortem generation pipeline |
| Frontend | Cloudflare Pages + React | Chat UI, sidebar, action tracking |
| Memory (short-term) | Durable Object storage | Conversation history, hypothesis, actions |
| Memory (long-term) | D1 + Vectorize (optional) | Incident history, runbook retrieval |

## Assignment Mapping

| Requirement | Implementation |
|---|---|
| **LLM** | Workers AI running `@cf/meta/llama-3.3-70b-instruct-fp8-fast` for all inference |
| **Workflow / Coordination** | Cloudflare Workflows for postmortem pipeline; Durable Objects for session orchestration |
| **User Input** | Chat-based React UI on Cloudflare Pages |
| **Memory / State** | Durable Objects persist conversation, hypothesis, actions, and findings per session |

## Project Structure
apps/web/          → React frontend (Cloudflare Pages)
workers/api/       → API Worker (routes, LLM calls, orchestration)
workers/state/     → Durable Object (incident session state)
workflows/postmortem/ → Workflow (multi-step postmortem generation)

## Local Development
```bash
# Install dependencies
npm install

# Run the API worker locally
cd workers/api && npx wrangler dev

# Run the frontend locally
cd apps/web && npm run dev
```

## Deployment
```bash
# Deploy in order: state → workflow → api → frontend
cd workers/state && npx wrangler deploy
cd workflows/postmortem && npx wrangler deploy
cd workers/api && npx wrangler deploy
cd apps/web && npm run build && npx wrangler pages deploy dist --project-name incident-copilot
```

## Screenshots
<img width="2864" height="1632" alt="shot2" src="https://github.com/user-attachments/assets/15af369e-3137-48b7-b361-d4202af5ada9" />

<img width="2864" height="1632" alt="incident-reponse" src="https://github.com/user-attachments/assets/2e10f839-7248-4c28-b0c7-612afc5d435c" />

flowchart TB
    subgraph User["👤 User"]
        Browser["Browser"]
    end

    subgraph Pages["Cloudflare Pages"]
        Frontend["React Frontend\n• Chat Panel\n• Hypothesis Sidebar\n• Action Checklist\n• Postmortem Viewer"]
    end

    subgraph Workers["Cloudflare Workers"]
        API["API Worker\n(incident-api)\n• /api/chat\n• /api/state\n• /api/postmortem\n• /api/action/toggle"]
    end

    subgraph AI["Workers AI"]
        LLM["Llama 3.3 70B\nInstruct FP8 Fast"]
    end

    subgraph DO["Durable Objects"]
        Session["Incident Session\n(per-incident state)"]
        Storage[("DO Storage\n• Messages\n• Hypothesis\n• Actions\n• Findings\n• Severity")]
    end

    subgraph WF["Cloudflare Workflows"]
        Postmortem["Postmortem Workflow"]
        Step1["Step 1:\nGenerate Timeline"]
        Step2["Step 2:\nRoot Cause Analysis"]
        Step3["Step 3:\nRetrospective"]
        Step4["Step 4:\nAssemble Document"]
    end

    subgraph Optional["Optional — Stretch Goals"]
        D1[("D1 Database\nIncident History")]
        Vectorize[("Vectorize\nRunbook Retrieval")]
    end

    Browser -->|"HTTPS"| Frontend
    Frontend -->|"POST /api/chat\nGET /api/state"| API
    API -->|"Diagnosis &\nAnalysis"| LLM
    API -->|"Read/Write\nSession State"| Session
    Session --- Storage
    API -->|"Trigger"| Postmortem
    Postmortem --> Step1 --> Step2 --> Step3 --> Step4
    Step1 -->|"LLM Call"| LLM
    Step2 -->|"LLM Call"| LLM
    Step3 -->|"LLM Call"| LLM
    API -.->|"Store resolved\nincidents"| D1
    API -.->|"Retrieve relevant\nrunbook chunks"| Vectorize

    style User fill:#1e1e2e,stroke:#6c7086,color:#cdd6f4
    style Pages fill:#1e3a5f,stroke:#3b82f6,color:#e1e4ed
    style Workers fill:#1e3a5f,stroke:#3b82f6,color:#e1e4ed
    style AI fill:#2d1b4e,stroke:#a855f7,color:#e1e4ed
    style DO fill:#1b3d2f,stroke:#22c55e,color:#e1e4ed
    style WF fill:#3d2f1b,stroke:#f97316,color:#e1e4ed
    style Optional fill:#1a1d27,stroke:#2a2d3a,color:#8b8fa3,stroke-dasharray: 5 5
