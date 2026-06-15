# Getting Started

## Prerequisites

- Python 3.13+ (matches `requires-python` in `pyproject.toml`)
- An Anthropic or OpenAI API key (or both)

## First-time setup

```bash
# 1. Clone and enter the repo
git clone <repo-url>
cd jarvis

# 2. Install dependencies
make install
# This also copies .env.example → .env

# 3. Fill in your API keys
open .env   # or edit with your editor

# 4. Start the dev server
make dev
# LangGraph server starts at http://localhost:2024

# 5. Open the chat UI (optional)
make dev-ui
# Opens https://agentchat.vercel.app — point it at http://localhost:2024
```

## Common commands

| Command | What it does |
|---|---|
| `make dev` | Start LangGraph dev server with hot-reload |
| `make test` | Run all tests |
| `make check` | Lint + typecheck before committing |
| `make format` | Auto-format code |
| `make eval` | Run the agent eval suite |
| `make seed-memory` | Seed initial facts into Jarvis memory |
| `make clear-memory` | Wipe all stored memories (destructive — prompts first) |
| `make help` | Show all available commands |

## Environment variables

All configuration comes from environment variables (12-factor III).
See `.env.example` for the full list with descriptions.
Never commit `.env` to git.
