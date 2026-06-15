[← Overview](00-overview.md) · Agent Architecture Blueprint

# Cross-cutting — Retrieval (RAG)

**Not a pillar — the data substrate underneath several of them.** Chunking, embedding, and vector search show up in three places:

- **Under [Memory](04-memory.md)** — episodic and semantic memory are typically vector-backed for similarity search.
- **Under [Context](03-context.md)** — just-in-time retrieval keeps the window small while still reaching the right info.
- **Under [Tools](05-tools-actions.md)** — a knowledge base exposed as a searchable tool ("how do I do X that I don't already know") is retrieval-backed semantic memory.

## The data pipeline behind it

Retrieval quality is entirely downstream of data engineering — garbage in, garbage out. An agent that "can't find how to do X" usually has a *data* problem, not a search problem. The pipeline that feeds retrieval is its own discipline:

- **Ingestion** — what sources, how often, and how changes are picked up.
- **Chunking strategy** — how documents are split; bad chunking shreds meaning and tanks recall.
- **Embedding choice** — the embedding model determines what "similar" means; pick deliberately.
- **Freshness / re-indexing** — stale indexes silently serve wrong answers; decide a refresh cadence.
- **Quality & dedup** — deduplicate, remove noise, and curate; one authoritative copy beats ten near-duplicates.

So "where does RAG go?" → it's the retrieval layer the pillars are built on, *and* the data pipeline that keeps that layer worth querying. Design both once, well.

The mental model: Retrieval is the *data* substrate; [Gateways](13-gateways.md) are the *control* substrate; [Cost & Latency](14-cost-latency.md) is the budget.

---
[← Prev: Runtime & Deployment](11-runtime-deployment.md) · [Overview](00-overview.md) · [Next: Gateways →](13-gateways.md)
