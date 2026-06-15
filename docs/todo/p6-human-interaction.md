# TODO: P6 — Human Interaction

Approval inbox and HITL bridge are done. Slack channel scaffold exists. Streaming and
handoff need to be built.

## `src/jarvis/interaction/hitl.py` (done)

- [x] `approvals_from_interrupt()` — map DeepAgents HITL interrupt → inbox approvals
- [x] `resume_payload()` — build HITLResponse for `Command(resume=...)`
- [ ] Wire `orchestration/triggers.handle_approval_response()` → runtime `Command(resume=resume_payload(...))` on the same thread (native resume, ADR 0001)

## `interaction/channels/slack.py` (scaffolded)

- [x] `post_reply()` and `surface_approval()` seams
- [ ] Block Kit interactive buttons for approve/reject
- [ ] Wire `integrations/slack/client.py` with `slack-sdk` (`pip install -e ".[channels]"`)

## Missing files — create these

- [ ] **`interaction/streaming.py`** — Real-time visibility
  - Wrap LangGraph's built-in streaming to surface: current node, tool being called, reasoning steps
  - Helper: `stream_to_console()` for dev, `stream_to_websocket()` for production UI
  - The user should never wait through a long silence wondering what's happening

- [ ] **`interaction/handoff.py`** — Graceful human handoff
  - Called when agent is stuck, low-confidence, or hits a guardrail
  - Formats a handoff message: what was attempted, what failed, what the human should do
  - Adds a `handoff_requested` flag to state so the UI can surface it

## `interaction/inbox.py`

- [x] `request_approval()` / `process_approval()` with decision fed back via notes + messages
- [ ] Add expiry: approval requests older than N minutes auto-reject
- [ ] Add `list_pending_approvals(state)` helper for the UI to poll

## Agent Chat UI

- [ ] Run `make dev-ui` and verify the graph connects at `http://localhost:2024`
- [ ] Confirm tool calls render in the UI
- [ ] Confirm interrupt/approval requests surface as "awaiting input" state

## Tests

- [x] `tests/unit/test_interaction.py` — request_approval, process_approval, HITL bridge
- [ ] `test_gated_tool_pauses` — a gated tool (e.g. execute) raises a native HITL interrupt
- [ ] Integration test for full HITL pause → approve → resume cycle (needs API key)
