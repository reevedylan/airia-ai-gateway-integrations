# Airia AI Gateway Integrations

This repository documents how to connect developer tools, editors, and SDKs to the **Airia AI Gateway**. If you're looking for how to wire up Claude Code, Cursor, an SDK, or a specific tool + model provider combination, you're in the right place.

New to the Airia AI Gateway? Start with [`docs/what-is-ai-gateway.md`](docs/what-is-ai-gateway.md).

## Before / after Airia

**Before** — calling a provider directly:

```python
from openai import OpenAI

client = OpenAI(api_key="sk-...")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
)
```

**After** — the same code, routed through the Airia AI Gateway:

```python
from openai import OpenAI

client = OpenAI(
    api_key="<YOUR_AIRIA_API_KEY>",
    base_url="<YOUR_AIRIA_GATEWAY_URL>",
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
)
```

Only the client configuration changes — swap in your Airia API key and gateway URL, and the rest of your code stays the same.

<!-- TODO: confirm this example accurately reflects how a client points at the Airia AI Gateway before publishing. -->

## How this repo is organized

- **[`cli-agents/`](cli-agents/)** — CLI-based coding agents (Claude Code, Codex CLI, Gemini CLI, etc.)
- **[`editors/`](editors/)** — IDEs and editor integrations (Cursor, VS Code, Zed, etc.)
- **[`sdks/`](sdks/)** — Language SDKs and frameworks (OpenAI SDK, Anthropic SDK, LangChain, etc.)
- **[`runbooks/`](runbooks/)** — Specific tool + model provider recipes that need extra configuration beyond the generic setup (e.g. a CLI agent routed through a particular cloud provider)

Each category holds one generic "how to point this tool at the Airia AI Gateway" doc per tool. Runbooks only exist for tool + provider combinations that need non-obvious extra steps — not every combination gets one.

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for how to add a new integration.

## CLI Agents

| Integration | Status | Docs |
|---|---|---|
| _No integrations documented yet_ | | |

## Editors

| Integration | Status | Docs |
|---|---|---|
| _No integrations documented yet_ | | |

## SDKs

| Integration | Status | Docs |
|---|---|---|
| _No integrations documented yet_ | | |

## Runbooks

| Runbook | Status | Docs |
|---|---|---|
| _No runbooks documented yet_ | | |

## Contributing

Contributions are welcome — see [`CONTRIBUTING.md`](CONTRIBUTING.md) for the folder structure, templates, and PR checklist.
