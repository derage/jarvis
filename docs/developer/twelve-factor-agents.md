# The 12-Factor Agents — as Jarvis applies them

Adapted from [12-Factor Agents](https://github.com/humanlayer/12-factor-agents)
(Dex Horthy / HumanLayer). This is the **owned** version: each factor states the
original principle, how Jarvis implements it *today*, and — where we deliberately
deviate — the **adaptation** and why.

The single most important thing to internalize: **Jarvis runs on the DeepAgents
harness, which owns the agent loop.** We don't hand-roll the loop; we own control
*points* as middleware. So read the ownership table first.

## What the harness owns vs. what Jarvis owns

| Concern | Owner | Where |
|---|---|---|
| Reason-act loop | **DeepAgents** | harness |
| Todos / planning | **DeepAgents** (TodoListMiddleware) | harness |
| Context compaction | **DeepAgents** (SummarizationMiddleware) | harness |
| Virtual filesystem / scratchpad | **DeepAgents** (FilesystemMiddleware) | harness |
| Sub-agents | **DeepAgents** (SubAgentMiddleware) | harness |
| HITL mechanism | **DeepAgents** (HumanInTheLoop + `interrupt_on`) | harness, configured by Jarvis |
| Model routing (LLM↔SLM) | Jarvis | `model/router.py` |
| Budget caps (P9) | Jarvis | `orchestration/middleware.py` `BudgetGuard` |
| Injection defense (P9) | Jarvis | `guardrails/middleware.py` `InjectionGuard` |
| Context *loading* (P2/P3) | Jarvis | `agents/personal/agent.py` `ContextLoader` |
| Prompts (P1) | Jarvis (+ harness base) | `prompts/`, `model/prompts.py` |
| Approval surfacing → Slack (P6) | Jarvis | `interaction/inbox.py`, `hitl.py`, `channels/` |
| Triggers (start/resume runs) | Jarvis | `orchestration/triggers.py` |
| Sleep-time compute | Jarvis (sibling, not in-loop) | `orchestration/sleep.py` + cron |
| Persistence / checkpointer | Jarvis (config) | `agent.py` `_CHECKPOINTER` (durable → P10) |
| Custom domain tools (P4) | Jarvis | `tools/` + `_langchain_tools` adapter |

**Rule of thumb:** if it's the generic mechanics of "an LLM running tools in a
loop," the harness owns it. If it's a *Jarvis policy* (which model, what's safe,
who gets asked, what's remembered), Jarvis owns it — as middleware or config.

---

## The factors

### 1. Natural language → tool calls
*Principle:* the model emits structured tool calls; deterministic code executes them.
**In Jarvis:** the harness handles tool-calling. Our `tools/base.py` `ToolResult`
is the discipline for *custom* domain tools, adapted to LangChain tools at the
boundary (`_langchain_tools`).

### 2. Own your prompts
*Principle:* prompts are version-controlled code, not buried strings.
**In Jarvis:** `src/jarvis/prompts/*.md`, loaded via `model/prompts.py` and
`agent.format_personal_system_prompt`.
**Adaptation:** the harness prepends its *own* base instructions (todos, FS,
sub-agents). We own our layer (`personal/system.md`); the harness owns its base —
the effective system prompt is the concatenation.

### 3. Own your context window
*Principle:* engineer the whole token set — JIT loading + compaction.
**Adaptation:** split ownership. **JIT loading = Jarvis** (`ContextLoader`
middleware). **Compaction = the harness** (SummarizationMiddleware + filesystem
offload). `context/compaction.py` is **not wired** — the harness does it better.
`context/` is now the retrieval substrate + the loading seam.

### 4. Tools are structured outputs
*Principle:* tools return structured data, never raw strings.
**In Jarvis:** `ToolResult` for custom tools.
**Smell to reconcile:** harness built-in tools use LangChain's result model, so
`ToolResult` is internal to *our* tools only. The `BaseTool → LangChain` adapter
is the boundary (TODO).

### 5. Unify execution + business state
*Principle:* one state object holds execution (steps, todos) and business (messages) state.
**In Jarvis:** `JarvisDeepState` (`DeepAgentState` + `step_count`/`estimated_cost_usd`)
is the runtime state — messages, todos, files, budget, all in one.
**Smell to reconcile:** `core/state.py` `AgentState` is a *separate* jarvis-native
state used by the approval helpers (`inbox`, `hitl`, `loop`). With native HITL the
harness's interrupt *is* the pending approval, so `AgentState.pending_approvals`
is partly parallel. Decide: source of truth, or Slack-surfacing view only?

### 6. Launch / pause / resume
*Principle:* simple pause/resume backed by a checkpointer.
**In Jarvis:** the harness graph + `_CHECKPOINTER` (`agent.py`); **native HITL**
pause/resume on a `thread_id`. *Strengthened* by the collapse. Durable checkpointer
is the remaining gap (P10 — `docs/todo/p10-runtime-deployment.md`).

### 7. Contact humans via tool calls
*Principle:* reaching a human is a tool call / interrupt, not a side channel.
**In Jarvis:** `interrupt_on` (native HITL) is the mechanism; `interaction/inbox.py`
+ `hitl.py` bridge the interrupt → approval inbox → Slack (`channels/slack.py`).

### 8. Own your control flow  ← our biggest adaptation
*Principle (original):* own the loop yourself — "explicit code, not framework magic."
**In Jarvis:** we run **DeepAgents' loop** and own control flow as **middleware**:
`ContextLoader.before_agent`, `BudgetGuard.before_model`,
`InjectionGuard.wrap_tool_call`, and `interrupt_on`. The old
`orchestration/loop.py:should_continue` is a retained *reference predicate*, not
wired into the graph.
**Why we deviate:** #8's intent is owning control *points* (validate, pause/resume,
break out) — which middleware hooks are: explicit, inspectable, testable code. We
introspected the harness, so it isn't opaque "magic." Anthropic ("add complexity
only when it pays") and Letta (the field is converging on one well-understood loop)
back the trade. We accept the framework loop **in exchange for** owning the points.
See [ADR 0001](../architecture/decisions/0001-loop-channels-gateways-integrations.md).

### 9. Compact errors into context
*Principle:* errors are compact and agent-readable.
**In Jarvis:** `core/errors.py` `to_agent_message()`. Harness tool errors surface
as tool messages.

### 10. Small, focused agents
*Principle:* each agent does one thing; compose via sub-agents.
**In Jarvis:** `agents/personal/` is the one agent; sub-agents via the harness
`task` tool.
**Adaptation:** DeepAgents is a *general* harness. We focus it by gating `execute`
(P9) and can exclude unused middleware/tools — pruning toward "focused" is ongoing.

### 11. Trigger from anywhere
*Principle:* decouple triggers from agent logic; respond on any channel.
**In Jarvis:** `orchestration/triggers.py` (Slack event, cron, webhook, approval
response) — all on the runtime boundary, never importing `agents/`.

### 12. Stateless reducer
*Principle:* nodes are `(state) -> state_update`; no process-memory side effects.
**In Jarvis:** the harness's nodes/middleware return state updates (e.g. `BudgetGuard`
returns `{"step_count": n+1}`). State lives in the checkpointer.
**Caveat:** `_CHECKPOINTER` is in-process today, so disposability (12-Factor App IX)
is only partly met until it's durable (P10).

---

## 12-Factor *App* layer

The infrastructure factors (see [principles.md](principles.md)) are **unaffected**
by the harness collapse — config, logs, dependencies, build/release, admin
processes all hold. The one open item is **IX Disposability / VI Processes**:
state is externalized to the checkpointer, but that checkpointer must become
**durable** (Sqlite dev / Postgres prod) for a worker to truly die and resume. See
`docs/todo/p10-runtime-deployment.md`.
