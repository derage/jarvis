# TODO: Testing

## Unit tests — fill in as pillars are implemented

| File | Covers | Status |
|---|---|---|
| `tests/unit/test_core.py` | Settings, errors, AgentState, TodoItem | ✅ Done |
| `tests/unit/test_tools.py` | ToolResult, BaseTool, registry | ✅ Done |
| `tests/unit/test_orchestration.py` | should_continue, todo queue, BudgetGuard middleware | ✅ Done |
| `tests/unit/test_model.py` | Prompt loading, model config | ✅ Scaffolded (stubs skip gracefully) |
| `tests/unit/test_context.py` | Budget errors, notes pattern, compactor import | ✅ Scaffolded |
| `tests/unit/test_memory.py` | Working memory, memory module imports | ✅ Scaffolded |
| `tests/unit/test_guardrails.py` | Injection detection/sanitization, limit errors, InjectionGuard | ✅ Done |
| `tests/unit/test_interaction.py` | Approval inbox, HITL bridge (approvals_from_interrupt, resume_payload) | ✅ Done |
| `tests/unit/test_logging.py` | configure_logging, get_logger, JSON vs pretty | ⬜ Not started |
| `tests/unit/test_runtime.py` | P10 runtime config and deployment helpers | ⬜ Not started |
| `tests/unit/test_agent.py` | format_personal_system_prompt, build_personal_graph (mock_llm) | ⬜ Not started |

## Integration tests — build as real backends are wired in

| File | Covers | Status |
|---|---|---|
| `tests/integration/test_llm_roundtrip.py` | Real Anthropic API call returns a response | ⬜ Not started |
| `tests/integration/test_memory_store.py` | Write/read from LangGraph Memory Store | ⬜ Not started |
| `tests/integration/test_web_search.py` | Tavily search tool with real key | ⬜ Not started |
| `tests/integration/test_graph_e2e.py` | Full graph run, real LLM, check end state | ⬜ Not started |

## Eval tests — build after first-run is working

| What | Description | Status |
|---|---|---|
| Greeting eval | Agent responds coherently to "hello" | ⬜ Not started |
| Memory recall eval | Agent can retrieve a seeded user fact | ⬜ Not started |
| Tool use eval | Agent calls web search when asked about current news | ⬜ Not started |
| Approval flow eval | Agent pauses for approval before a sensitive action | ⬜ Not started |
| Long-task eval | Agent completes a 5-step research task with no context overflow | ⬜ Not started |

## Coverage

- Current threshold: **70%** (intentionally high — skeleton stub modules are at 0%)
- Measured coverage today: ~52% — expected until agent/trigger/integration modules are tested
- Run `make coverage` to see the HTML report
- Options when ready: add smoke tests for stub modules, or lower `fail_under` in `pyproject.toml` temporarily
- Bump threshold as coverage improves

## Skeleton-phase note

Lint (`make lint`) and typecheck (`make typecheck`) are not yet clean — strict mypy + LangGraph/DeepAgents typings need attention as the agent layer solidifies. Do not run `make format` blindly on the skeleton; some ruff auto-fixes would remove intentional stub imports.
