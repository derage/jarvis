# ADR 0001 — Loop shape, channels, gateways, and the integrations ring

**Status:** Accepted · **Date:** 2026-06-15

Four related structural decisions made while hardening the skeleton. They share
one theme: **classify each boundary by what's on the other side, and keep one
concern in one place.** Grounded in Anthropic's *Building Effective Agents*,
the 12-Factor Agents manifesto, Letta's agent-loop write-ups, and LangChain/
LangGraph + DeepAgents docs.

---

## 1. One agent loop — and one graph (the harness IS the graph)

**Context.** The skeleton ran an outer LangGraph continuation loop
(`should_continue` re-invoking `agent`) that would wrap a `create_agent` /
`create_deep_agent` call which has its *own* reason-act loop — two loops both
deciding "keep going?".

**Decision.** Exactly **one** continuation loop: the agent's — and (revised) **one
graph**: the DeepAgents harness IS the registered graph, no outer wrapper. An
earlier iteration wrapped the harness in an outer linear graph
(`load_context → agent → sleep`); that created a second checkpointer and a separate
resume path, so it was collapsed. The session lifecycle is now **middleware**
(`ContextLoader`) + a **cron sibling** (sleep-time). We still "own our control flow"
(12-factor #8) — via middleware control *points*, not a second graph.

**Consequences (done).**
- **One graph, one checkpointer.** `graph.py` is a thin entry point exposing the
  compiled harness (`agent.build_personal_graph()`); the outer `StateGraph`, its
  nodes (`nodes.py`), and `state.py` (`PersonalAgentState`) were removed.
- **Harness = DeepAgents** (`create_deep_agent`), chosen because its
  TodoListMiddleware fits the multi-step tasks.
- **Lifecycle as middleware/sibling:** `load_context` → `ContextLoader.before_agent`
  (agent.py); sleep-time → a cron sibling (`orchestration/sleep.py` via
  `triggers.handle_cron`).
- Budget cap = `BudgetGuard` (`before_model`); injection defense = `InjectionGuard`
  (`wrap_tool_call`); approvals = `interrupt_on`. All attached in `build_personal_graph`.
- **P9 tool blast-radius:** the harness ships built-in tools (write_todos, a
  *virtual* filesystem, task, `execute`). `execute` is gated behind `interrupt_on`;
  the virtual-FS tools (sandboxed scratchpad) run freely.
- **HITL pause/resume is NATIVE:** a gated tool interrupts on its own `thread_id`;
  the runtime surfaces it (bridged to inbox/Slack via
  `interaction/hitl.approvals_from_interrupt`) and resumes with
  `Command(resume=hitl.resume_payload(...))` on the SAME thread — one checkpointer,
  no separate resume path. `triggers.handle_approval_response` is the inbound seam
  (still a stub). *Live cycle needs an API key.*
- **Durability:** the single checkpointer is in-process today; because the Slack
  approval round-trip is async (possibly cross-process), **production requires a
  durable shared checkpointer** (Sqlite dev / Postgres prod). Tracked in
  docs/todo/p10-runtime-deployment.md.
- `should_continue` remains the P5 reference predicate (not called by the graph).
  `orchestration/todo.py` as a *loop driver* is superseded by the agent's todo
  middleware.

> Anthropic: "add complexity *only* when it demonstrably improves outcomes."
> Letta v1: the field is converging on simpler, **single** loops, not nesting.

## 2. "Keep working on a schedule" = stop + re-trigger, not a spinning loop

**Context.** We want outstanding work to get done on a schedule, which felt like
it needed a long-running loop.

**Decision.** A run **stops** when its in-loop todos drain (disposable, no idle
cost). Outstanding work lives in a **durable backlog** (a backing service:
LangGraph `Store` / DB / an external task list), and a **scheduler** (cron /
queue) re-triggers a *new* run to drain it. "Keep working" is a system-level
property (trigger + backlog), not a process-level one (spinning loop).

**Consequences.** Three distinct "todo" concepts, never conflated:
- **In-loop todos** — this task's plan; thread state; empty → run ends.
- **Durable backlog** — outstanding work across runs; a backing service.
- **Triggers** — how runs start; `orchestration/triggers.py`.

Sleep-time compute (`orchestration/sleep.py`) grooms the backlog while idle — a
*sibling* process over shared state (the Letta pattern), not a wrapper loop.

## 3. `gateway/` = egress control substrate (models + MCP/tools) **only**

**Context.** Where does a Slack integration go? `gateway/` looked plausible.

**Decision.** A gateway is **egress control for the agent's capability
dependencies** — the models and tool/MCP servers it calls *out* to, under policy
(routing, fallback, authz, rate-limit, audit). It is **not** ingress, channels,
or transport. Slack is none of those things, so it does **not** belong in
`gateway/`.

**Consequences.** `gateway/` stays scoped to model gateway + MCP/tool gateway.
The `13-gateways.md` doc and `gateway/__init__.py` now carry an explicit "what a
gateway is NOT" section + litmus test.

## 4. `integrations/` ring for shared external-service transport

**Context.** Slack is used by two pillars — `orchestration/triggers.py`
(inbound) and `interaction/channels/slack.py` (human surface). Sharing the
client by importing one pillar from the other violates "pillars never import
pillars" and risks duplicating SDK/auth code.

**Decision.** Add an `integrations/` ring for **transport-only** adapters (SDK
wiring, auth, (de)serialization, retries — zero agent logic), *inward* of the
pillars:

```
agents/ → pillars/ → integrations/ → core/
```

A pillar depending on `integrations/` is **inward**, not sideways — the rule
holds. Every external service lands the same way: `integrations/{slack,gmail,
calendar,…}/`. This is the ports-and-adapters boundary: pillars declare *what*
they need; adapters implement *how*.

**Consequences.**
- New files: `integrations/slack/client.py` (transport), with usages in
  `orchestration/triggers.py` (ingress) and `interaction/channels/slack.py` (P6).
- `integrations/` ≠ `gateway/`: transport (dumb) vs policy (smart). A gateway can
  sit in front of an adapter; a channel needs only the adapter.
- Secrets stay in `core/config.py` + `.env.example` (12-factor III).

---

## Net dependency shape

```
agents/ ── invoke runtime ──▶ (LangGraph server)
   │
   └─▶ pillars ──▶ integrations/ ──▶ core/
       (model, memory, tools, orchestration, interaction, guardrails, …)

gateway/  = egress policy over models + MCP/tools   (control substrate)
retrieval/ = data substrate
integrations/ = transport adapters (Slack, Gmail, …)  (shared, dumb)
channels (interaction/channels) + triggers = the human/event edge
```

Triggers invoke the agent through the **runtime boundary** (LangGraph API/SDK),
never by importing `agents/` — which is what makes "trigger from anywhere"
(12-factor #11) work and keeps the dependency direction clean.
