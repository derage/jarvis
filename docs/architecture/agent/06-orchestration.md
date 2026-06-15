[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 5 — Orchestration

**What it is:** The control flow — the agent loop, how many agents there are, how they coordinate, and whether the loop keeps going on its own.

**The teaching:** A production agent is a control loop: plan, act via tools, observe, update state, repeat until goal or budget/timeout. The architecture runs from a single loop to multi-agent graphs with predefined paths, dynamic paths, and coded edges (the LangGraph design space). The principle that ages well: **build small agents with clear interfaces and let them compose — the complexity belongs in the composition, not in any single monolithic agent.**

## Autonomy & loop continuation

Does the loop keep going on its own, or stall waiting for the user? Four mechanisms, one decision underneath:

**The key decision — continue-by-default vs. terminate-by-default.** When the model finishes a step, does the loop continue automatically or halt? ReAct-style loops continue by default; classic MemGPT terminated by default. This is why one agent powers through a multi-step task while another announces "I'll go research more," then goes silent — to the second, the announcement and the follow-through were separate turns and nothing triggered the second. You own the default.

**Heartbeats (keep going *within* a task).** Lets an agent chain tool calls in one execution loop without user intervention (coined in the MemGPT paper). A failed tool call can auto-fire a heartbeat so the agent gets a turn to read the error and retry — which only works if the API returned a *readable* error (see [Tools](05-tools-actions.md)).

**The task queue / todo list (the loop's engine and stop condition).** The concrete mechanism for "continue until done": the agent maintains an explicit todo list, adds items as it discovers work, checks them off, and **goes dormant when the list is empty.** Before going dormant it can enqueue its own follow-up work — consolidation, memory updates, "dream" tasks — then drains those and stops. It's primarily Orchestration, but it's also [Context's](03-context.md) "structured notes" (externalized state surviving compaction) and the enqueue point for [Memory's](04-memory.md) consolidation.

**Ambient triggers (wake up to *check for* work).** The loop is kicked off by an event or schedule (webhooks, change streams, a cron timer) rather than a user message. LangChain calls these *ambient agents*: push-based, not pull-based.

**Sleep-time compute (use idle time to "dream").** When no task is pending, a background agent processes accumulated context — consolidating memory, precomputing. Letta coined the term (2025). Different from ambient triggers: ambient = periodic *external* work; sleep-time = internal *cognitive* prep over context the agent already has.

## Multi-agent coordination

Going multi-agent buys parallelism and isolation but inherits a class of distributed-systems problems the single-loop view hides. Name and decide these before you split:
- **Shared-state races** — two agents reading/writing the same memory block or record concurrently.
- **Conflict resolution** — what happens when two agents produce contradictory updates (last-write-wins? merge? a referee agent?).
- **Communication pattern** — direct calls, a shared blackboard/state, or a message queue between agents.
- **Ownership & deadlock** — who owns which resource, and can two agents wait on each other forever.

If you can't name your coordination model, you're not ready to go multi-agent.

**Simple vs. complex — how to choose:**
- *Single agent loop:* roughly linear task, modest tool set, a human can describe the steps. **Default here.**
- *Workflow (predefined paths):* steps known; you want predictability over flexibility.
- *Multi-agent / dynamic paths:* genuinely independent sub-problems, or sub-tasks needing isolated contexts. Reach for it when a single loop's context can't hold the job — not because it sounds sophisticated.
- *Autonomy level:* reactive turn-by-turn is simplest; add a todo-list loop for multi-step work; add ambient triggers when work should start without a human; add sleep-time compute only when reusable context justifies the cost.

**Architect's checklist:**
- [ ] Can this be a single loop? (Try to make it one before adding agents.)
- [ ] Does the loop continue by default or terminate by default — and is that what you want?
- [ ] Is there an explicit todo list / task queue with a clear "list empty → dormant" stop condition?
- [ ] What triggers the loop — only a user message, or also events/schedules (ambient)?
- [ ] If multi-agent: what's the coordination model (shared state, conflict resolution, communication, ownership)?
- [ ] Are there hard budgets/timeouts so the loop can't run away?

**Failure mode:** Over-architecting — a multi-agent graph where one loop would do — and its inverse, a single loop stuffed with a job too big for its context. With multi-agent specifically: silent shared-state races and unresolved write conflicts.

---
[← Prev: Tools / Actions](05-tools-actions.md) · [Overview](00-overview.md) · [Next: Human Interaction →](07-human-interaction.md)
