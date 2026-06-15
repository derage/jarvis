# TODO: P8 — Improvement / Learning Loop

Feedback data class is stubbed. Nothing is persisted or acted on yet.

## `src/jarvis/improvement/feedback.py`

- [ ] Implement `record_feedback(event)` — persist to LangSmith dataset or SQLite
- [ ] Add `get_feedback_summary(days=7)` — returns signal counts by type for review cadence
- [ ] Wire implicit feedback: detect retries (user repeats a similar request) and log `IMPLICIT_RETRY`
- [ ] Wire implicit feedback: detect task completion (todos all done) and log `IMPLICIT_COMPLETE`

## Missing file — create `improvement/datasets.py`

- [ ] `trace_to_dataset(run_id)` — pull a LangSmith trace, format as an eval case
- [ ] `export_failures(since_date)` — export all failed runs as a dataset for review
- [ ] `cluster_by_task_type(traces)` — group traces by tool usage pattern (feeds SLM specialization from P1)

## Missing file — create `improvement/optimizer.py`

- [ ] Stub for automated prompt optimization (DSPy or manual)
- [ ] `suggest_prompt_edits(dataset)` — analyze failure cases and suggest prompt changes

## Cadence setup

- [ ] Schedule a weekly review cron: `make eval` + `improvement/datasets.py` export
- [ ] Define what triggers a prompt update vs a fine-tune vs neither
- [ ] Document the improvement cadence in `docs/developer/improvement-loop.md`

## Tests

- [ ] `tests/unit/test_improvement.py`
  - `test_feedback_event_creation`
  - `test_implicit_retry_detection`
