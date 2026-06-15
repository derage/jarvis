[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 3 — Memory

**What it is:** What the agent retains beyond the current window, so you don't re-teach it the same thing every session.

**The teaching:** Memory is what turns a stateless chatbot into something that learns and improves over time. The standard taxonomy (from CoALA / cognitive science):

- **Working / short-term** — the context window itself; ephemeral.
- **Episodic (long-term)** — specific past experiences and their outcomes ("last time I used pandas on a 5GB file it OOM'd; polars with lazy eval fixed it").
- **Semantic (long-term)** — facts, preferences, domain knowledge. Knowledge bases live here.
- **Procedural (long-term)** — *how* to do things; reusable plans/skills validated by prior runs. (This overlaps with the Tools pillar — a learned skill is procedural memory.)

The lifecycle that makes memory valuable is **consolidation**: compressing short-term experience into long-term storage, isolating signal from conversational noise — plus *intelligent forgetting* and *conflict resolution* so memory doesn't drift or bloat. The feeling that an agent can "have original ideas" largely comes from episodic + procedural memory letting it retrieve and re-apply what worked before.

**Simple vs. complex:**
- *Simple:* no persistent memory, or a single summary string carried forward.
- *Complex:* a dedicated memory store (e.g. vector-backed) with episodic/semantic/procedural separation, consolidation, decay, and conflict resolution. Worth it for personal assistants (episodic), domain experts (semantic), and any agent meant to improve with use.

**Architect's checklist:**
- [ ] Which memory types does this agent actually need? (Don't build all three by default.)
- [ ] What's the write policy — what gets promoted from short-term to long-term, and when?
- [ ] What's the consolidation/forgetting strategy? How are conflicting memories resolved?
- [ ] How is memory retrieved into context (and does that retrieval respect the Context budget)?

**Failure mode:** Either no memory (re-teaching every session) or unbounded memory (drift, stale facts, retrieval that floods the window).

> Related: consolidation / "dreaming" is triggered from the [Orchestration](06-orchestration.md) loop (sleep-time compute); memory stores are backed by [Retrieval (RAG)](12-retrieval-rag.md).

---
[← Prev: Context](03-context.md) · [Overview](00-overview.md) · [Next: Tools / Actions →](05-tools-actions.md)
