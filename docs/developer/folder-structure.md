# Folder Structure

## Overview

```
jarvis/
├── src/jarvis/              # Main Python package
│   ├── core/                # Shared infra — config, logging, state, errors
│   ├── model/               # P1: model routing, prompt loading
│   ├── context/             # P2: context loading seam (compaction → harness)
│   ├── memory/              # P3: episodic, semantic, procedural memory
│   ├── tools/               # P4: tool registry, base class, built-ins
│   ├── orchestration/       # P5: triggers (#11), budget guard, sleep sibling (loop/todos → harness)
│   ├── interaction/         # P6: approval inbox, channels/ (slack), streaming, handoff
│   ├── observability/       # P7: tracing, metrics, eval suites
│   ├── improvement/         # P8: feedback capture, trace→dataset pipelines
│   ├── guardrails/          # P9: injection defense, input/output filtering
│   ├── retrieval/           # CX: RAG — chunking, embedding, vector store
│   ├── gateway/             # CX: model + MCP gateway (egress control ONLY — not channels)
│   ├── integrations/        # Shared transport adapters (slack, …) — not a pillar
│   ├── agents/              # Entry points — LangGraph graphs
│   │   └── personal/        # The personal assistant agent (DeepAgents harness)
│   │       ├── graph.py     # ← registered in langgraph.json (thin: exposes the harness)
│   │       └── agent.py     # build_personal_graph + Jarvis middleware (ContextLoader, …)
│   └── prompts/             # Version-controlled prompt files (packaged with app)
│       └── personal/
│           └── system.md    # System prompt for the personal agent
├── tests/
│   ├── unit/                # Fast tests, no external calls
│   └── integration/         # Tests that hit real APIs (need .env)
├── scripts/                 # Admin/one-off processes (12-factor XII)
│   ├── seed_memory.py
│   └── run_evals.py
├── docs/
│   ├── developer/           # ← you are here
│   └── architecture/agent/  # Pillar design docs (the blueprints)
├── .env.example             # Config template — copy to .env and fill in keys
├── pyproject.toml           # Dependencies and tool config
├── langgraph.json           # LangGraph entry point registration
└── Makefile                 # All common dev tasks
```

## The dependency rule

**Always inward, never sideways.** Agents import from pillars; pillars import from `integrations/` and `core/`; pillars never import from other pillars.

```
agents/ → pillars (model, memory, tools, …) → integrations/ → core/
```

Anything two pillars share — e.g. a Slack client used by both `orchestration/triggers.py` and `interaction/channels/slack.py` — lives in the inward `integrations/` ring, never in a sibling pillar. This means you can add, swap, or delete any pillar without rewriting siblings.

## One pillar = one directory

Each directory under `src/jarvis/` maps to one architecture pillar. The correspondence:

| Directory | Pillar | Core question |
|---|---|---|
| `model/` | P1 — Model/Reasoning | Which model, what altitude prompt? |
| `context/` | P2 — Context | Smallest high-signal token set? |
| `memory/` | P3 — Memory | What persists across sessions? |
| `tools/` | P4 — Tools/Actions | Are tools agent-legible? |
| `orchestration/` | P5 — Orchestration | Continue or stop? How? |
| `interaction/` | P6 — Human Interaction | Can Jesse see, steer, and trust it? |
| `observability/` | P7 — Eval & Observability | How do we know it's working? |
| `improvement/` | P8 — Improvement Loop | How does it get better? |
| `guardrails/` | P9 — Guardrails & Safety | What's the blast radius? |
| `retrieval/` | CX — RAG | Data substrate for memory and context |
| `gateway/` | CX — Gateways | Model + MCP/tool **egress** routing/policy (not channels) |
| `integrations/` | (shared infra) | Transport adapters for external services (Slack, …) |
| `core/` | (shared infra) | Config, logging, state, errors |

## Where to put new code

- **New tool** → `src/jarvis/tools/builtin/your_tool.py`, register in `tools/registry.py`
- **New agent** → new folder under `src/jarvis/agents/`, add entry to `langgraph.json`
- **New memory behavior** → `src/jarvis/memory/`
- **New prompt** → `src/jarvis/prompts/<agent-name>/your-prompt.md`, load via `model/prompts.py`
- **New eval case** → `src/jarvis/observability/evals/`
- **New config value** → add to `core/config.py` + `.env.example`
- **New external service** (Slack, Gmail, …) → `src/jarvis/integrations/<service>/` (transport only)
- **New channel** (how the agent talks to a person) → `src/jarvis/interaction/channels/`
- **New trigger** (how a run starts: webhook, cron, queue) → `src/jarvis/orchestration/triggers.py`
