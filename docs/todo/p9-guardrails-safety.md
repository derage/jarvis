# TODO: P9 — Guardrails & Safety

Injection scanner and middleware are working. Input validation, output filtering, and
cost tracking need real logic.

## `src/jarvis/guardrails/output.py`

- [ ] Add PII detection: SSN pattern, credit card pattern, phone numbers
- [ ] Add hallucinated tool call detection: output contains `{"tool": ...}` outside a tool call
- [ ] Add response length guardrail: warn if response is suspiciously short or long
- [ ] Log every filtered output to the observability layer for audit

## Missing file — create `guardrails/input.py`

- [ ] `validate_user_input(message: str) -> str` — sanitize incoming user messages
- [ ] Block/flag inputs that look like injection attempts on Jarvis itself
- [ ] Rate limit: track request frequency per session, reject if too fast (runaway trigger)

## `src/jarvis/guardrails/injection.py`

- [ ] Add more injection patterns (current list is minimal)
- [ ] Add a LLM-based fallback for ambiguous content: call a cheap model to classify
- [ ] Make pattern list configurable (load from config, not hardcoded)
- [ ] Test against known injection payloads from research papers

## Budget enforcement

Step cap is enforced via `BudgetGuard` middleware (`orchestration/middleware.py`) —
`before_model` hook calls `budget_exceeded()` and jumps to end when `step_count` hits
`JARVIS_MAX_STEPS`. Cost cap uses the same predicate but is not yet live.

- [x] Step cap enforced in agent loop via `BudgetGuard` middleware (ADR 0001)
- [ ] **Wire `estimated_cost_usd`** — increment after each LLM call in an `after_model`
      hook on `BudgetGuard` (or a sibling middleware). Token counting + USD conversion.
- [ ] Surface `estimated_cost_usd` from `JarvisDeepState` to the user / observability
      (one state now — no outer graph to sync with)
- [ ] Add a `budget_warning` log when 80% of the cost cap is reached (before hard stop)
- [ ] Test cost limit enforcement once `estimated_cost_usd` is wired (see
      `docs/todo/testing.md`)

## Sandbox (future)

- [ ] When tools execute code or shell commands, wrap in a sandbox
  - Option A: Docker container per run
  - Option B: Deep Agents sandbox backends (`LocalShellBackend` → sandbox backend)

## Tests

- [x] `tests/unit/test_guardrails.py` — injection scanner, sanitize, InjectionGuard middleware
- [x] `tests/unit/test_orchestration.py` — BudgetGuard step cap
- [ ] `test_budget_guard_halts_on_cost_cap` — once `estimated_cost_usd` is incremented
- [ ] `test_output_filter_catches_pii` — once `output.py` is implemented
