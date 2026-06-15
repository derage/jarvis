# TODO: CX — Retrieval / RAG

`retrieval/` is an empty stub. This is the data substrate for P3 (Memory) and P2 (Context).

## `src/jarvis/retrieval/`

- [ ] **`chunker.py`** — Document chunking
  - Implement recursive character splitting (LangChain `RecursiveCharacterTextSplitter`)
  - Configurable chunk size and overlap
  - Preserve document metadata (source URL, timestamp) on each chunk

- [ ] **`embedder.py`** — Embedding interface
  - Default: `text-embedding-3-small` (cheap, fast, good)
  - Wrap in a thin interface so the model can be swapped via config
  - Cache embeddings for chunks that haven't changed (cost reduction)

- [ ] **`store.py`** — Vector store interface
  - Abstract interface: `add(chunks)`, `search(query, top_k)`, `delete(ids)`
  - Implement `ChromaStore` (local dev)
  - Plan for `PgVectorStore` (production)
  - Config-driven via `JARVIS_VECTOR_STORE`

- [ ] **`pipeline.py`** — Ingestion pipeline
  - `ingest(source: str | Path)` — loads, chunks, embeds, stores
  - Handles URLs, local files, raw text
  - Tracks what's been ingested so re-runs skip unchanged content

## Freshness

- [ ] Add a `last_indexed_at` timestamp to each stored chunk
- [ ] Add `make reindex` command for refreshing stale content
- [ ] Log when a query returns chunks older than a configurable threshold

## Tests

- [ ] `tests/unit/test_retrieval.py`
  - `test_chunker_splits_on_paragraphs`
  - `test_chunker_preserves_metadata`
- [ ] `tests/integration/test_vector_store.py`
  - `test_store_and_retrieve_by_similarity`
  - `test_delete_removes_from_search_results`
