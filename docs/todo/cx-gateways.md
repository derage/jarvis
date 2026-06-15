# TODO: CX — Gateways

`gateway/` is an empty stub. This is the control substrate for model routing and tool access.

## `src/jarvis/gateway/model_gateway.py`

- [ ] Implement LLM router using LiteLLM or OpenRouter
  ```bash
  pip install litellm
  ```
  - Single `chat(messages, model=None, task_type=None)` entrypoint
  - Internally calls `model/router.py` to pick the model
  - Handles rate limit retries with exponential backoff
  - Handles provider failover (Anthropic down → fall back to OpenAI)
  - Logs every call: model, token count, latency, cost estimate (P7)

- [ ] Add to config: `JARVIS_LITELLM_FALLBACK_MODEL`
- [ ] Add circuit breaker: if provider fails 3x in 60s, mark it degraded and route elsewhere

## `src/jarvis/gateway/mcp_gateway.py`

- [ ] Implement MCP proxy layer
  - Single endpoint that forwards tool calls to the right MCP server
  - Auth: verify the calling agent is allowed to use this tool
  - Audit log: every tool invocation with timestamp, caller, args (P7 / P9)
  - Tool filtering: expose only the tools an agent's config permits (reduces context bloat)

- [ ] Add `JARVIS_ALLOWED_TOOLS` env var — comma-separated list of allowed tool names

## When to build this

Gateway work pays off once you have multiple tools and/or multiple model providers.
Build `model_gateway.py` first (as soon as you add a second model), `mcp_gateway.py`
later (as soon as you have 5+ tools or need audit logging).

## Tests

- [ ] `tests/unit/test_model_gateway.py`
  - `test_routes_plan_tasks_to_planner_model`
  - `test_fallback_on_rate_limit` (mock provider failure)
- [ ] `tests/unit/test_mcp_gateway.py`
  - `test_blocks_tool_not_in_allowed_list`
  - `test_audit_log_created_on_tool_call`
