# TODO: First Run — Get Jarvis Talking

The minimum work to get a real conversation working end-to-end.
Do these in order before tackling per-pillar depth.

## Step 1: Install and configure (30 min)

- [ ] Run `make install` (requires **Python 3.13+**)
- [ ] Copy `.env.example` → `.env`, fill in `ANTHROPIC_API_KEY`
- [ ] Run `make dev` — confirm LangGraph server starts at `http://localhost:2024`
- [ ] Open https://agentchat.vercel.app, connect to `http://localhost:2024`, graph ID: `jarvis`
- [ ] Send "hello" — confirm a response comes back (even if minimal)

## Step 2: Verify the agent harness (1 hour)

The DeepAgents harness IS the registered graph (`agents/personal/agent.py`,
`build_personal_graph`). Built-in tools (write_todos, virtual filesystem, execute)
ship with the harness; `execute` is gated behind HITL.

- [ ] Confirm a real LLM response via Agent Chat UI (Step 1)
- [ ] Confirm `BudgetGuard` halts long runs at `JARVIS_MAX_STEPS`
- [ ] Confirm a gated `execute` call surfaces a pending approval in state

## Step 3: Add a custom tool (1-2 hours)

- [ ] Build `tools/builtin/web_search.py` using Tavily or SerpAPI
- [ ] Adapt `BaseTool` → LangChain tool and register in `tools/registry.py`
- [ ] Pass custom tools via `_langchain_tools()` in `agent.py`
- [ ] Test: "what's the weather in Chicago?" → agent calls search → returns result

## Step 4: Add memory (half day)

- [ ] Pick LangGraph Memory Store as the backend
- [ ] Implement `SemanticMemory` with it
- [ ] Wire `ContextLoader.before_agent` (agent.py) to load user facts into context
- [ ] Run `make seed-memory` with a few initial facts
- [ ] Test: "what do you know about me?" → agent retrieves and lists facts

## Step 5: Wire triggers (half day)

- [ ] Implement `_run_agent()` in `orchestration/triggers.py` via LangGraph SDK
- [ ] Wire Slack client in `integrations/slack/client.py` (`pip install -e ".[channels]"`)
- [ ] Wire `handle_approval_response()` → runtime `Command(resume=...)` (native resume)

## Step 6: Verify tests pass

- [ ] `pytest` — unit tests should pass (coverage threshold may still fail until stub modules are tested)
- [ ] Fix any behavioral failures before adding more features

---

After these steps, Jarvis can have a real conversation, remember facts, search the web,
and resume after human approval. Everything else builds on top of this foundation.
