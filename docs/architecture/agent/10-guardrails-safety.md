[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 9 — Guardrails & Safety

**What it is:** The boundaries — input/output validation, permissions, injection defense, execution isolation, budgets, reliability, and human-in-the-loop.

**The teaching:** The more an agent can *act*, the more it can go wrong, so power requires guardrails. Key principles:

- **Treat retrieved content as untrusted input.** Web pages, documents, and tool outputs can carry prompt-injection payloads. Separate "data" from "instructions." Watch specifically for *indirect injection* (a malicious instruction buried in a retrieved doc), *data exfiltration via tools* (the agent tricked into sending data out), and *confused-deputy* problems (the agent using its privileges on an attacker's behalf).
- **Least privilege.** Restrict tool permissions; require approval for irreversible actions.
- **Execution isolation.** When the agent runs code or tools with side effects, isolate execution — a sandbox, container, or ephemeral workspace — so a bad action can't touch the host or other tenants. (Infra side lives in [Runtime](11-runtime-deployment.md).)
- **Input/output guardrails.** Validate incoming requests (block malformed/injection attempts) and filter outputs (catch policy violations, sensitive-data leakage) before the user sees them.
- **Budgets and timeouts.** Cap loops, API calls, cost, and time so a stuck agent can't burn money calling the same endpoint thousands of times.
- **Reliability patterns.** Retries with backoff, fallbacks, circuit breakers, and graceful degradation so a single failed dependency doesn't sink the run.

**Simple vs. complex:**
- *Simple:* basic input validation + timeouts.
- *Complex:* layered input/output guardrails, injection defense, sandboxed execution, RBAC, audit logging, human approval gates — scaled to blast radius.

**Architect's checklist:**
- [ ] What's the blast radius of a wrong action? Which actions are irreversible?
- [ ] Is retrieved/tool-returned content treated as untrusted (instruction/data separation)?
- [ ] Does code/tool execution happen in an isolated sandbox?
- [ ] Are there hard budgets/timeouts on loops, calls, and cost?
- [ ] Are reliability patterns (retry, fallback, circuit breaker) in place for flaky dependencies?
- [ ] Where does a human need to approve before the agent acts? Are guardrail decisions logged for audit?

**Failure mode:** The runaway loop ($800 of inference, an API hit 4,200 times, nothing accomplished); the injection hole (a malicious document silently redirects the agent); and unsandboxed execution letting a bad tool call escape into the host.

> Note: budgets/timeouts become non-optional once [Orchestration](06-orchestration.md) gives the agent autonomy (continue-by-default loops, self-enqueued work) — exactly the kind that can run away. The human-approval path is the safety side of [Human Interaction](07-human-interaction.md).

---
[← Prev: Improvement Loop](09-improvement-loop.md) · [Overview](00-overview.md) · [Next: Runtime & Deployment →](11-runtime-deployment.md)
