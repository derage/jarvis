# TODO: NotebookLM Research Notebooks

One notebook per pillar — upload the pillar's `.md` file + run Fast Research with the query below.

The pillar `.md` files are all in `docs/architecture/agent/`.

## Status

| Notebook | Source File | Fast Research Query | Status |
|---|---|---|---|
| Jarvis \| P0 – Should This Be an Agent? | `01-should-this-be-an-agent.md` | "When to use AI agents vs. workflows vs. single LLM calls: decision frameworks, real-world failure modes of over-agentification, agentic theater anti-pattern, cost-reliability tradeoffs..." | ✅ Done |
| Jarvis \| P1 – Model / Instruction / Reasoning | `02-model-reasoning.md` | "System prompt engineering for AI agents: right instruction altitude, SLM vs frontier LLM for agentic tasks, heterogeneous model routing, fine-tuning small models from production logs" | ⏳ Source uploaded, query submitted |
| Jarvis \| P2 – Context | `03-context.md` | "AI agent context window engineering: just-in-time dynamic loading vs preloading, context rot causes and fixes, compaction strategies for long-horizon tasks, structured notes pattern, token budget management, MCP tool bloat" | ⬜ Not started |
| Jarvis \| P3 – Memory | `04-memory.md` | "AI agent long-term memory: episodic, semantic, procedural memory taxonomy, consolidation and forgetting strategies, conflict resolution, sleep-time compute, MemGPT Letta Mem0 Zep LangMem comparison for personal assistants" | ⬜ Not started |
| Jarvis \| P3 – Memory (bonus) | — | "MemGPT architecture deep dive: working memory management, sleep-time consolidation, context window as RAM analogy, comparison with Mem0 and Zep for production personal assistants" | ⬜ Add as 2nd query in P3 notebook |
| Jarvis \| P4 – Tools / Actions | `05-tools-actions.md` | "AI agent tool design best practices: agent-legible APIs, Agent-Computer Interface ACI, MCP protocol design, preventing tool sprawl, idempotent tool design, actionable error messages for self-correcting agents" | ⬜ Not started |
| Jarvis \| P5 – Orchestration | `06-orchestration.md` | "AI agent loop design: ReAct continue-by-default vs terminate-by-default, todo-list driven task queues, multi-agent coordination patterns, ambient agents event-driven triggers, sleep-time compute, LangGraph state machines" | ⬜ Not started |
| Jarvis \| P6 – Human Interaction | `07-human-interaction.md` | "Human-in-the-loop AI agent UX: agent inbox approval patterns, streaming reasoning visibility, mid-run interruption and steering, trust and transparency design, graceful handoff to humans, oversight UI patterns" | ⬜ Not started |
| Jarvis \| P7 – Eval & Observability | `08-evaluation-observability.md` | "AI agent evaluation: trajectory-level eval beyond final output, tool call accuracy metrics, LangSmith Langfuse Braintrust Arize Phoenix comparison, eval suites in CI/CD pipelines, production trace sampling and drift detection" | ⬜ Not started |
| Jarvis \| P8 – Improvement Loop | `09-improvement-loop.md` | "AI agent self-improvement from production: capturing explicit and implicit feedback, trace-to-dataset pipelines, automated prompt optimization DSPy, fine-tuning from interaction logs, SLM specialization from production clusters" | ⬜ Not started |
| Jarvis \| P9 – Guardrails & Safety | `10-guardrails-safety.md` | "AI agent safety and security: prompt injection defense, indirect injection attacks, confused deputy problem, execution sandboxing strategies, least privilege tool permissions, circuit breaker patterns, blast radius management for autonomous agents" | ⬜ Not started |
| Jarvis \| P10 – Runtime & Deployment | `11-runtime-deployment.md` | "AI agent production deployment: state-compute separation, durable execution Temporal DBOS Inngest vs checkpointing, stateless worker pools, queue-based intake autoscaling, idempotent node design, canary shadow releases for agents" | ⬜ Not started |
| Jarvis \| CX – Retrieval / RAG | `12-retrieval-rag.md` | "RAG for AI agents: chunking strategies and tradeoffs, embedding model selection, hybrid dense-sparse search, retrieval quality evaluation, freshness and re-indexing cadence, deduplication for knowledge bases" | ⬜ Not started |
| Jarvis \| CX – Gateways | `13-gateways.md` | "AI agent gateway architecture: LLM router LiteLLM OpenRouter Portkey comparison, MCP gateway proxy auth and auditing, model routing by request complexity, fallback strategies, tool filtering to reduce context bloat" | ⬜ Not started |
| Jarvis \| CX – Cost & Latency | `14-cost-latency.md` | "AI agent cost and latency optimization: cost-per-task budgeting, caching model calls and retrieval results, SLM routing for cheap repetitive tasks, sleep-time compute for amortizing reasoning, token reduction strategies per pillar" | ⬜ Not started |

## How to create each notebook

1. Go to https://notebooklm.google.com → **Create notebook**
2. Dismiss the overlay (click X)
3. Triple-click the title → type the notebook name from the table above
4. **Add sources** → **Upload files** → upload the source `.md` file from `docs/architecture/agent/`
5. Wait for ingestion, then paste the Fast Research query into the search bar
6. Switch to **Fast Research** tab → click the blue arrow
7. Wait for "Fast Research completed!" → click **Import**
