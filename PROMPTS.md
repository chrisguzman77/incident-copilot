# AI Prompts Used

This document records the AI prompts used during the development of the Incident Response Copilot.
AI-assisted coding was done via **Claude (Anthropic)** throughout the project.

---

## 1. Project Planning & Architecture

**Prompt:**
> Can you give me a step by step guide on how to build this project from the github repo to the finished product: Build the Incident Response Copilot. It is relevant to my background, impressive but still manageable, clearly AI-powered, easy to demo, and naturally uses all required Cloudflare pieces. For the repo, keep it focused: `apps/web` for Pages/React frontend, `workers/api` for Worker routes, `workers/state` for Durable Object logic, `workflows/` for postmortem pipeline, `README.md` with architecture diagram, screenshots, and deployment steps. In the README, explicitly say how the app satisfies the assignment: LLM (Workers AI / Llama 3.3), Workflow/coordination (Workflows + Durable Objects), User input (chat UI on Pages), Memory/state (Durable Objects, optionally D1/Vectorize).

**What it produced:** A full 8-phase build guide covering repo structure, Durable Object implementation, API Worker with LLM integration, Workflow for postmortem generation, React frontend, and deployment steps.

---

## 2. Durable Object — Incident Session State

**Prompt:** (Part of the build guide)
> Phase 2: The Durable Object — Incident Session State. This is the brain of your app. Every incident gets its own Durable Object instance that holds conversation history, current hypothesis, confirmed findings, and the action checklist.

**What it produced:** The `IncidentSession` Durable Object class with HTTP API for state management (`/state`, `/message`, `/update`, `/reset` endpoints) and the `IncidentState` TypeScript interface.

---

## 3. API Worker — Routes and LLM Integration

**Prompt:** (Part of the build guide)
> Phase 3: The API Worker — Routes and LLM Calls. This is the entry point for all API requests from the frontend. It handles routing, calls Workers AI, and delegates state to the Durable Object.

**What it produced:** The main API Worker with routes for `/api/chat`, `/api/state`, `/api/postmortem`, `/api/postmortem/status`, and `/api/action/toggle`. Includes the system prompt for incident analysis, structured JSON extraction from LLM responses, and CORS handling.

---

## 4. LLM System Prompt Design

**Prompt:** (Embedded in the API Worker)
> You are an incident response copilot for software engineers. When a user describes an outage, error, or suspicious behavior, respond with: Likely Diagnosis, Severity Assessment, Prioritized Actions, Confidence, and Follow-up Questions. Also return a JSON block containing title, severity, hypothesis, actions, and findings.

**What it produced:** The system prompt that drives all incident analysis. Designed to return both human-readable analysis and machine-parseable structured data in a single response.

---

## 5. Postmortem Workflow

**Prompt:** (Part of the build guide)
> Phase 4: The Postmortem Workflow. Workflows let you run multi-step, retryable, long-running processes. The postmortem Workflow takes the full incident state, calls the LLM for each section, and assembles the result.

**What it produced:** A 4-step Cloudflare Workflow (`PostmortemWorkflow`) that generates a timeline, root cause analysis, retrospective, and assembles them into a final markdown postmortem document.

---

## 6. React Frontend Components

**Prompt:** (Part of the build guide)
> Phase 5: The Frontend — React on Cloudflare Pages. Use a clean React interface with incident title, chat panel, current hypothesis sidebar, action checklist, and generate postmortem button.

**What it produced:** Six React components: `App.jsx` (main layout with three-column grid), `ChatPanel.jsx`, `HypothesisSidebar.jsx`, `ActionChecklist.jsx`, `PostmortemButton.jsx`, and `IncidentHeader.jsx`. Dark theme CSS with severity color coding.

---

## 7. Debugging — CORS Configuration

**Prompt:**
> I got this error: Access to fetch has been blocked by CORS policy: The 'Access-Control-Allow-Origin' header has a value 'https://incident-copilot.pages.dev' that is not equal to the supplied origin.

**What it produced:** A dynamic CORS origin handler (`getAllowedOrigin`) that accepts any `*.incident-copilot.pages.dev` preview deploy, the production Pages URL, and localhost for development. Also a try/catch wrapper to ensure CORS headers are present even on error responses.

---

## 8. Debugging — Durable Object Migration

**Prompt:**
> I got this error: In order to use Durable Objects with a free plan, you must create a namespace using a `new_sqlite_classes` migration.

**What it produced:** Fix to change `new_classes` to `new_sqlite_classes` in the Durable Object migration config, as required by Cloudflare's free tier.

---

## 9. Debugging — LLM Token Limit

**Prompt:**
> The JSON block is getting cut off and the sidebar never updates.

**What it produced:** Added `max_tokens: 2048` to the Workers AI call to ensure the full structured JSON response is generated.

---

## 10. Architecture Diagram

**Prompt:**
> Can you write a .md for the README for the architecture diagram?

**What it produced:** Two Mermaid diagrams — a system architecture flowchart showing all Cloudflare services and their connections, and a sequence diagram showing the data flow for chat and postmortem generation. Both render natively on GitHub.

---

## Summary

AI assistance was used for:
- Initial project architecture and build planning
- Scaffolding all code files (Durable Object, API Worker, Workflow, React frontend)
- System prompt engineering for the incident analysis LLM
- Debugging deployment issues (CORS, migrations, token limits)
- Documentation and architecture diagrams

All code was reviewed, understood, and modified by hand during development. The AI provided the starting scaffolding and debugging guidance, while implementation decisions, integration, and deployment were done manually.