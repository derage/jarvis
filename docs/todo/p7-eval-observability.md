# TODO: P7 — Evaluation & Observability

Tracing config and metrics data class are stubbed. Need real eval cases and dashboards.

## `src/jarvis/observability/tracing.py`

- [ ] Add custom span metadata: session_id, step_count, task description, tool name
- [ ] Add `@trace_node` decorator for wrapping graph nodes with a named span
- [ ] Test that traces appear in LangSmith: set key, run `make dev`, send a message

## `src/jarvis/observability/metrics.py`

- [ ] Persist `RunMetrics` after each run — write to SQLite or LangSmith dataset
- [ ] Add `tool_error_rate` alerting: log a warning when rate > 20%
- [ ] Add p95 latency tracking across runs

## `src/jarvis/observability/evals/`

- [ ] **`evals/trajectory.py`** — Framework for trajectory-level eval cases
  - Each eval: input message, expected tools called, expected step count range
  - Scorer: did the agent call the right tools? Did it stop at the right time?

- [ ] **`evals/cases/basic.py`** — First eval cases
  - "What's my schedule tomorrow?" → should call `calendar`, ≤ 3 steps
  - "Search for the latest LangGraph release" → should call `web_search`, return structured result
  - "Remember that I prefer morning meetings" → should store fact in semantic memory

## CI integration

- [ ] Add `make eval` to a GitHub Actions workflow
- [ ] Gate merges on eval pass rate ≥ threshold (start at 80%, raise over time)

## Tests

- [ ] `tests/unit/test_metrics.py` — `RunMetrics.tool_error_rate` calculation
- [ ] `tests/integration/test_evals.py` — run eval cases against live graph
