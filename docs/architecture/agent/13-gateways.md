[← Overview](00-overview.md) · Agent Architecture Blueprint

# Cross-cutting — Gateways

**Not a pillar — the infrastructure your model calls and tool calls are routed through.** A gateway sits between the agent and something it depends on, centralizing concerns you don't want scattered across every agent. Two kinds matter:

**Model gateway / LLM router** (e.g. OpenRouter, LiteLLM, Portkey). Sits in front of the models and routes each request to the right one. The high-value capability is **routing by what's coming in** — pick a cheap/fast model (or SLM) for easy requests and a stronger one for hard ones, fall back automatically when a provider is down or rate-limited, and present one API across many providers. This is how you *operationalize* the [Model](02-model-reasoning.md) decision ("which model does what," including LLM↔SLM) without hardcoding it, and it centralizes cost control, rate limiting, and per-request logging.
- *Touches:* **[Model](02-model-reasoning.md)** (routing, fallback), **[Guardrails](10-guardrails-safety.md)** (rate/cost limits), **[Eval](08-evaluation-observability.md)** (one place to log every model call), **[Cost & Latency](14-cost-latency.md)** (the main routing lever).

**MCP gateway** (a proxy in front of your MCP servers). A single control point for the tool layer: **authentication and authorization** (who/what may call which server), **auditing** (a log of every tool invocation), rate limiting, and tool **filtering/governance** — which also curbs context bloat by exposing only the tools an agent actually needs.
- *Touches:* **[Tools / Actions](05-tools-actions.md)** (single endpoint, tool filtering), **[Guardrails](10-guardrails-safety.md)** (authn/authz, audit trail), **[Context](03-context.md)** (expose fewer tools → fewer tokens).

The mental model: [Retrieval](12-retrieval-rag.md) is the *data* substrate; Gateways are the *control* substrate. Both are shared infrastructure you build once and route many agents through.

## What a gateway is NOT

A gateway is **egress control for the agent's capability dependencies** — the models and tool/MCP servers it calls *out* to. It is **not**:

- **Inbound triggers** — Slack/webhook/cron events that *start* a run live in `orchestration/triggers.py` (12-factor #11), not here.
- **Human channels** — posting to Slack/email and rendering the approval inbox are [Human Interaction](07-human-interaction.md) (P6), under `interaction/channels/`.
- **Transport/SDK plumbing** — speaking a service's protocol is the `integrations/` ring (dumb adapters). A gateway adds *policy* (routing, authz, rate-limit) and may sit *in front of* an adapter.

**Litmus test:** if the agent is calling *out to a model or a tool* under policy, it's a gateway. If a *human or an external event* is reaching *in*, or you're just speaking a service's protocol, it isn't.

---
[← Prev: Retrieval (RAG)](12-retrieval-rag.md) · [Overview](00-overview.md) · [Next: Cost & Latency →](14-cost-latency.md)
