[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 7 — Evaluation & Observability

**What it is:** Being able to see what the agent did, score it, and catch regressions. The single biggest gap between a demo and a production agent. (Eval *measures*; the [Improvement Loop](09-improvement-loop.md) *acts* on those measurements.)

**The teaching:** You can't improve a loop you can't see. Evaluate **full trajectories, not just the final message** — tool-choice correctness, argument validity, step count, time/cost, and policy compliance. Run eval suites in CI/CD and **block changes that drop quality or safety below a threshold**, then reuse the same eval logic on sampled production traffic to catch drift after a prompt, model, tool, or data change. Track task-completion rate, tool-call accuracy, drift, and cost-per-request, and alert on them. Edge cases found in production become tomorrow's test cases — that feedback loop is the flywheel.

**Simple vs. complex:**
- *Simple:* manual spot-checks and a few golden test cases.
- *Complex:* automated trajectory evals in CI, production trace sampling, dashboards, drift alerts. Tools: LangSmith, Langfuse, Arize/Phoenix, Braintrust.

**Architect's checklist:**
- [ ] What does "the agent did its job" mean here, measurably? (Define success criteria first.)
- [ ] Are full traces captured — every tool call, argument, and result?
- [ ] Is there an eval suite gating changes in CI?
- [ ] What metrics are tracked in production, and what triggers an alert?
- [ ] How do production failures get fed back into the eval set? (Hands off to the [Improvement Loop](09-improvement-loop.md).)

**Failure mode:** Shipping on vibes. The agent looks great in the demo, then silently degrades in production with no trace to debug from. (Informal evals — like comparing two setups by hand — are a real start; the move is to formalize that instinct.)

---
[← Prev: Human Interaction](07-human-interaction.md) · [Overview](00-overview.md) · [Next: Improvement Loop →](09-improvement-loop.md)
