import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Safe AI Labs

## Overview

| Property | Details |
|-------|-------|
| Description | Safe AI Labs serves an OpenAI-compatible API with one endpoint per region, over OpenAI Chat Completions, OpenAI Responses and Anthropic Messages. |
| Provider Route on LiteLLM | `safeailabs/` |
| Link to Provider Doc | [Safe AI Labs Documentation ↗](https://safeailabs.ai/docs) |
| Base URL | `https://us-region.api.safeailabs.ai/v1` |
| Supported Operations | [`/chat/completions`](#usage---litellm-python-sdk), [`/responses`](#responses-api), [`/messages`](#anthropic-messages-api) |

<br />
<br />

**We support ALL Safe AI Labs models, just set `safeailabs/` as a prefix when sending requests**

## Available Models

| Model | Context window | Max output tokens |
|-------|----------------|-------------------|
| `safeailabs/sail-frontier` | 1,048,576 | 65,536 |

Model ids carry the vendor prefix, so a LiteLLM model is `safeailabs/<vendor>/<model>`. Each region lists what it serves, with limits and prices, at `GET https://us-region.api.safeailabs.ai/v1/models`. LiteLLM ships the price of the model above, so its spend is tracked; for another model, pass `input_cost_per_token` and `output_cost_per_token` in `litellm_params`.

## Required Variables

```python showLineNumbers title="Environment Variables"
os.environ["SAFE_AI_KEY"] = ""  # your Safe AI Labs API key, from https://safeailabs.ai/keys
```

`SAFE_AI_BASE_URL` overrides the base URL, to use another region or the global endpoint `https://gbl.api.safeailabs.ai/v1`. The regions are listed at https://safeailabs.ai/docs/regions.

## Usage - LiteLLM Python SDK

### Non-streaming

```python showLineNumbers title="Safe AI Labs Non-streaming Completion"
import os
from litellm import completion

os.environ["SAFE_AI_KEY"] = ""  # your Safe AI Labs API key

response = completion(
    model="safeailabs/sail-frontier",
    messages=[{"content": "Hello, how are you?", "role": "user"}],
)

print(response)
```

### Streaming

```python showLineNumbers title="Safe AI Labs Streaming Completion"
import os
from litellm import completion

os.environ["SAFE_AI_KEY"] = ""  # your Safe AI Labs API key

response = completion(
    model="safeailabs/sail-frontier",
    messages=[{"content": "Write a short story about AI", "role": "user"}],
    stream=True,
)

for chunk in response:
    print(chunk)
```

### Function Calling

```python showLineNumbers title="Safe AI Labs Function Calling"
import os
from litellm import completion

os.environ["SAFE_AI_KEY"] = ""  # your Safe AI Labs API key

tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get the current weather in a location",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "The city, e.g. San Francisco"
                }
            },
            "required": ["city"]
        }
    }
}]

response = completion(
    model="safeailabs/sail-frontier",
    messages=[{"role": "user", "content": "What's the weather in San Francisco?"}],
    tools=tools,
    tool_choice="auto",
)

print(response)
```

### Responses API

Safe AI Labs serves the OpenAI Responses API, so `litellm.responses` sends the request to `https://us-region.api.safeailabs.ai/v1/responses`.

```python showLineNumbers title="Safe AI Labs Responses API"
import os
import litellm

os.environ["SAFE_AI_KEY"] = ""  # your Safe AI Labs API key

response = litellm.responses(
    model="safeailabs/sail-frontier",
    input="Say hello",
)

print(response.output_text)
```

### Anthropic Messages API

Safe AI Labs also serves the Anthropic Messages API, so `litellm.anthropic.messages.acreate` sends the request to `https://us-region.api.safeailabs.ai/v1/messages`.

```python showLineNumbers title="Safe AI Labs Anthropic Messages API"
import asyncio
import os
import litellm

os.environ["SAFE_AI_KEY"] = ""  # your Safe AI Labs API key

async def main():
    response = await litellm.anthropic.messages.acreate(
        model="safeailabs/sail-frontier",
        messages=[{"role": "user", "content": "Say hello"}],
        max_tokens=64,
    )
    print(response["content"][0]["text"])

asyncio.run(main())
```

## Usage - LiteLLM Proxy Server

```yaml showLineNumbers title="config.yaml"
model_list:
  - model_name: sail-frontier
    litellm_params:
      model: safeailabs/sail-frontier
      api_key: os.environ/SAFE_AI_KEY
```

A deployment configured this way serves all three endpoints on the proxy: `/v1/chat/completions`, `/v1/responses`, and `/v1/messages`.

<Tabs>
<TabItem value="chat" label="Chat Completions">

```bash showLineNumbers title="curl"
curl http://0.0.0.0:4000/v1/chat/completions \
  -H "Authorization: Bearer sk-1234" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sail-frontier",
    "messages": [{"role": "user", "content": "Hello, how are you?"}]
  }'
```

</TabItem>
<TabItem value="responses" label="Responses">

```bash showLineNumbers title="curl"
curl http://0.0.0.0:4000/v1/responses \
  -H "Authorization: Bearer sk-1234" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sail-frontier",
    "input": "Hello, how are you?"
  }'
```

</TabItem>
<TabItem value="messages" label="Messages">

```bash showLineNumbers title="curl"
curl http://0.0.0.0:4000/v1/messages \
  -H "x-api-key: sk-1234" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sail-frontier",
    "max_tokens": 64,
    "messages": [{"role": "user", "content": "Hello, how are you?"}]
  }'
```

</TabItem>
</Tabs>

## Custom API Base

**Option 1: Environment variable**

```python showLineNumbers title="Custom API Base via env var"
import os
from litellm import completion

os.environ["SAFE_AI_BASE_URL"] = "https://gbl.api.safeailabs.ai/v1"
os.environ["SAFE_AI_KEY"] = ""  # your API key

response = completion(
    model="safeailabs/sail-frontier",
    messages=[{"content": "Hello!", "role": "user"}],
)
```

**Option 2: Pass directly**

```python showLineNumbers title="Custom API Base via parameter"
from litellm import completion

response = completion(
    model="safeailabs/sail-frontier",
    messages=[{"content": "Hello!", "role": "user"}],
    api_base="https://gbl.api.safeailabs.ai/v1",
    api_key="your-api-key",
)
```

## Supported OpenAI Parameters

- `temperature`
- `top_p`
- `max_tokens`
- `max_completion_tokens`
- `frequency_penalty`
- `presence_penalty`
- `stop`
- `seed`
- `stream`
- `stream_options`
- `tools`
- `tool_choice`
- `parallel_tool_calls`
- `response_format`
- `logprobs`
- `top_logprobs`
- `reasoning_effort`
