# Testing Strategy

Jarvis uses a three-tier test strategy with explicit pytest markers so you can run
exactly the level of testing that matches your current context.

## The three tiers

### Unit (default — `make test`)

- No API keys needed
- No network calls
- Runs in < 5 seconds
- LLM calls are mocked with `pytest-mock`
- HTTP calls are mocked with `respx`
- Everything in `tests/unit/`

### Integration (`make test-integration`)

- Requires real API keys in `.env`
- Makes real network calls to Anthropic, LangGraph, memory backends
- Verifies the wiring between Jarvis and external services
- Everything in `tests/integration/`

### Eval (`make eval`)

- Full trajectory-level agent evaluation
- Runs complete conversations against a golden dataset
- Slow — not part of CI by default
- Lives in `scripts/run_evals.py` (backed by LangSmith datasets)

## Running tests

```bash
make test               # unit only (no API keys needed)
make test-all           # all tiers (needs .env filled in)
make test-integration   # integration only
make test-watch         # re-run unit tests on file save
make coverage           # run unit tests + open HTML coverage report
```

## Marking tests

```python
import pytest

pytestmark = pytest.mark.unit          # top of file — marks every test in it

@pytest.mark.integration
def test_real_api_call(): ...

@pytest.mark.eval
def test_full_trajectory(): ...
```

## Mocking LLMs

Never make real LLM calls in unit tests. Use the `mock_llm` fixture from `conftest.py`:

```python
async def test_agent_runs_without_a_real_llm(mock_llm, mock_llm_response):
    # Build the harness UNDER the patch so its model resolves to the mock.
    from jarvis.agents.personal.agent import build_personal_graph
    mock_llm.ainvoke.return_value = mock_llm_response("I found 3 results.")

    agent = build_personal_graph()
    result = await agent.ainvoke(
        {"messages": [{"role": "user", "content": "search"}]},
        {"configurable": {"thread_id": "t1"}},
    )
    assert result["messages"]  # assert on outcomes, not LLM wording
```

The `mock_llm` fixture patches both `ChatAnthropic` and `ChatOpenAI` for the duration
of the test, so any code that builds an LLM client during the test gets the mock instead.

## Mocking HTTP calls (tools)

Tool tests that make HTTP calls use `respx` via the `mock_http` fixture:

```python
import httpx

def test_web_search_tool(mock_http, echo_tool):
    mock_http.post("https://api.tavily.com/search").mock(
        return_value=httpx.Response(200, json={"results": [{"title": "Test", "url": "..."}]})
    )
    result = web_search_tool(query="python asyncio")
    assert result.success is True
```

## Freezing time

Use `freezegun` for anything that depends on `datetime.now()` or timestamps:

```python
from freezegun import freeze_time

@freeze_time("2025-01-15 09:00:00")
def test_memory_with_timestamp(state):
    state.notes["created_at"] = "2025-01-15 09:00:00"
    assert state.notes["created_at"] == "2025-01-15 09:00:00"
```

## Shared fixtures

All fixtures in `tests/conftest.py` are automatically available. Key ones:

| Fixture | What it gives you |
|---|---|
| `state` | Clean `AgentState(session_id="test-session", run_id="test-run")` |
| `state_with_todos` | State with 2 pending todos |
| `state_at_step_limit` | State at the configured max_steps |
| `state_with_pending_approval` | State with a pending human approval |
| `mock_llm` | Patched LLM returning `"Test response"` by default |
| `mock_llm_response` | Factory: `mock_llm_response("custom text")` |
| `echo_tool` | `EchoTool` instance for tool chain tests |
| `failing_tool` | `FailingTool` — always returns `success=False` |
| `clean_tool_registry` | Restores registry state after your test (use `usefixtures`) |
| `mock_http` | `respx_mock` pre-configured for httpx interception |

## Coverage

Coverage is measured on every `make test` run and reported in the terminal.
An HTML report is generated at `.coverage-html/index.html` (open with `make coverage`).

**Threshold: 70%** — the build fails if coverage drops below this. During the skeleton
phase measured coverage is ~52% because stub modules (agents, triggers, integrations)
are intentionally untested. See `docs/todo/testing.md` for the plan.

To see exactly which lines are uncovered:

```bash
make test   # terminal shows "Missing" column with uncovered line numbers
```

## What NOT to test in unit tests

- Exact wording of LLM responses (non-deterministic)
- Latency or timing (flaky)
- Whether the LangGraph graph compiles (too slow; test the nodes, not the graph)
- External service availability (that's what integration tests are for)

## Adding new tests

1. Create `tests/unit/test_<pillar>.py`
2. Add `pytestmark = pytest.mark.unit` at the top
3. Use fixtures from `conftest.py`
4. Run `make test` to confirm they pass with no API keys
5. Add the file to the per-pillar todo doc once it exists
