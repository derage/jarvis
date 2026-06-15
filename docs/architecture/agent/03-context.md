[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 2 — Context

**What it is:** Engineering the working window — what the model actually sees at inference time, and how that set is curated as the task runs.

**The teaching:** Treat context as a finite, precious resource. The goal is **the smallest set of high-signal tokens that maximize the likelihood of the desired outcome.** More context is not better; past a point you hit *context rot* — the more tokens loaded, the worse the model gets at finding the right one.

Two loading strategies:
- **Static / up-front:** load everything you might need. Simple, but bloats fast and rots.
- **Just-in-time / dynamic:** load lightweight identifiers, fetch full detail at runtime via tools. Mirrors how a person uses notes and bookmarks. This is what good agentic coding tools do — dozens of MCP tools available, but only ~10k tokens resident because tools are pulled in dynamically rather than all preloaded.

When the window fills, the tactics are: **compaction** (summarize and restart from a compressed summary), **structured notes** (persist state outside the window), and **sub-agent architectures** (isolated contexts that return only condensed results).

**Simple vs. complex:**
- *Simple:* fixed system prompt + recent message history. Fine for short, single-turn-ish tasks.
- *Complex:* dynamic tool/context loading, compaction policies, externalized notes — required for long-horizon tasks or any setup with heavy tool/MCP surface.

**Architect's checklist:**
- [ ] What's the token budget, and what's resident vs. fetched-on-demand?
- [ ] Are tools loaded dynamically or all preloaded? (Preloading many MCP tools is the #1 silent bloat source.)
- [ ] What's the compaction trigger, and what gets preserved vs. dropped?
- [ ] Is persistent state held outside the window (notes/scratchpad) rather than re-fed every turn?
- [ ] Is the system prompt at the right altitude? (See [Pillar 1 — Instruction design](02-model-reasoning.md).)

**Failure mode:** The thrash loop — e.g. two heavy MCP servers consuming ~100k of a 200k window, compaction firing at 120k on every turn, the agent constantly summarizing and forgetting until it's useless. The fix is upstream (load less), not more aggressive summarizing.

---
[← Prev: Model / Instruction / Reasoning](02-model-reasoning.md) · [Overview](00-overview.md) · [Next: Memory →](04-memory.md)
