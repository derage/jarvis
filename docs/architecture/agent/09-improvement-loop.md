[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 8 — Improvement / Learning Loop

**What it is:** The subsystem that makes the agent better over time. [Evaluation](08-evaluation-observability.md) *measures*; this pillar *acts on* the measurements. It's the engine that compounds — the difference between an agent that's static after launch and one that keeps getting sharper and cheaper.

**The teaching:** Production is the richest source of training signal you have; most teams throw it away. Close the loop:

- **Capture feedback** — both explicit (thumbs up/down, user edits, corrections) and implicit (task completion, retries, abandonment, time-to-done).
- **Turn traces into data** — production trajectories become eval cases *and* training data. Edge cases found in prod → tomorrow's regression tests → next model/prompt.
- **Act on it** — three levers, cheapest first: optimize the prompt (manually or with automated prompt-optimization), then fine-tune, then preference learning. Run on a deliberate cadence, not ad hoc.

This is also the engine behind the SLM strategy from [Pillar 1](02-model-reasoning.md): cluster your interaction logs by tool/task and fine-tune specialized small models for the hot spots. Your logs *are* the SLM's training set — which is what makes "slide down the model-size curve" actually achievable rather than aspirational.

**Simple vs. complex:**
- *Simple:* manually review failures and tweak the prompt.
- *Complex:* structured feedback capture, trace→dataset pipelines, scheduled fine-tunes, automated prompt optimization, preference data, log-clustering to specialize SLMs.

**Architect's checklist:**
- [ ] Is feedback captured structurally — both explicit and implicit signals?
- [ ] Do production traces flow into the eval/training set, or are they discarded?
- [ ] What's the improvement cadence and unit — prompt tweak, fine-tune, preference update?
- [ ] Are you optimizing the prompt systematically, or by vibes?
- [ ] Are interaction logs being mined to specialize/distill smaller, cheaper models?

**Failure mode:** A static agent that never improves — every fix is a manual one-off, and the goldmine of production traces is never turned into better prompts, models, or evals.

---
[← Prev: Evaluation & Observability](08-evaluation-observability.md) · [Overview](00-overview.md) · [Next: Guardrails & Safety →](10-guardrails-safety.md)
