[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 10 — Runtime & Deployment *(the foundation)*

**What it is:** How the agent actually runs in production — the process/serving topology, where state is stored, how work is scheduled and scaled, how it's released, and how failures are handled. Numbered last, but decided early, because it constrains every pillar above it.

**The teaching — separate state from compute.** The most common production mistake is coupling the two: pinning a conversation to a living process that holds the graph *and* the state in memory. When config or the graph changes, you tear down the process — and its state — and eat the cold start. State should be long-lived and cheap to move; compute should be ephemeral and fungible. Three moves decouple them:

1. **Externalize state to a durable store.** Persist graph progress and conversation outside any process — a checkpointer keyed by a thread/session ID, snapshotting at every step. The conversation is no longer pinned to a PID; any worker can resume it.
2. **Make workers stateless and fungible.** A pool of identical workers, none "owning" a user. A worker pulls work, loads that thread's state, runs a step with the *current* graph definition, writes state back, releases. Changing the graph stops requiring a process-tree teardown.
3. **Put a queue between intake and workers** (Kafka, SQS, NATS, Redis Streams). Decouples arrival from execution, gives backpressure, and lets you autoscale on **queue depth** instead of spawning a process per request.

**The teaching — checkpointing vs. durable execution.** *Checkpointing* saves state between steps and leaves recovery to you. *Durable execution* (Temporal, DBOS, Restate, Inngest, Dapr Workflows) guarantees run-to-completion: on crash it replays event history to reconstruct exact state and resumes from the failure point. A common pattern is the agent graph (e.g. LangGraph) wrapped in a durable-execution worker. The catch that reaches up into the other pillars: **resume usually re-runs the interrupted node, including its LLM and API calls**, so nodes must be deterministic and idempotent — which is why this pillar *drives the internals*. It forces idempotent tools ([Pillar 4](05-tools-actions.md)), externalized state ([Pillars 2](03-context.md) & [3](04-memory.md)), and deterministic node logic ([Pillar 5](06-orchestration.md)).

**Versioning, reproducibility & release.**
- **Version as a unit.** Pin prompt + model + tool versions together so a trace replays deterministically and you can roll the whole bundle back, not just one piece.
- **Release safely.** Ship agent changes with canary, shadow mode (run the new version alongside the old without serving its output), and one-click rollback — the same discipline you'd use for any production service.
- **Execution environment.** Decide where tool/code execution physically runs and how it's isolated (sandbox/container) — the safety rationale is in [Guardrails](10-guardrails-safety.md).

**Transport vs. interop — don't conflate three layers:**
- **Message queue (Kafka, etc.)** — transport between *your own* components. The scaling fix.
- **A2A (Agent2Agent)** — open standard for *independent* agents (different teams/vendors) to discover and delegate across ownership boundaries. Reach for it only across a boundary. (See the [glossary](15-buzzwords-glossary.md).)
- **MCP** — agent-to-tool.

**Simple vs. complex:**
- *Simple:* a single stateless service with session state in a checkpointer + DB. Fine for low concurrency, short tasks.
- *Complex:* queue + autoscaled stateless worker pool + durable-execution engine + canary/shadow release. Needed for high concurrency, long-running tasks, or anything that must survive restarts.
- *Anti-pattern:* a per-user forked process holding graph and state in memory, where a graph/config change kills the whole process tree.

**Architect's checklist:**
- [ ] Is state externalized (checkpointer/durable store), or living in process memory?
- [ ] Are workers stateless and fungible, or does a process "own" a user/session?
- [ ] Is there a queue decoupling intake from execution? Do you autoscale on queue depth?
- [ ] Checkpointing or full durable execution — do interrupted runs resume automatically?
- [ ] Are nodes deterministic and idempotent, so they're safe to replay?
- [ ] Are prompt/model/tool versions pinned together and rollable as a unit? Is there a canary/shadow path?
- [ ] Does changing the graph/config require killing live processes? (If yes, fix the coupling.)

**Failure mode:** Coupling state to a forked process so a config/graph change forces a process-tree teardown and cold start — and the inverse, holding no durable state so a pod restart loses all in-flight work.

---
[← Prev: Guardrails & Safety](10-guardrails-safety.md) · [Overview](00-overview.md) · [Next: Retrieval (RAG) →](12-retrieval-rag.md)
