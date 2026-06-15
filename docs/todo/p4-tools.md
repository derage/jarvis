# TODO: P4 — Tools / Actions

The registry and base class are done. Now build the actual tools.

## Built-in tools to create (`src/jarvis/tools/builtin/`)

Priority order — build these first:

- [ ] **`web_search.py`** — Search the web for current info
  - Provider: Tavily (`pip install tavily-python`) or SerpAPI
  - Inputs: `query: str`, `max_results: int = 5`
  - Returns: list of `{title, url, snippet}` dicts
  - Config: `TAVILY_API_KEY` in `.env`

- [ ] **`calendar.py`** — Read and create calendar events
  - Provider: Google Calendar API or local iCal
  - Inputs: `action: "list"|"create"`, date range / event details
  - Gate creates/deletes behind `requires_approval = True`

- [ ] **`files.py`** — Read files from Jesse's filesystem
  - Inputs: `path: str`, `action: "read"|"list"`
  - Scope to a safe root dir (P9: least privilege)
  - Write operations require approval

- [ ] **`memory_search.py`** — Expose memory retrieval as a tool
  - Lets the agent explicitly search memory mid-task
  - Inputs: `query: str`, `memory_type: "episodic"|"semantic"|"procedural"`
  - Thin wrapper over P3 memory interfaces

- [ ] **`notes.py`** — Read/write the agent's scratchpad
  - Lets the agent persist notes that survive context compaction
  - Wraps `context/notes.py`

## `agents/personal/agent.py`

- [ ] Return adapted tools from `_langchain_tools()` (BaseTool → LangChain tool);
      `build_personal_graph()` passes them to `create_deep_agent(tools=...)`
- [ ] Derive `interrupt_on` from each tool's `requires_approval` (P6/P9)

## `src/jarvis/tools/builtin/__init__.py`

- [ ] Auto-register all built-in tools on import

## Tests

- [ ] `tests/unit/tools/test_web_search.py` — mock HTTP, assert structured ToolResult
- [ ] `tests/unit/tools/test_files.py` — assert path scoping rejects outside-root paths
- [ ] `tests/integration/tools/test_web_search.py` — real API call (needs key)

## ACI checklist for every new tool

- [ ] Name is unique and unambiguous
- [ ] Description is precise enough a human picks the right tool every time
- [ ] `_validate()` raises `ToolValidationError` with an actionable message
- [ ] Side effects are idempotent OR `requires_approval = True`
- [ ] ToolResult on failure has a message that tells the agent how to fix it
