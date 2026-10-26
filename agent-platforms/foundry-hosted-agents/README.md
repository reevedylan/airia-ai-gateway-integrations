# Microsoft Foundry Hosted Agents (Azure AI Foundry)

Microsoft Foundry is Azure's platform for building and running AI agents. This guide deploys a Foundry **hosted agent** that sends its model requests through the Airia AI Gateway, using the Anthropic Python SDK.

## TL;DR

Hosted agents are your own code, packaged as a container and run by Foundry Agent Service. Because you control the code, you point your model SDK at the Airia AI Gateway directly. No admin-connected model is required.

| Framework / SDK | Protocol | Status | Notes |
|---|---|---|---|
| Anthropic Python SDK | Invocations | ✅ Supported | Change the base URL and API key only. |
| Microsoft Agent Framework | Responses | ❓ Untested | |
| Other frameworks (LangGraph, OpenAI Agents SDK, etc.) | Any | ❓ Untested | Should work with the same base URL and API key change. |

> **Note:** The rest of this guide uses the Anthropic Python SDK with the invocations protocol.

## Overview

A hosted agent calls the Airia AI Gateway in the same way any other application does. Only two values change in your code:

- **Base URL:** your Airia AI Gateway URL, instead of `api.anthropic.com`
- **API key:** your Airia AI Gateway API key, instead of an Anthropic API key

Every request is covered by Airia's guardrails, data loss prevention, and observability. Foundry's own **AI Gateway** (under **Manage** > **AI Gateway**) is not involved and doesn't need to be enabled.

## Prerequisites

- An Airia account with access to the AI Gateway
- An Airia AI Gateway API key
- A Microsoft Foundry project in a region that supports hosted agents, with the **Foundry Project Manager** role on the project
- Python 3.10 or later
- [Docker Desktop](https://docs.docker.com/get-docker/)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) version 2.80 or later
- An Azure Container Registry, or permission to create one

## Setup

### Part 1: Configure the Airia AI Gateway

1. In Airia, go to **Gateway** > **AI Gateway** and open (or create) the configuration you'll use for Foundry. On the **Models** tab, add the Anthropic models your agent will use (for example, `claude-sonnet-5-5`).
2. Copy the Anthropic base URL from the configuration's Python snippet. The Anthropic SDK adds `/v1/messages` to the base URL itself, so don't add `/v1` as you would for a prompt agent.

### Part 2: Create the agent

3. Create a project folder with the following three files.

   **`main.py`**

   ```python
   import json
   import os

   from anthropic import AsyncAnthropic
   from azure.ai.agentserver.invocations import InvocationAgentServerHost
   from starlette.requests import Request
   from starlette.responses import JSONResponse, Response

   # Point the Anthropic SDK at Airia instead of api.anthropic.com.
   client = AsyncAnthropic(
       base_url=os.environ["AIRIA_BASE_URL"],
       api_key=os.environ["AIRIA_API_KEY"],  # Airia gateway API key, not an Anthropic key
   )

   MODEL = os.environ["ANTHROPIC_MODEL"]

   app = InvocationAgentServerHost()


   @app.invoke_handler
   async def handle(request: Request) -> Response:
       # Accept {"message": "..."} or a plain-text body.
       raw = (await request.body()).decode("utf-8", errors="replace").strip()
       try:
           data = json.loads(raw) if raw else {}
       except json.JSONDecodeError:
           data = raw
       if isinstance(data, dict):
           message = data.get("message") or data.get("input") or ""
       else:
           message = str(data)
       message = message or "Say hello and confirm you are reachable."

       try:
           result = await client.messages.create(
               model=MODEL,
               max_tokens=512,
               messages=[{"role": "user", "content": message}],
           )
       except Exception as exc:
           return JSONResponse({"error": str(exc)}, status_code=502)

       text = "".join(block.text for block in result.content if block.type == "text")
       return JSONResponse(
           {"reply": text, "model": result.model, "session_id": request.state.session_id}
       )


   if __name__ == "__main__":
       app.run()
   ```

   **`requirements.txt`**

   ```text
   anthropic
   azure-ai-agentserver-invocations
   ```

   **`Dockerfile`**

   ```dockerfile
   FROM python:3.12-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   COPY main.py .
   EXPOSE 8088
   CMD ["python", "main.py"]
   ```

### Part 3: Test locally

4. Install the dependencies and start the agent:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt

   export AIRIA_BASE_URL="<your-airia-anthropic-base-url>"
   export AIRIA_API_KEY="<your-airia-api-key>"
   export ANTHROPIC_MODEL="claude-sonnet-5-5"
   python main.py
   ```

5. In a second terminal, send a test request:

   ```bash
   curl -X POST localhost:8088/invocations \
     -H "Content-Type: application/json" \
     -d '{"message": "hello"}'
   ```

   A successful response looks like this:

   ```json
   {"reply": "Hello! How can I help you today?", "model": "claude-sonnet-5-5", "session_id": "..."}
   ```

### Part 4: Deploy to Foundry

6. Sign in to Azure and create a container registry if you don't have one:

   ```bash
   az login
   az acr create --resource-group <resource-group> --name <registry-name> --sku Basic
   ```

7. Build and push the image. Hosted agents require `linux/amd64` images, so always include `--platform`, especially on Apple Silicon:

   ```bash
   docker build --platform linux/amd64 -t airia-agent:v1 .
   az acr login --name <registry-name>
   docker tag airia-agent:v1 <registry-name>.azurecr.io/airia-agent:v1
   docker push <registry-name>.azurecr.io/airia-agent:v1
   ```

8. Grant Foundry access to the registry. In the Azure portal, open your Foundry project resource, select **Identity**, and copy the **Object (principal) ID** under **System assigned**. On the container registry, assign the **Container Registry Repository Reader** role to that ID.

9. Store the Airia API key as a project connection. In Foundry, add a **Custom keys** connection to the project named `airia-secrets`, with a key named `airia_api_key` set to your Airia AI Gateway API key. Create the connection before you deploy; otherwise the key resolves to an empty value.

10. Create `deploy.py` in the project folder:

    ```python
    import os
    import time

    from azure.ai.projects import AIProjectClient
    from azure.ai.projects.models import (
        AgentEndpointProtocol,
        ContainerConfiguration,
        HostedAgentDefinition,
        ProtocolVersionRecord,
    )
    from azure.identity import DefaultAzureCredential

    AGENT_NAME = "airia-agent"

    project = AIProjectClient(
        endpoint=os.environ["PROJECT_ENDPOINT"],
        credential=DefaultAzureCredential(),
    )

    agent = project.agents.create_version(
        agent_name=AGENT_NAME,
        definition=HostedAgentDefinition(
            protocol_versions=[
                ProtocolVersionRecord(protocol=AgentEndpointProtocol.INVOCATIONS, version="2.0.0")
            ],
            cpu="1",
            memory="2Gi",
            container_configuration=ContainerConfiguration(image=os.environ["IMAGE"]),
            environment_variables={
                "AIRIA_BASE_URL": os.environ["AIRIA_BASE_URL"],
                "AIRIA_API_KEY": "${{connections.airia-secrets.credentials.airia_api_key}}",
                "ANTHROPIC_MODEL": os.environ.get("ANTHROPIC_MODEL", "claude-sonnet-5-5"),
            },
        ),
    )
    print(f"Created {agent.name} version {agent.version}")

    while True:
        info = project.agents.get_version(agent_name=AGENT_NAME, agent_version=agent.version)
        status = info["status"]
        print(f"Status: {status}")
        if status == "active":
            print("Agent is ready.")
            break
        if status == "failed":
            print(f"Provisioning failed: {info['error']}")
            break
        time.sleep(5)
    ```

11. Run the deployment. Copy the project endpoint from the project's overview page in Foundry:

    ```bash
    pip install "azure-ai-projects>=2.3.0" azure-identity

    export PROJECT_ENDPOINT="https://<resource>.services.ai.azure.com/api/projects/<project>"
    export IMAGE="<registry-name>.azurecr.io/airia-agent:v1"
    export AIRIA_BASE_URL="<your-airia-anthropic-base-url>"
    python deploy.py
    ```

    Wait for `Agent is ready.`

### Part 5: Invoke the agent

12. Send a request to the deployed agent:

    ```bash
    TOKEN=$(az account get-access-token --resource https://ai.azure.com --query accessToken -o tsv)
    curl -X POST "$PROJECT_ENDPOINT/agents/airia-agent/endpoint/protocols/invocations?api-version=v1" \
      -H "Authorization: Bearer $TOKEN" \
      -H "Content-Type: application/json" \
      -d '{"message": "hello"}'
    ```

## Example

Every request from the hosted agent appears in Airia under **Audit** > **Gateway Monitoring**, with the guardrail policies evaluated and the full request.

## Updating the agent

To deploy a code change, build and push a new image tag (for example, `:v2`), update `IMAGE`, and run `deploy.py` again. Each run creates a new agent version. Use a unique tag for each version instead of `:latest`.

## Cleaning up

Foundry releases the agent's compute after 15 minutes of inactivity by default, so an idle agent doesn't incur compute charges. To delete the agent and all its versions:

```python
project.agents.delete(agent_name="airia-agent", force=True)
```

## Supported features

- [x] Anthropic Messages API through the Airia AI Gateway
- [x] Model switching via the `ANTHROPIC_MODEL` environment variable
- [x] API key stored as a Foundry project connection
- [ ] Streaming (not implemented in this sample)
- [ ] Tool / function calling (not tested)
- [ ] Multi-turn conversation history (not implemented in this sample)

## Limitations

- **Invocations protocol only.** This sample is tested through the REST endpoint. Foundry's agents playground and other conversational tooling are built around the responses protocol.
- **Images must be `linux/amd64`.** Images built on Apple Silicon without `--platform linux/amd64` fail to run.
- **Outbound network access is required.** The agent's container must be able to reach your Airia AI Gateway URL. If the project uses a bring-your-own virtual network, allow outbound traffic to the gateway domain.
- **Don't use `azd down` on a shared project.** It deletes every resource in the resource group, including the Foundry project. Delete the agent with the SDK instead.