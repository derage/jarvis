[← Overview](00-overview.md) · Agent Architecture Blueprint

# Cross-cutting — Cost & Latency

**Not a pillar — the budget every pillar trades against.** A correct agent that costs $2 a turn or takes 40 seconds is a dead product. Cost and latency aren't an afterthought you optimize later; they're a constraint you design against from the start, as first-class as accuracy.

**The teaching:** Set a cost-per-task and a p95-latency budget up front, measure them as core metrics (this rides on [Evaluation](08-evaluation-observability.md)), and treat blowing the budget as a failing test — not a "nice to have." Then know where the levers are, because nearly every pillar has one:

- **[Model](02-model-reasoning.md)** — the biggest lever by far. Tiered/heterogeneous models and SLMs cut cost 10–30× on the calls that don't need a frontier LLM.
- **[Context](03-context.md)** — fewer resident tokens = cheaper and faster every turn. Just-in-time loading over preloading.
- **[Orchestration](06-orchestration.md)** — fewer steps and tool calls; parallelize independent work; cap runaway loops.
- **[Retrieval](12-retrieval-rag.md)** & **[Gateways](13-gateways.md)** — cache repeated lookups and model calls; route to the cheapest model that clears the bar.
- **[Memory](04-memory.md)** — sleep-time compute amortizes heavy reasoning into idle periods so interactive turns stay cheap and fast.

**Architect's checklist:**
- [ ] What's the cost-per-task and p95-latency budget for this agent?
- [ ] Are cost and latency tracked as first-class metrics, with alerts when they drift?
- [ ] Where is a frontier LLM doing work an SLM or cached result could do?
- [ ] What's cacheable (retrieval results, model calls, tool outputs)?
- [ ] Are there hard caps so a single task can't blow the budget?

**Failure mode:** A technically excellent agent that's too slow or too expensive to ship — the demo dazzles, the unit economics kill it.

---
[← Prev: Gateways](13-gateways.md) · [Overview](00-overview.md) · [Next: Where the buzzwords fit →](15-buzzwords-glossary.md)
