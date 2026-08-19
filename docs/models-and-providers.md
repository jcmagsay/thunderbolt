# Models, Providers, and Portability

Thunderbolt distinguishes between an AI **model** and a **provider**.

- A **model** is the AI system that processes a prompt and generates a response.
- A **provider** is the service or runtime that makes a model available through an API.

```text
Thunderbolt → provider or runtime → selected model
```

## Examples

| Provider or runtime | Example models |
| --- | --- |
| OpenAI | GPT models |
| Anthropic | Claude models |
| Mistral | Mistral models |
| Ollama | Locally installed Llama, Qwen, Gemma, and other supported models |
| OpenRouter | Models from multiple model creators exposed through one service |
| Your organization | A model exposed through an on-prem OpenAI-compatible endpoint |

A provider can offer several models. The same model family can also be available through more than one provider or deployment environment.

Ollama is slightly different from a conventional cloud provider: it is a runtime that normally serves models from your own computer. From Thunderbolt's perspective, it still occupies the provider role because Thunderbolt sends requests to its API.

## What Portability Means

Thunderbolt keeps the user-facing workspace separate from the service performing inference:

```text
                         ┌── Anthropic → Claude model
Thunderbolt workspace ───├── OpenAI → GPT model
                         ├── Ollama → local model
                         └── On-prem API → organization-managed model
```

You can switch:

- Between models offered by one provider
- From one cloud provider to another
- From a cloud model to local inference
- From a public service to an organization-managed endpoint

Your Thunderbolt conversations, skills, MCP connections, settings, and interface remain in the Thunderbolt workspace rather than belonging exclusively to one model provider.

## What Portability Does Not Mean

Changing models is not guaranteed to produce identical behavior. Models and providers differ in:

- Reasoning and response quality
- Context-window size
- Tool-calling reliability
- Image, audio, and document support
- Provider-specific options
- Latency, availability, and price
- Safety behavior and data-handling terms

Some conversation content may transfer cleanly while a provider-specific capability does not. Portability means you can change the inference route without replacing the entire workspace; it does not make every route interchangeable.

For quality tradeoffs, see [Local vs. Native Quality](./local-vs-native-quality.md).
