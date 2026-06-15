# Done: Repo Skeleton & Infrastructure

## What was built

The full skeleton of the Jarvis repo — every pillar has a module, the entry point
graph is wired, and the developer toolchain is in place. Nothing is production-ready
yet, but every seam is defined and documented.

Key structural decision: **ADR 0001** — one agent loop AND one graph (the DeepAgents
harness IS the registered graph; the early outer lifecycle graph was collapsed),
`integrations/` ring for shared transport, gateways scoped to egress only. See
`docs/architecture/decisions/0001-loop-channels-gateways-integrations.md`.

---

## Core infrastructure (`src/jarvis/core/`)

- **`config.py`** — Pydantic-Settings config reading all values from env vars (12-factor III). Nested groups per pillar (model, memory, guardrails, log, observability, slack). Each nested group declares its own `env_file` so `.env` loads outside the LangGraph runtime.
- **`logging.py`** — Structlog structured logging to stdout only (12-factor XI). JSON in prod, pretty in dev, controlled by `LOG_FORMAT` env var.
- **`state.py`** — `AgentState` base class: messages (append-only), todos, notes, step_count, cost, pending_approvals. 12-factor agents #12: stateless reducer.
- **`errors.py`** — Full error hierarchy with `to_agent_message()` on every type. Covers tool errors, guardrail errors, approval gates, context budget errors.

---

## Orchestration (`src/jarvis/orchestration/`)

- **`loop.py`** — `should_continue()` and `budget_exceeded()` predicates. Reference implementation for "continue or stop?" — no longer called by the outer graph (ADR 0001); budget enforcement lives in middleware instead.
- **`middleware.py`** — `BudgetGuard` agent middleware: enforces step cap before each model call via `before_model` hook. Cost cap predicate exists but `estimated_cost_usd` is not yet incremented (see `docs/todo/p9-guardrails-safety.md`).
- **`todo.py`** — `add_todo`, `complete_todo`, `get_pending`, `clear_done`. Jarvis-level todo queue; in-loop planning is owned by DeepAgents TodoListMiddleware.
- **`sleep.py`** — Stub hook for sleep-time compute. Called by the graph after the agent node completes.
- **`triggers.py`** — Trigger seams: Slack Events API (`handle_slack_event`), cron (`handle_cron`), approval resume (`handle_approval_response`). All delegate to `_run_agent()` which is `NotImplementedError` until LangGraph SDK is wired.

---

## Tools (`src/jarvis/tools/`)

- **`base.py`** — `BaseTool` abstract class + `ToolResult` dataclass. Enforces ACI patterns: honest status, structured output, actionable errors, idempotency flag.
- **`registry.py`** — `register()`, `get()`, `list_tools()`, `get_tool_descriptions()`. Dynamic tool loading — don't preload everything (P2).

---

## Human Interaction (`src/jarvis/interaction/`)

- **`inbox.py`** — `request_approval()` / `process_approval()`. Approval requests live in `AgentState.pending_approvals`; decisions feed back via `notes` + `messages`.
- **`hitl.py`** — Bridge between DeepAgents HITL interrupts and the inbox: `approvals_from_interrupt()`, `resume_payload()`.
- **`channels/slack.py`** — P6 human surface: post replies, surface approval inbox over Slack. Transport from `integrations/slack/client.py`.

---

## Integrations (`src/jarvis/integrations/`)

Shared transport adapters — dumb SDK wiring, no agent logic. Used by multiple pillars without sideways imports.

- **`slack/client.py`** — `SlackClient`, `SlackEvent`. Signature verify, parse, post. Stubs raise `NotImplementedError` until `slack-sdk` is wired (`pip install -e ".[channels]"`).

---

## Guardrails (`src/jarvis/guardrails/`)

- **`injection.py`** — Regex-based prompt injection scanner + `sanitize()`. Detects "ignore instructions", "act as if", etc. in retrieved content.
- **`middleware.py`** — `InjectionGuard` agent middleware: sanitizes tool output before it reaches the model.
- **`output.py`** — Stub output filter (PII, policy violations — ready for implementation).

---

## Memory interfaces (`src/jarvis/memory/`)

- **`episodic.py`** — `Episode` dataclass + `EpisodicMemory` abstract interface (store, retrieve, forget).
- **`semantic.py`** — `Fact` dataclass + `SemanticMemory` abstract interface (store, retrieve, get_by_subject, delete).
- **`procedural.py`** — `Skill` dataclass + `ProceduralMemory` abstract interface (store, retrieve, record_success).
- **`consolidation.py`** — Stub consolidation hook. Called during sleep-time compute to compress session → long-term memory.

---

## Agent entry point (`src/jarvis/agents/personal/`)

> Updated: the early outer-graph design (`nodes.py`, `state.py`, a linear
> `load_context → agent → sleep` graph) was collapsed into the single harness — see ADR 0001.

- **`agent.py`** — the DeepAgents harness IS the graph: `build_personal_graph()` wires
  `create_deep_agent` with `ContextLoader`, `BudgetGuard`, `InjectionGuard`, `interrupt_on`
  for `execute`, `JarvisDeepState`, and one `_CHECKPOINTER` for HITL pause/resume.
  `format_personal_system_prompt()` fills `{user_context}`.
- **`graph.py`** — thin entry point: configures logging/tracing and exposes the compiled
  harness as `graph`. Registered in `langgraph.json`.

---

## Gateway (`src/jarvis/gateway/`)

- **`__init__.py`** — Scoped to egress control (model + MCP/tool gateway) only. Explicit "what a gateway is NOT" documentation. Implementation stubs for future work.

---

## Prompts

- **`prompts/personal/system.md`** — Version-controlled system prompt for the personal assistant. Loaded at runtime via `model/prompts.py`; `{user_context}` filled by `format_personal_system_prompt()`.

---

## Model layer (`src/jarvis/model/`)

- **`router.py`** — `TaskType` enum + `get_model_for_task()`. Dispatches to planner vs worker model based on task type.
- **`prompts.py`** — `load_prompt()` with `@lru_cache`. Loads `.md` files from `prompts/` directory.

---

## Context (`src/jarvis/context/`)

- **`compaction.py`** — `should_compact()` + `compact_messages()` heuristics. **Not wired** —
  compaction is the harness's job (SummarizationMiddleware); kept only as a reference for a
  custom policy. Context *loading* is `ContextLoader` middleware in `agent.py`.

---

## Observability / Improvement stubs

- **`observability/tracing.py`** — LangSmith project setup when `LANGSMITH_API_KEY` is set.
- **`observability/metrics.py`** — `RunMetrics` dataclass stub.
- **`improvement/feedback.py`** — `Feedback` dataclass stub for P8 improvement loop.

---

## Project config

- **`pyproject.toml`** — Dependencies, dev extras, ruff, mypy, pytest config. **Python ≥ 3.13**.
- **`langgraph.json`** — Registers `jarvis` graph at `agents/personal/graph.py:graph`.
- **`.env.example`** — All config vars documented with defaults and comments.
- **`Makefile`** — `make install`, `dev`, `dev-ui`, `test`, `lint`, `format`, `typecheck`, `check`, `eval`, `build`, `deploy`, `seed-memory`, `clear-memory`, `clean`.

---

## Scripts

- **`scripts/seed_memory.py`** — One-off memory seeding (12-factor XII). Run via `make seed-memory`.
- **`scripts/clear_memory.py`** — Wipe stored memories (destructive). Run via `make clear-memory` (prompts for confirmation).
- **`scripts/run_evals.py`** — Eval suite runner. Run via `make eval`.

---

## Tests

Unit tests in `tests/unit/` (90+ passing; coverage threshold 70% not yet met — stub modules at 0%):

- **`test_core.py`** — Config, errors, AgentState, TodoItem
- **`test_tools.py`** — BaseTool, ToolResult, registry
- **`test_orchestration.py`** — `should_continue`, todo queue, `BudgetGuard` middleware
- **`test_model.py`** — Prompt loading, model config (scaffolded)
- **`test_context.py`** — Budget errors, notes pattern (scaffolded)
- **`test_memory.py`** — Memory module imports (scaffolded)
- **`test_guardrails.py`** — Injection detection/sanitization, limit errors, `InjectionGuard`
- **`test_interaction.py`** — Approval inbox, HITL bridge (`approvals_from_interrupt`, `resume_payload`)

Integration tests: `tests/integration/README.md` only (not started).

---

## Developer docs (`docs/developer/`)

- **`getting-started.md`** — First-time setup, common commands.
- **`folder-structure.md`** — Full tree with pillar mapping table and dependency rule.
- **`adding-a-tool.md`** — Step-by-step guide with ACI checklist and test examples.
- **`principles.md`** — 12-factor app + 12-factor agents tables mapping each factor to the codebase.
- **`testing.md`** — Three-tier test strategy, fixtures, coverage notes.

---

## Architecture docs (`docs/architecture/`)

- **`agent/`** — All 16 pillar + cross-cutting docs (00-overview through 15-buzzwords-glossary).
- **`decisions/0001-loop-channels-gateways-integrations.md`** — Accepted ADR for loop shape, triggers, gateways, integrations ring.

---

## Research

- **NotebookLM research prompts** — One targeted Fast Research query per pillar (P0–CX-Cost). See `docs/todo/notebooklm-research.md`.
- **P0 notebook** — Created in NotebookLM with source + Fast Research complete.
- **P1 notebook** — Created with source uploaded; research query submitted.
