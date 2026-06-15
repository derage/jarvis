# TODO: P1 — Model / Instruction / Reasoning

## `src/jarvis/model/router.py`

- [ ] Add real routing logic — classify request complexity before deciding model
- [ ] Add cost-aware routing: check remaining budget before routing to frontier model
- [ ] Add fallback: if planner_model is unavailable, fall back to default_model
- [ ] Wire up the model gateway (CX-Gateways) so routing goes through a single proxy

## `src/jarvis/model/prompts.py`

- [ ] Add prompt versioning — track which prompt version was used in each trace (P7)
- [ ] Add template variable validation — raise early if a required `{var}` is missing

## `prompts/personal/system.md`

- [ ] Fill in `{user_context}` injection with real retrieved memory at runtime (P3)
- [ ] Iterate on instruction altitude — test and refine after first real conversations
- [ ] Add examples of good vs bad responses inline in the prompt (few-shot)

## General

- [ ] Add `model/providers.py` — provider client initialization (Anthropic, OpenAI, OpenRouter)
- [ ] Test prompt loading in `tests/unit/test_model.py`
- [ ] Document the "right altitude" principle with before/after examples in `docs/developer/`
