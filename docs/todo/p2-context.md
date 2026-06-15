# TODO: P2 — Context

## Compaction is the harness's job now (ADR 0001)

DeepAgents' SummarizationMiddleware + filesystem offload own context compaction.
`src/jarvis/context/compaction.py` is **not wired** — keep it only if you want a
*custom* compaction policy, in which case express it as agent middleware (a
`before_model` hook), not a standalone function.

- [ ] Decide: rely on the harness's summarization (default), or write a custom
      compaction middleware (then: real token counting via `tiktoken` + `worker_model`)

## Missing files — create these

- [ ] **`context/loader.py`** — JIT context loading
  - `load_for_task(task_description)` fetches only what's needed from memory + tools
  - Returns a list of context items with their token costs
  - Respects `max_tokens` budget (reject if loading would overflow)

- [ ] **`context/notes.py`** — Structured scratchpad
  - `read_note(key)` / `write_note(key, value)` operating on `AgentState.notes`
  - Notes survive compaction because they're in state, not messages
  - Add a `notes_to_context_block()` helper that formats notes for the prompt

## `agents/personal/agent.py` — `ContextLoader` middleware

- [ ] Implement `ContextLoader.before_agent`: call `context/loader.py` (or memory
      directly) and inject the result into the run
- [ ] (No compaction hook needed — the harness handles compaction)

## Tests

- [ ] `tests/unit/test_context.py`
  - `test_compaction_threshold` — triggers at the right token count
  - `test_compact_keeps_recent` — recent messages are preserved verbatim
  - `test_notes_survive_compaction` — notes are in state, not messages
