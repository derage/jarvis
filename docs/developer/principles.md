# Design Principles

The two rule sets that govern every decision in this codebase.

## 12-Factor App (infrastructure)

| Factor | How it applies to Jarvis |
|---|---|
| I. Codebase | One repo, one package. Multiple agents are entry points, not repos. |
| II. Dependencies | Declared in `pyproject.toml`. Never `pip install` ad hoc. |
| III. Config | Everything in env vars. `core/config.py` reads them. No hardcoded keys. |
| IV. Backing services | Memory stores, vector DBs, queues are attached resources. Swap via config. |
| V. Build/release/run | `make build` → artifact. `make deploy` → release. `make dev` → run. |
| VI. Processes | Agents are stateless. All state lives in the LangGraph checkpointer. |
| IX. Disposability | Any worker can die and restart. State is externalized (P10). |
| X. Dev/prod parity | Same Docker image, same env var names. `.env.example` documents all vars. |
| XI. Logs | `structlog` to stdout only. JSON in prod, pretty in dev. Never to a file. |
| XII. Admin processes | `scripts/` for one-off tasks. Run via `make seed-memory`, `make eval`. |

## 12-Factor Agents (agent design)

Each factor — and where Jarvis deliberately **adapts** it (notably #8: we run the
DeepAgents loop and own control flow as *middleware*) — is detailed in the owned
version: [twelve-factor-agents.md](twelve-factor-agents.md). Quick index:

| Factor | Where it lives |
|---|---|
| 1. Natural language → tool calls | `tools/base.py` — structured ToolResult, not free text |
| 2. Own your prompts | `src/jarvis/prompts/` — version-controlled `.md` files, loaded via `model/prompts.py` |
| 3. Own your context window | JIT load → `ContextLoader` middleware; compaction → **the harness**. [Details](twelve-factor-agents.md) |
| 4. Tools are structured outputs | `tools/base.py` — ToolResult dataclass, never raw strings |
| 5. Unify execution + business state | `JarvisDeepState` (agent.py) = runtime state; `core/state.py` `AgentState` = helper state. [Details](twelve-factor-agents.md) |
| 6. Launch/pause/resume | harness graph + `_CHECKPOINTER` (`agent.py`); native HITL pause/resume |
| 7. Contact humans via tool calls | `interaction/inbox.py` — approval requests are tool calls |
| 8. Own your control flow | **via middleware** (Budget/Injection/Context guards + `interrupt_on`); we run the DeepAgents loop, not a hand-rolled one — [why](twelve-factor-agents.md) |
| 9. Compact errors into context | `core/errors.py` — `to_agent_message()` on every error type |
| 10. Small focused agents | `agents/` — each agent does one thing; compose via subagents |
| 11. Trigger from anywhere | `orchestration/triggers.py` — webhook, cron, message queue |
| 12. Stateless reducer | Every node: `(state) -> state_update`. No side effects in process memory. |

## The dependency rule

```
agents/ → pillar modules → integrations/ → core/
```

Pillars never import from other pillars — anything two pillars share (e.g. a Slack client used by both `orchestration/triggers.py` and `interaction/channels/`) lives in the inward `integrations/` ring, not in a sibling pillar. This keeps each pillar independently replaceable.

## The logging rule

Every significant action gets a structured log event:

```python
log.info("tool_called", tool="get_weather", city="Chicago")
log.warning("injection_detected", source="retrieved_doc", pattern="ignore instructions")
log.error("tool_failed", tool="calendar", error="rate limited", retry_in=60)
```

Never use `print()`. Never log to a file. Logs flow to stdout and are collected by the runtime.
