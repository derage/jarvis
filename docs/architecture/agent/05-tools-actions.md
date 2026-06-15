[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 4 — Tools / Actions

**What it is:** The agent's hands — everything it can *do* to the outside world. Tools (atomic capabilities), MCP (a protocol for exposing tools), and skills (packaged procedures) all live here.

**The teaching — tool design:** Tools are the contract between the agent and its environment, so design them like good code: **clear, non-overlapping functions; well-scoped, self-contained, error-resilient purpose; unambiguous input parameters.** The test: if a human engineer can't confidently pick the right tool from the set, the agent will do worse. A toolset that grows until functions overlap is a primary failure mode — and it's *also* a Context problem, since every tool definition costs tokens.

**The teaching — agent-legible APIs (the Agent–Computer Interface):** The APIs your agent calls are part of its design surface, not a given. An API built only for human developers is often hostile to an agent. The chain to watch:

> Bad API → you compensate with verbose prompts and tool descriptions → context bloats → MCP server gets heavy → agent degrades.

So fixing the API upstream is frequently the cheapest way to shrink your MCP server. Three properties make an API agent-legible:

1. **Honest status codes.** A 200 on a real failure is the worst case — the agent believes it succeeded and proceeds on a false premise. It cannot recover from an error it can't see.
2. **Real validation.** Reject bad input at the boundary instead of silently accepting it and failing weirdly downstream.
3. **Actionable errors.** An error should tell the agent *what* was wrong and *how* to fix it, in enough detail to self-correct on the next turn — not just "400 Bad Request." This is the difference between an agent that retries intelligently and one that loops blindly.

Get these right and your prompts/tool descriptions shrink, because you're no longer documenting workarounds for a broken contract.

**Other tool hygiene:** make side effects idempotent (so retries are safe) and require explicit approval for irreversible actions.

**Simple vs. complex:**
- *Simple:* a handful of well-named tools, direct function calls.
- *Complex:* many tools behind dynamic loading, MCP servers, skills as reusable procedures. The more tools, the more the Context and tool-design disciplines matter.

**Architect's checklist:**
- [ ] Are tools non-overlapping and unambiguous? Could a human pick the right one every time?
- [ ] Are the APIs behind the tools agent-legible — honest codes, real validation, actionable errors?
- [ ] Are side effects idempotent? Which actions are irreversible and need a human gate?
- [ ] Is every tool definition earning its token cost, or is it bloating context?
- [ ] Are any tool descriptions secretly compensating for a bad API? (If so, fix the API.)

**Failure mode:** Tool sprawl (overlapping tools, the model picks wrong) and the dishonest-API trap (silent 200s, useless errors, prompts ballooning to compensate).

---
[← Prev: Memory](04-memory.md) · [Overview](00-overview.md) · [Next: Orchestration →](06-orchestration.md)
