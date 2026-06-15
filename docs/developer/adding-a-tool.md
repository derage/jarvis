# Adding a Tool

Tools are how Jarvis acts on the world. Each tool is a class that extends `BaseTool`.

## 1. Create the tool file

```python
# src/jarvis/tools/builtin/weather.py
from jarvis.tools.base import BaseTool, ToolResult
from jarvis.core.errors import ToolValidationError


class WeatherTool(BaseTool):
    name = "get_weather"
    description = (
        "Get current weather for a city. "
        "Use when the user asks about weather, temperature, or conditions. "
        "Input: city name as a string."
    )

    def _validate(self, kwargs):
        if not kwargs.get("city"):
            raise ToolValidationError("'city' is required. Example: get_weather(city='Chicago')")

    async def _run(self, city: str) -> ToolResult:
        # Your implementation here
        data = {"city": city, "temp_f": 72, "condition": "Sunny"}
        return ToolResult(success=True, data=data)
```

## 2. Register the tool

```python
# src/jarvis/tools/builtin/__init__.py  (or in your agent's setup)
from jarvis.tools.registry import register
from jarvis.tools.builtin.weather import WeatherTool

register(WeatherTool())
```

## 3. Add it to the agent

Custom tools are surfaced to the harness via `_langchain_tools()` in
`agents/personal/agent.py`. (The built-in harness tools — `write_todos`, the
virtual filesystem, `task`, `execute` — are always present; this adds your
*domain* tools.)

```python
# agents/personal/agent.py
def _langchain_tools() -> list:
    from jarvis.tools.registry import get
    # NOTE: adapt jarvis BaseTool → LangChain tool before returning (see the
    # _langchain_tools TODO). build_personal_graph passes these to create_deep_agent.
    return [get("get_weather"), get("web_search")]
```

## Design checklist (P4 — ACI patterns)

- [ ] `description` is precise enough that a human (and agent) could pick the right tool every time
- [ ] `_validate` raises `ToolValidationError` with an actionable message on bad input
- [ ] `_run` returns `ToolResult(success=False, error="...")` on failure — never raises silently
- [ ] Side effects are idempotent, OR `requires_approval = True` is set
- [ ] The tool name doesn't overlap with existing tools (`make tool-list` to check)

## Testing tools

```python
# tests/unit/tools/test_weather.py
import pytest
from jarvis.tools.builtin.weather import WeatherTool

@pytest.mark.asyncio
async def test_weather_returns_data():
    tool = WeatherTool()
    result = await tool(city="Chicago")
    assert result.success
    assert result.data["city"] == "Chicago"

async def test_weather_validates_input():
    tool = WeatherTool()
    result = await tool()  # missing city
    assert not result.success
    assert "city" in result.error
```
