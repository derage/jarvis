# TODO: P10 — Runtime & Deployment

The graph runs locally with `MemorySaver`. Need persistent state and a deploy path.

## Persistent checkpointer (swap `_CHECKPOINTER` in `agent.py`)

- [ ] **Local dev:** swap to `SqliteSaver`
  ```python
  from langgraph.checkpoint.sqlite import SqliteSaver
  checkpointer = SqliteSaver.from_conn_string("jarvis.db")
  ```
  Add `jarvis.db` to `.gitignore`.

- [ ] **Production:** swap to `PostgresSaver`
  ```python
  from langgraph.checkpoint.postgres import PostgresSaver
  ```
  Add `DATABASE_URL` to `.env.example`.

- [ ] **Config-driven:** make checkpointer backend selectable via `JARVIS_CHECKPOINT_BACKEND=sqlite|postgres|memory`

## Stateless workers (future)

- [ ] Add a queue between intake and graph runner (Redis Streams or SQS)
- [ ] Worker pool: N stateless workers each pulling from the queue, loading state by thread_id
- [ ] Autoscale workers on queue depth, not on incoming request count

## LangSmith deployment

- [ ] Fill in `LANGSMITH_API_KEY` in `.env`
- [ ] Run `make deploy` and verify the graph deploys
- [ ] Set up environment variables in LangSmith deployment UI
- [ ] Test resume: start a run, kill the server, restart, confirm state is recovered

## Versioning & release

- [ ] Pin prompt + model + tool versions together in a `release.json` or similar
- [ ] Document rollback procedure: how to revert to the previous graph version
- [ ] Add canary support: run new graph version on 10% of traffic before full rollout (future)

## `agents/personal/agent.py` (where the checkpointer + interrupts live now)

- [ ] Make the checkpointer backend configurable (swap `_CHECKPOINTER`) — see above
- [ ] Ensure every run config carries a `thread_id` so state is scoped per conversation

## Tests

- [ ] `tests/integration/test_checkpointing.py`
  - Start a run, simulate a crash mid-run, resume, assert state is recovered
  - Verify that `step_count` and `todos` are preserved across resume
