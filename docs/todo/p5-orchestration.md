# TODO: P5 — Orchestration

Loop predicates, todo queue, triggers scaffold, and budget middleware are in place.
Remaining work is wiring triggers to the LangGraph runtime and sleep-time consolidation.

## `src/jarvis/orchestration/sleep.py`

- [ ] Wire up `memory.consolidation.consolidate()` call with session messages
- [ ] Wire up semantic fact extraction from the session
- [ ] Add a configurable flag to skip sleep tasks in dev/test mode
- [ ] Log what was consolidated (count of facts, episodes, skills stored)

## `src/jarvis/orchestration/triggers.py` (scaffolded)

- [x] File created with Slack Events API, cron, and approval-response entry points
- [ ] Wire `_run_agent()` to LangGraph SDK (`client.runs.create(...)`)
- [ ] Wire `handle_approval_response()` to resume paused HITL runs via runtime API
- [ ] **Webhook trigger** — HTTP endpoint that starts a graph run
  - `POST /trigger` with a JSON payload kicks off an agent run
  - Useful for: "when calendar event is added", "when email arrives"
- [ ] **Cron trigger** — use LangGraph Cron or external scheduler hitting the runs API
- [ ] **Event stream trigger** — subscribe to a queue or change stream (future)

## `agents/personal/graph.py` (thin entry point)

- [x] The DeepAgents harness IS the graph (ADR 0001 — collapsed; no outer graph)
- [x] One checkpointer (`agent.py` `_CHECKPOINTER`); HITL pause/resume is native
- [ ] Swap `MemorySaver` → durable saver for non-dev (Sqlite local / Postgres prod) — see P10

## `agents/personal/agent.py` (the harness)

- [x] `build_personal_graph()` wires the harness + Jarvis middleware
- [x] HITL interrupt bridged to approval inbox via `interaction/hitl.py`
- [ ] `ContextLoader.before_agent` → load semantic memory into context (P2/P3)
- [ ] Register custom Jarvis tools via `_langchain_tools()` (BaseTool → LangChain adapter)

## `src/jarvis/orchestration/middleware.py`

- [x] `BudgetGuard` — step cap via `before_model` hook
- [ ] `after_model` hook — token usage → `estimated_cost_usd` (see `docs/todo/p9-guardrails-safety.md`)

## `src/jarvis/orchestration/loop.py`

- [x] `budget_exceeded()` — used by `BudgetGuard` middleware; `should_continue()` — reference predicate (not wired into the graph)
- [ ] Consider removing `should_continue()` if nothing ever needs a hand-rolled loop

## Tests

- [x] `tests/unit/test_orchestration.py` — should_continue, todo queue, BudgetGuard step cap
- [ ] `tests/unit/test_orchestration.py` — cost limit enforcement (blocked on cost wiring)
- [ ] `tests/integration/test_graph_e2e.py` — end-to-end graph run with a simple task
