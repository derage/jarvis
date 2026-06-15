# TODO: P3 — Memory

This is the deepest pillar. Start with semantic memory (facts/preferences),
then episodic (past experiences), then procedural (reusable skills).

## Backend: pick one and implement

- [ ] **Option A (recommended first):** LangGraph Memory Store
  - Built into LangGraph, works with `MemorySaver` or Postgres backend
  - Docs: https://docs.langchain.com/oss/python/langgraph/add-memory
  - Implement: `memory/backends/langgraph_store.py`

- [ ] **Option B:** Chroma (local vector DB)
  - `pip install chromadb`
  - Good for local dev, easy to swap later
  - Implement: `memory/backends/chroma.py`

## `src/jarvis/memory/semantic.py`

- [ ] Implement `SemanticMemory` backed by chosen backend
- [ ] `store()` — upsert by (subject, predicate) — newer value wins
- [ ] `retrieve(query, top_k)` — similarity search, returns ranked Facts
- [ ] `get_by_subject(subject)` — exact match for loading all user prefs
- [ ] `delete(fact_id)` — remove stale/wrong facts
- [ ] Add conflict resolution: log a warning when overwriting an existing fact

## `src/jarvis/memory/episodic.py`

- [ ] Implement `EpisodicMemory` with similarity search over `content` field
- [ ] `store(episode)` — persist with timestamp and tags
- [ ] `retrieve(query, top_k)` — vector search over episode content
- [ ] `forget(episode_id)` — hard delete (user-requested forgetting)

## `src/jarvis/memory/procedural.py`

- [ ] Implement `ProceduralMemory` — search over skill `description` field
- [ ] `store(skill)` — upsert by name
- [ ] `retrieve(task_description, top_k)` — find relevant skills before planning
- [ ] `record_success(skill_id)` — increment success_count (feeds P8 improvement)

## `src/jarvis/memory/consolidation.py`

- [ ] Wire up LLM call (worker_model) to extract memories from session messages:
  - Extract facts → `semantic_memory.store()`
  - Extract experiences → `episodic_memory.store()`
  - Extract successful plans → `procedural_memory.store()`
- [ ] Add forgetting: after consolidation, flag low-confidence facts for review
- [ ] Add dedup: don't store an episode if a nearly-identical one already exists

## `agents/personal/agent.py` — `ContextLoader` middleware

- [ ] `ContextLoader.before_agent`: query semantic memory for user facts
- [ ] `ContextLoader.before_agent`: query episodic memory for relevant experiences
- [ ] `ContextLoader.before_agent`: query procedural memory for relevant skills
- [ ] Inject retrieved memories into context (the `{user_context}` slot / a context message)

## `scripts/seed_memory.py`

- [ ] Add Jesse's initial facts to `INITIAL_FACTS` (timezone, preferences, project paths, etc.)
- [ ] Wire `SemanticMemory.store()` call
- [ ] Run `make seed-memory` and verify

## Tests

- [ ] `tests/unit/test_memory.py`
  - `test_semantic_store_and_retrieve`
  - `test_semantic_conflict_resolution` — newer value wins
  - `test_episodic_retrieve_by_similarity`
  - `test_procedural_success_count_increments`
- [ ] `tests/integration/test_memory_backends.py` (requires real vector store)
