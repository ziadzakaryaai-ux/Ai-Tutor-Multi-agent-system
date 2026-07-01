# 🎓 Learning Copilot

**A Multi-Agent Adaptive Learning System that optimizes for deep understanding — not just correct answers.**

[![Python](https://img.shields.io/badge/python-3.11%2B-blue)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-async-009688)](https://fastapi.tiangolo.com/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Status](https://img.shields.io/badge/status-MVP-orange)]()
[![Built with Azure](https://img.shields.io/badge/built%20with-Microsoft%20Foundry-0078D4)]()

> 🏆 Built during Microsoft's **Agentic AI Hackathon** (Microsoft Foundry track), June 2026.

---

## Overview

Most AI tutoring tools optimize for one thing: giving the student a correct answer as fast as possible. **Learning Copilot does the opposite.**

It's a multi-agent backend that treats a correct final answer as *weak evidence* of learning — and instead evaluates the student's **reasoning process**, step by step, to distinguish genuine conceptual understanding from memorized pattern-matching. It builds a live, evolving model of what a student actually knows, diagnoses *why* they got something wrong (not just *that* they did), adapts its teaching strategy accordingly, and verifies understanding through Socratic questioning and transfer problems before ever assuming mastery.

**The problem it solves:** traditional tutoring systems (and most LLM chatbots) reward students for reaching the right output, which silently reinforces pattern-matching over understanding. A student can memorize "subtract, then divide" without ever grasping *why* — and that gap surfaces later, on a slightly different problem, when the pattern breaks. Learning Copilot is designed to catch that gap immediately.

**Why it matters:** this is infrastructure for *durable* learning — deep understanding, long-term retention, and transfer — rather than short-term task completion. The routing logic is a deterministic, auditable state machine (not an LLM making unpredictable pedagogical decisions), which makes the system's behavior explainable and testable — a requirement for any real educational product.

---

## Features

- 🧠 **Step-level reasoning evaluation** — analyzes every step of a student's solution, not just the final answer, and identifies the *first* incorrect step (the root cause of any downstream error).
- 🏷️ **Five-type mistake taxonomy** — classifies errors as `conceptual`, `procedural`, `careless`, `knowledge_gap`, or `confidence_mismatch`, each triggering a different teaching response.
- 🔄 **Adaptive teaching strategies** — deterministically selects between direct explanation, Socratic questioning, worked examples, analogies, and progressive hints based on the student's actual state.
- 📊 **Dynamic student model** — tracks mastery, confidence, calibration error, misconceptions, and memorization risk per concept.
- 🕵️ **Memorization vs. understanding detection** — uses transfer problems and Socratic probing to reveal pattern-matching that a correct answer would otherwise hide.
- ⏳ **Ebbinghaus-inspired spaced repetition** — models knowledge decay over time and schedules reviews before concepts are forgotten.
- 🎯 **Personalized problem generation** — targets the student's specific weaknesses at a calibrated difficulty (zone of proximal development).
- 🔍 **Retrieval-augmented grounding** — pulls curriculum material for genuine knowledge gaps only, avoiding retrieval noise for other mistake types.
- ☁️ **Dual LLM backend** — runs on Azure OpenAI or the Anthropic API through a single unified client, with zero agent-code changes required to switch.
- ✅ **100 passing tests** — runnable with no API key required.

---

## Architecture

The system runs a **deterministic Teach → Test → Evaluate → Diagnose → Adapt** loop, orchestrated by a plain Python state machine — deliberately **not** an LLM. Routing decisions (which phase comes next, which teaching strategy to use) are algorithmic and auditable; LLM calls are used only to *generate content* within a strategy the orchestrator has already chosen.

```
                         ┌─────────────────────┐
                         │   Orchestrator       │
                         │ (deterministic FSM)  │
                         └──────────┬───────────┘
                                    │ routes to
        ┌──────────────┬───────────┼───────────┬──────────────┐
        ▼              ▼           ▼           ▼              ▼
  ┌──────────┐  ┌─────────────┐ ┌────────┐ ┌──────────┐ ┌────────────┐
  │  Tutor   │  │Step Evaluator│ │Socratic│ │Evaluation│ │ Assessment │
  │  Agent   │  │    Agent     │ │ Agent  │ │  Agent   │ │   Agent    │
  └──────────┘  └─────────────┘ └────────┘ └──────────┘ └────────────┘
        │              │           │           │              │
        └──────────────┴───────────┴───────────┴──────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Tool Agent          │
                         │ (curriculum retrieval,│
                         │  invoked selectively) │
                         └──────────┬───────────┘
                                    ▼
                         ┌─────────────────────┐
                         │   Storage Layer       │
                         │ In-Memory / JSON /     │
                         │ Azure Cosmos DB        │
                         └─────────────────────┘
```

**Workflow:**
1. **Teach** — the Tutor Agent explains a concept using a strategy chosen deterministically from the student's profile.
2. **Test** — the student submits a step-by-step solution.
3. **Evaluate** — the Step Evaluator traces the reasoning, finds the first incorrect step, and classifies the mistake type. The Student Model updates mastery using ELO-style differentiated penalties.
4. **Diagnose** — the Evaluation Agent synthesizes signals (understanding, confidence, retention, focus) into a recommended action.
5. **Adapt** — the Tutor Agent re-teaches using a different strategy matched to the diagnosed mistake type, or the Socratic Agent challenges the student to verify the understanding is real, not memorized.
6. **Retest** — the loop repeats until stable understanding is demonstrated across contexts.

Azure is used strictly as an **infrastructure layer** (LLM backend, storage, search, deployment) — agent logic, prompts, and the learning model itself are backend-independent.

---

## Tech Stack

**Core**
- Python 3.11+
- FastAPI (async REST API)
- Pydantic v2 (typed inter-agent contracts)
- pytest (100-test suite, no API key required)

**LLM Backends** (unified via a single client abstraction)
- Anthropic API (Claude models)
- Azure OpenAI

**Azure Infrastructure**
- Azure Container Apps (deployment)
- Azure Cosmos DB (production storage backend)
- Azure AI Search (curriculum retrieval for the Tool Agent)
- Bicep (Infrastructure as Code)

**Frontend (reference implementation)**
- React (single-component adaptive tutoring UI)

**Tooling**
- Docker / docker-compose
- GitHub Actions (CI: tests, lint, API smoke test)
- Ruff (linting)

---

## Project Structure

```
learning-copilot/
├── models/
│   └── schemas.py            # Pydantic contracts shared by all agents
├── core/
│   ├── orchestrator.py       # Deterministic state machine
│   ├── llm_client.py         # Azure / Anthropic unified client
│   ├── storage.py            # InMemory / JSON / Cosmos DB backends
│   ├── spaced_repetition.py  # Ebbinghaus retention scheduler
│   └── logging.py            # Structured JSON logging + middleware
├── agents/
│   ├── step_evaluator.py     # Reasoning-trace evaluation
│   ├── tutor.py               # Explanation generation + strategy selection
│   ├── socratic.py            # Probing / transfer questions
│   ├── evaluator.py           # Holistic understanding evaluation
│   ├── assessment.py          # Targeted problem generation
│   └── tool_agent.py          # Curriculum retrieval (Azure AI Search)
├── api/
│   └── main.py                # FastAPI application (10 endpoints)
├── tests/                     # 100 tests — conftest fixtures + unit/integration
├── azure/                     # Bicep IaC, deployment scripts
├── frontend/
│   └── LearningCopilotUI.jsx  # Reference React chat UI
├── demo.py                    # 3-minute CLI demo script
├── requirements.txt
├── Makefile
└── README.md
```

---

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/ziadzakaryaai-ux/learning-copilot.git
cd learning-copilot

# 2. Install dependencies
pip install -r requirements.txt

# 3. Set your LLM backend credentials (choose ONE)

# Option A — Anthropic (simplest for local dev)
export ANTHROPIC_API_KEY="sk-ant-..."

# Option B — Azure OpenAI
export AZURE_OPENAI_ENDPOINT="https://<your-resource>.openai.azure.com/"
export AZURE_OPENAI_API_KEY="<your-key>"

# 4. Run the test suite (no API key required)
pytest tests/ -q
```

---

## Usage

**Start the API server:**

```bash
uvicorn api.main:app --reload --port 8000
```

Interactive API docs: `http://localhost:8000/docs`

**Run the 3-minute CLI demo:**

```bash
python demo.py linear_equations conceptual
```

This walks through: student profile creation → initial teaching → a deliberately wrong-step attempt → Step Evaluator diagnosis → student model update → adaptive re-teaching → Socratic verification.

**Run the reference React UI:**

The `frontend/LearningCopilotUI.jsx` file is a self-contained React component (no external component library required beyond `useState`/`useRef`/`useEffect`).

```bash
# Inside any Vite or Create React App project:
npm create vite@latest learning-copilot-ui -- --template react
cd learning-copilot-ui
cp ../learning-copilot/frontend/LearningCopilotUI.jsx src/App.jsx
npm install
npm run dev
```

Make sure the API server is running at `http://localhost:8000` (the default `API_BASE` in the component) — update it in `App.jsx` if your API runs elsewhere.

---

## Example Workflow

1. **Start a session:** `POST /session/start` with a topic (e.g. `linear_equations`) and student name.
2. **Learn:** the Tutor Agent teaches the concept using the strategy best suited to a first exposure.
3. **Attempt a problem:** the student submits step-by-step reasoning via `POST /session/{id}/respond`.
4. **Get diagnosed, not just graded:** the Step Evaluator pinpoints the *first* wrong step and its mistake type — e.g. "procedural: right idea, wrong execution" rather than a generic ✗.
5. **Watch the system adapt:** if the mistake is conceptual, the Tutor switches to an analogy-based explanation; if it's careless, a progressive hint; if memorization is suspected, a Socratic transfer question.
6. **Get challenged before being told "you've got it":** once the student answers correctly, the Socratic Agent verifies the win wasn't luck or memorization — e.g. asking them to explain *why*, or apply the concept in a completely different context.
7. **Check the knowledge map:** `GET /session/{id}/knowledge-map` shows mastery, calibration status, and memorization-risk flags per concept, sorted by what needs attention most.

---

## Future Improvements

This repository is an **MVP** (see [Project Status](#-project-status--mvp-disclaimer) below). Planned roadmap:

- 🗺️ Curriculum dependency graph (concept prerequisite modeling)
- ⏰ Spaced-repetition scheduling wired directly into the live routing loop
- 🧩 Structured misconception entities (linked to concepts, tracked over time)
- 🛡️ Content-safety/guardrail pass on retrieved curriculum material
- 🧭 Metacognitive coaching layer (reflection on *how* the student learns, not just *what*)
- 📈 Trend-based plateau and calibration-drift detection (not just point-in-time scores)
- 🔐 Authentication, authorization, and multi-tenant data isolation
- 🚀 Full production hardening — see below

---

## 🚧 Project Status — MVP Disclaimer

**This repository is a Minimum Viable Product (MVP)**, built to validate the core concept during Microsoft's Agentic AI Hackathon within a fixed time constraint. The goal of this phase was to prove that step-level reasoning evaluation, deterministic adaptive routing, and memorization detection are viable as the foundation of a real learning system — **not to ship a production-ready platform.**

What this MVP *does* demonstrate solidly:
- A working multi-agent loop with real LLM-backed reasoning evaluation
- A deterministic, auditable orchestrator (a genuine architectural strength, not a shortcut)
- 100 passing tests and Azure IaC for real deployment
- A functioning end-to-end demo, API, and reference UI

What it *intentionally* does not yet include — planned for a future **Production Version**:

| Area | Planned for Production |
|---|---|
| 📈 Scalability | Stateless workers, externalized session state, load-tested throughput |
| 🔐 Auth & Authorization | Multi-tenant auth, role-based access, per-student data isolation |
| 🗄️ Persistent Databases | Fully managed Cosmos DB / Postgres in place of JSON/in-memory storage |
| 🛠️ Error Handling | Retries, timeouts, and circuit breakers on every LLM and network call |
| 📡 Monitoring & Logging | Distributed tracing, per-agent latency/cost dashboards, alerting |
| 🔒 Security | Secrets management, input sanitization, guardrails on retrieved content |
| 🧪 Automated Testing | Expanded integration/load testing, contract testing between agents |
| ⚙️ CI/CD Pipeline | Automated build → test → deploy pipeline with staged environments |
| ☁️ Cloud Deployment | Hardened, autoscaled Azure Container Apps deployment with monitoring |
| ⚡ Performance Optimization | Concurrent agent execution, caching, reduced write amplification |
| 🎨 UI/UX | A polished, fully productized front-end beyond the reference component |

The MVP is a deliberate, important milestone — not a finished product — and the production version above is part of this project's long-term roadmap.

---

## Contributing

This project started as a solo hackathon build and isn't yet set up for external contributions in a structured way. That said:

- 🍴 Feel free to fork the repository and experiment.
- 🐛 Bug reports and suggestions are welcome via [GitHub Issues](https://github.com/ziadzakaryaai-ux/learning-copilot/issues).
- 🔀 Pull requests may be considered on a case-by-case basis, but there's no formal contribution process (CONTRIBUTING.md, style guide, etc.) at this stage — that will come with the production version.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Acknowledgments

- 🙏 **Microsoft Foundry mentors and organizers** of the Agentic AI Hackathon, for their guidance and support throughout the build.
- 🏆 Built as part of Microsoft's **Agentic AI Hackathon** (Microsoft Foundry track), June 2026.
- 🤖 Powered by **Azure OpenAI** and the **Anthropic API**, unified through a single backend-agnostic client abstraction.
- 📚 Pedagogical design informed by established learning-science principles: the Ebbinghaus forgetting curve, Item Response Theory, and mistake taxonomy research distinguishing conceptual, procedural, and careless errors.
