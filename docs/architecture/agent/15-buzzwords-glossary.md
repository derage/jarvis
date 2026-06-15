[← Overview](00-overview.md) · Agent Architecture Blueprint

# Where the buzzwords fit

A reference for the terms that get thrown around, and which pillars each one actually maps to. The recurring lesson: most of these are not new pillars — they're either a *toolkit* you build the pillars with, an *integrated bundle* of pillars you already have, or a *protocol* that lives inside one pillar.

## Agentic framework

Two unrelated meanings. Know which one someone means.

**(a) Architecture sense — what builders mean.** The libraries/SDKs you build agents *with*: LangGraph/LangChain, CrewAI, AutoGen, Semantic Kernel, plus first-party SDKs (Claude Agent SDK, OpenAI Agents SDK, Google ADK). They exist to fight fragmentation — an agent written in one framework can't run in another without a rewrite.
- **Where it maps:** *not a pillar.* A cross-cutting toolkit that hands you ready-made [Orchestration](06-orchestration.md), often [Memory](04-memory.md) and [Tools](05-tools-actions.md), sometimes [Context](03-context.md) and [Runtime](11-runtime-deployment.md). It's "how you build the pillars."
- **Tradeoff:** accelerates the start, adds abstraction. Lean on it early; peel abstraction back in production.

**(b) Marketing sense — a false friend.** Some articles (e.g. the HackerNoon "A.G.E.N.T.I.C. Framework") use the term for a *brand-visibility / agentic-commerce* methodology — getting your brand surfaced and sold by AI shopping agents. Nothing to do with architecture. If it's about commerce or search optimization, it's this sense — ignore it for build purposes.

## Agent OS (agentic operating system)

A coordination/runtime layer that manages memory, tools, scheduling, and access control for agents, using the operating-system metaphor:

| OS concept | Agent equivalent |
|---|---|
| Kernel / scheduler | the loop + scheduling of work (cron-like) |
| Filesystem | persistent memory |
| Processes | sub-agents |
| Applications | skills |
| Drivers / syscalls | tools + gateways |

Key framing: **the model is not the OS — it's one component the OS schedules.** Same stance as this blueprint, where [Model](02-model-reasoning.md) is orchestrated, not the orchestrator. (Examples: AIOS from Rutgers, Microsoft reframing Windows as an agentic OS, PwC's cross-vendor Agent OS, Letta's "LLM-as-OS.")
- **Where it maps:** *not a new pillar — the integrated form of several.* Essentially [Runtime](11-runtime-deployment.md) + [Memory](04-memory.md) + [Orchestration](06-orchestration.md) + [Tools](05-tools-actions.md) + [Guardrails](10-guardrails-safety.md) + [Gateways](13-gateways.md), bundled as a managed layer.
- **The decision it poses:** build-vs-buy. Write your own scheduler and memory manager, or adopt a runtime/OS that provides those primitives. A framework and an OS are complementary: framework = how you express the logic; OS = the runtime that schedules and persists it.

## MCP vs. A2A (the two protocols)

Different problems, different pillars:
- **MCP (Model Context Protocol)** — *agent-to-tool.* How an agent connects to tools and data. Lives in [Tools](05-tools-actions.md); governed by an [MCP gateway](13-gateways.md).
- **A2A (Agent2Agent)** — *agent-to-agent.* Open standard (Google → Linux Foundation) for *independent* agents from different teams/vendors to discover and delegate across ownership boundaries, via Agent Cards + Tasks/Messages. Like HTTP for agents. Relevant to [Orchestration](06-orchestration.md)/[Runtime](11-runtime-deployment.md) only across an ownership boundary.
- **Message queue (Kafka, etc.)** — *not a protocol for agents*; transport between *your own* components. The [Runtime](11-runtime-deployment.md) scaling fix. Don't reach for A2A to solve an internal process-topology problem.

## Quick map

| Buzzword | Is it a pillar? | Maps to |
|---|---|---|
| Agentic framework (arch) | No — a toolkit across pillars | Orchestration, Memory, Tools (+ Context/Runtime) |
| Agentic framework (marketing) | No — unrelated | brand/commerce, not architecture |
| Agent OS | No — an integrated bundle | Runtime + Memory + Orchestration + Tools + Guardrails + Gateways |
| MCP | Lives inside a pillar | Tools / Actions (+ Gateways) |
| A2A | Lives inside a pillar | Orchestration / Runtime, across boundaries |
| Message queue | Lives inside a pillar | Runtime & Deployment |

---
[← Prev: Cost & Latency](14-cost-latency.md) · [Overview](00-overview.md)
