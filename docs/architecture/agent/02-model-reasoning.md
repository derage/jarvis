[← Overview](00-overview.md) · Agent Architecture Blueprint

# Pillar 1 — Model / Instruction / Reasoning Core

**What it is:** The LLM(s) doing the planning and execution, *how you instruct them* (the system prompt), the reasoning strategy, and how work is split across models. This is the center of the wheel — but a stronger model never fixes a weak architecture; it raises the ceiling, not the floor.

## Instruction design (the system prompt)

The system prompt is the single highest-leverage artifact for behavior. It physically lives in [Context](03-context.md), but it's how you *steer* the model, so it belongs here.

The craft is hitting the **right altitude** — a Goldilocks zone between two failures:
- *Too low:* brittle, hardcoded if-this-then-that logic the model can't generalize from.
- *Too high / too broad:* vague hand-waving that gives no real steer, so the model has to guess what you want and flails. "Broad" lives at this end — it's why an over-general prompt makes the model worse, not more flexible.

Aim for the middle: clear, scoped, direct instructions that say *how to behave* and *where the boundaries are* — without scripting every branch. Use structure and concrete examples. Narrow and specific beats broad and permissive.

## Model choice — and the LLM↔SLM spectrum

Don't reflexively default to a frontier LLM. The model is a *spectrum*, and most agentic calls are narrow and repetitive — follow a fixed prompt, return JSON, call a tool. NVIDIA's position paper *"Small Language Models are the Future of Agentic AI"* (Belcak et al., 2025) argues SLMs are sufficiently powerful, more suitable, and more economical for many agentic invocations: a fine-tuned sub-10B model can match or beat an LLM on a narrow task at roughly 10–30× lower cost/latency, can be fine-tuned in hours, and can run locally/on-prem (privacy, sovereignty, no cloud dependency).

The shape this implies is a **heterogeneous, "Lego-style" system**: reserve a strong generalist LLM for the decide/plan moments, and use specialized SLMs everywhere for the repetitive errands. You route between them via the [model gateway](13-gateways.md), and you specialize the SLMs from your own interaction logs via the [Improvement Loop](09-improvement-loop.md). This is the concrete cash-out of the blueprint's framing principle: do the other pillars well and you can slide down to a smaller, cheaper, local model.

## Modality

If the agent perceives or produces beyond text — vision, voice, files, or driving a screen (computer use) — that shapes model choice, what goes in [Context](03-context.md), and the [Tools](05-tools-actions.md) it needs. Decide the modalities up front; they ripple.

**Simple vs. complex:**
- *Simple:* one capable model handles everything, text-only, with a clean scoped prompt.
- *Complex:* tiered/heterogeneous models (generalist planner + fine-tuned SLM workers), multimodal I/O, automated prompt management.

**Architect's checklist:**
- [ ] Is the system prompt at the right altitude — neither brittle hardcoded logic nor vague breadth?
- [ ] Which model plans? Which executes? Could narrow steps run on a cheaper/fine-tuned SLM?
- [ ] Is there a path to specialize SLMs from production logs, and to run them locally if it helps?
- [ ] What's the fallback when the primary model is unavailable or rate-limited?
- [ ] What modalities (text/vision/voice/files/screen) does this actually need?

**Failure mode:** Reaching for a bigger model (or a broader prompt) to paper over a problem that's actually in context, tools, orchestration, or instruction design.

---
[← Prev: Should this be an agent?](01-should-this-be-an-agent.md) · [Overview](00-overview.md) · [Next: Context →](03-context.md)
