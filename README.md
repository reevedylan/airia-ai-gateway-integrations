# Airia AI Gateway Integrations

This repository documents how to connect developer tools, editors, and SDKs to the **Airia AI Gateway**. If you're looking for how to wire up Claude Code, Cursor, an SDK, or a specific tool + model provider combination, you're in the right place.

New to the Airia AI Gateway? Start with [`docs/what-is-ai-gateway.md`](docs/what-is-ai-gateway.md).

## Quick example

Point the OpenAI SDK at the Airia AI Gateway by changing its base URL and API key:

```python
import openai

# Configure client to use the Airia AI Gateway
client = openai.OpenAI(
    base_url="https://prodaus.gateway.airia.ai/openai/v1",
    api_key="<YOUR-AIRIA-AI-GATEWAY-API-KEY>"  # Replace with your Airia AI Gateway API key.
)

# Make requests as usual
response = client.responses.create(
    model="gpt-6.1-sol",
    input="In one sentence, explain what an enterprise AI gateway does."
)

print(response.output_text)
```

Replace the base URL with your tenant's Gateway URL from the Airia dashboard.

## How this repo is organized

- **[`cli-agents/`](cli-agents/)**: CLI-based coding agents (Claude Code, Codex CLI, Gemini CLI, etc.)
- **[`editors/`](editors/)**: IDEs and editor integrations (Cursor, VS Code, Zed, etc.)
- **[`sdks/`](sdks/)**: Language SDKs and frameworks (OpenAI SDK, Anthropic SDK, LangChain, etc.)
- **[`agent-platforms/`](agent-platforms/)**: AI and agent platforms that use the gateway for model access (Microsoft Foundry, etc.)
- **[`runbooks/`](runbooks/)**: Specific tool + model provider recipes that need extra configuration beyond the generic setup (e.g. a CLI agent routed through a particular cloud provider)

## CLI Agents

| Integration | Docs |
|---|---|
| _No integrations documented yet_ | |

## Editors

| Integration | Docs |
|---|---|
| _No integrations documented yet_ | |

## SDKs

| Integration | Docs |
|---|---|
| _No integrations documented yet_ | |

## Agent Platforms

| Integration | Docs |
|---|---|
| Microsoft Foundry (Azure AI Foundry) | [Setup guide](agent-platforms/microsoft-foundry/README.md) |

## Runbooks

| Runbook | Docs |
|---|---|
| _No runbooks documented yet_ | |
