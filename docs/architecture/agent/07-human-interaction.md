[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 6 — Human Interaction

**What it is:** The human edge — how a person sees, steers, trusts, and intervenes in the agent. Distinct from Guardrails (which constrain the agent); this is about the *experience* of working with it.

**The teaching:** Technically excellent agents fail in production for reasons that have nothing to do with the model — they're opaque, uninterruptible, or untrustworthy, so people stop using them. Four things make an agent usable:

- **Visibility.** Surface what the agent is doing in-flight — its plan, current step, and reasoning — not just a final answer after a long silence.
- **Steerability.** Let a human interrupt and redirect *mid-run*, not only at the start. An agent you can't stop or correct is one people won't trust with anything that matters.
- **Oversight UX.** Design how decisions get surfaced for approval. The **agent inbox** pattern is the sweet spot between fully autonomous and fully manual: the agent does the work but routes consequential decisions to a human queue for review/approval.
- **Trust & transparency.** Show provenance — citations, sources, confidence — and make it answerable *why* it did something. Provide a graceful handoff to a human when it's stuck or low-confidence.

**Simple vs. complex:**
- *Simple:* request → response, show the final answer.
- *Complex:* streamed reasoning/progress, mid-run interrupt and redirect, an approval queue / agent inbox, provenance and citations, explicit human-handoff paths.

**Architect's checklist:**
- [ ] Can the user see what the agent is doing while it runs, not just at the end?
- [ ] Can they interrupt and redirect mid-run?
- [ ] How are consequential decisions surfaced for human review/approval?
- [ ] Is the output attributable (sources, citations, confidence)?
- [ ] What happens when the agent is stuck or unsure — is there a clean handoff to a person?

**Failure mode:** A black-box agent users can't see into, stop, or trust — abandoned regardless of how good it is under the hood.

> Related: the approval/oversight path here is the human side of [Guardrails](10-guardrails-safety.md)' human-in-the-loop gates.

---
[← Prev: Orchestration](06-orchestration.md) · [Overview](00-overview.md) · [Next: Evaluation & Observability →](08-evaluation-observability.md)
