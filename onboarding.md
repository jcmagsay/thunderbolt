# Thunderbolt Contributor Onboarding

Thunderbolt is an open-source, cross-platform AI workspace for organizations and individuals who want to choose their models, control their infrastructure, and own their data.

This page is the short route into the repository. Follow the links when you need the deeper architecture or operational detail.

## Why It Is Interesting

- **Model-neutral:** choose both the AI model and the cloud service, local runtime, or on-prem endpoint that serves it.
- **Local-first:** every client works against a local SQLite database.
- **Self-hostable:** operate the backend, authentication, proxy, PostgreSQL, and PowerSync infrastructure.
- **Cross-platform:** one React application targets web, macOS, Windows, Linux, iOS, and Android through Tauri.
- **Extensible:** connect MCP tools and context, create reusable skills, and render interactive response widgets.
- **Model and provider portable:** keep the Thunderbolt conversations, skills, tools, and interface while changing the model or service used for inference. Capabilities and results can still differ.
- **Privacy-oriented:** optionally encrypt protected fields before synchronizing them between approved devices.

Practical uses include a private company assistant, a local-model client, a provider evaluation environment, a governed interface to organizational context, and a reference implementation for local-first AI.

## Two Ways to Adopt Thunderbolt

The simplest mental model is:

```text
Thunderbolt = workspace and control plane
Ollama / llama.cpp / cloud API / on-prem cluster = inference engine
Model = the weights and behavior served by that engine
```

Thunderbolt manages the user-facing workspace: conversations, model definitions, skills, tools, company context, and local data. It routes work to inference engines, but it does not download model weights, schedule GPUs, or replace an inference runtime such as Ollama.

### User A: I want the easy workspace

This user wants to chat, add company context and tools, and move between models without replacing the whole application. The shortest privacy-oriented route is one Thunderbolt client plus Ollama on the same machine. A hosted API is also an option when model quality or hardware requirements outweigh the local-only preference.

The hosted “easy button” is not complete today: Thunderbolt does not provide a public inference service, and the project remains early. Users must currently supply either a local inference endpoint or credentials for a hosted provider.

### User B: I want to control the AI stack

This operator chooses and runs each layer:

- Thunderbolt clients and backend
- Ollama, llama.cpp, or another OpenAI-compatible inference service
- Identity and access policy
- PostgreSQL and optional multi-device synchronization
- Optional E2EE for protected synced fields
- MCP tools, company context, search, and observability

This is ownership through replaceable, self-hostable components—not a claim that Thunderbolt is an all-in-one GPU orchestration platform. For example, Ollama manages local model downloads and serving; Kubernetes or another platform may manage a larger inference cluster; Thunderbolt supplies the consistent product experience above them.

### Local-only request flow

With a custom Ollama model configured at `http://localhost:11434/v1`, loopback inference bypasses Thunderbolt's universal backend proxy:

```text
prompt ─▶ Thunderbolt client ─▶ Ollama on this machine ─▶ local model
               │
               └── local SQLite: conversations, settings, skills, tools
```

No cloud model provider receives that inference request. The backend is still part of the current application startup for configuration and authentication. If sync is enabled, supported records also move through PowerSync and PostgreSQL; deploy those locally or on infrastructure you control. E2EE protects supported synced fields, but sync metadata and any data deliberately sent to an external tool retain the limitations described in [Production Readiness](./docs/architecture/production-readiness.md).

### Which outside services do I actually need?

| Service                                           | Required for local Ollama use?    | Why it appears in the repository                                                                           |
| ------------------------------------------------- | --------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Ollama or llama.cpp                               | Yes, choose one local runtime     | Downloads and serves local models                                                                          |
| Thunderbolt backend                               | Yes, currently                    | App configuration, authentication, device APIs, and optional proxy/integrations                            |
| PostgreSQL + PowerSync                            | Used by the complete local stack  | Local Docker services provide persistence and multi-device sync; no vendor account is required             |
| Fireworks, Anthropic, OpenAI, Mistral, OpenRouter | No                                | Optional hosted inference alternatives                                                                     |
| Render                                            | No                                | A deployment target used by the project, replaceable with local Docker, Kubernetes, or your infrastructure |
| Linear                                            | No                                | Project planning for contributors; it is not in the runtime data path                                      |
| Resend                                            | No                                | Optional transactional email delivery                                                                      |
| PostHog                                           | No                                | Optional analytics and inference observability                                                             |
| Exa                                               | No                                | Optional web search                                                                                        |
| Tinfoil                                           | No                                | Optional confidential hosted inference                                                                     |
| Keycloak or another IdP                           | No for the basic development path | Optional enterprise identity; self-hostable alternatives are supported                                     |

You can leave optional keys empty; their corresponding hosted capabilities will be unavailable. You should not create accounts or add credit cards merely to satisfy the example environment file.

### Start with Ollama and no model-provider key

1. Install Ollama, start it, and pull a model appropriate for your hardware:

   ```sh
   ollama serve
   ollama pull <model>
   ```

2. Start Thunderbolt using the local development steps below. Leave the hosted provider keys empty.
3. Open **Settings → Models → Add model** and choose **Custom**.
4. Enter `http://localhost:11434/v1` as the URL, the exact Ollama model name as the model ID, and leave the API key empty.
5. Test the connection, add the model, select it in chat, and send a prompt.

Ollama must permit the Thunderbolt browser origin. Native Tauri clients do not have exactly the same browser CORS boundary. See [Models and Providers](./docs/models-and-providers.md) for terminology and [Local vs. Native Quality](./docs/local-vs-native-quality.md) for the quality and hardware tradeoffs.

## Sync and E2EE, Briefly

Thunderbolt stores working data locally.

- **Sync** copies supported local data between your devices through PowerSync and PostgreSQL.
- **End-to-end encryption (E2EE)** encrypts protected fields before upload and decrypts them on an approved receiving device.

```text
Device A ── encrypt ──▶ sync service stores ciphertext ──▶ decrypt ──▶ Device B
```

E2EE protects the synchronization path. It does not hide prompts from a selected cloud model, protect an already compromised device, or encrypt every piece of sync metadata. Sync and E2EE are preview features that need broader testing, operational evidence, and an independent cryptography audit.

Read [Sync and E2EE Production Readiness](./docs/architecture/production-readiness.md) for device approval, recovery, revocation, protection boundaries, risks, and contribution opportunities.

## Architecture in One Picture

```text
Web or Tauri client
├── React UI: chat, settings, tasks, skills, widgets
├── AI layer: model adapters, streaming, MCP, ACP
├── Local data: Drizzle + SQLite
└── Optional client-side encryption
        │
        ├── REST/SSE → Bun/Elysia backend → model providers
        └── Sync → PowerSync → PostgreSQL
```

The client is the working application. The backend handles identity, account/device APIs, inference proxying, and integrations. PowerSync moves supported data between local SQLite and PostgreSQL.

For the full component and data-flow map, read [Architecture](./docs/architecture/README.md).

## Repository Map

| Location                                | Purpose                                                     |
| --------------------------------------- | ----------------------------------------------------------- |
| `src/app.tsx`                           | Routes and top-level providers; start here                  |
| `src/chats/` and `src/components/chat/` | Chat state and interface                                    |
| `src/ai/`                               | Prompts, model middleware, streaming, and evaluations       |
| `shared/agent-core/`                    | Agent execution, MCP, coding tools, and browser environment |
| `src/widgets/`                          | Interactive assistant-response components                   |
| `src/skills/`                           | Reusable model instructions and invocation UI               |
| `src/integrations/`                     | Google, Microsoft, and other connections                    |
| `src/dal/`                              | Client data-access layer                                    |
| `src/db/`                               | SQLite, PowerSync, encryption, migrations, and seeding      |
| `src/search/`                           | Local full-text search                                      |
| `src/settings/`                         | Models, connections, devices, skills, and preferences       |
| `src/acp/`                              | Agent Client Protocol integration                           |
| `src-tauri/`                            | Rust desktop/mobile shell                                   |
| `backend/src/`                          | Bun/Elysia API, auth, proxy, devices, and integrations      |
| `shared/`                               | Frontend/backend types and behavior                         |
| `powersync-service/`                    | Local sync-service configuration                            |
| `deploy/`                               | Docker Compose, Kubernetes, and Pulumi assets               |

## Follow One Chat Message

This is the fastest useful code tour:

1. Find the chat route in `src/app.tsx`.
2. Follow rendering and state through `src/chats/` and `src/components/chat/`.
3. Follow request construction into `src/ai/`.
4. Inspect streamed responses under `src/ai/streaming/`.
5. Follow persistence through `src/dal/` and `src/db/`.
6. Follow structured output into `src/widgets/`.
7. For proxied inference, continue into `backend/src/`.

This crosses the important boundaries without requiring you to understand sync or encryption first.

## Run It Locally

You need Bun, Docker, and either a hosted AI provider key or a local OpenAI-compatible endpoint. A hosted-provider key is not required when using Ollama or llama.cpp. Rust is required for Tauri clients.

```sh
make doctor
make setup
cp .env.example .env
cp backend/.env.example backend/.env
make doctor
make up
make run
```

Open `http://localhost:1420`, sign in, configure a model, and send a message. The backend runs on `:8000` and PowerSync on `:8080`.

Native clients:

```sh
bun tauri:dev:desktop
bun tauri:dev:ios
bun tauri:dev:android
```

See [Development Quick Start](./docs/development/quick-start.md) for prerequisites and troubleshooting.

## Development Workflow

```sh
make check
bun run test
bun test <path> --timeout 5000
bun run test:backend
```

Do not run bare `bun test` at the repository root; it can discover backend integration tests that expect external services and wait indefinitely.

Tests live beside source files as `<file>.test.ts`. Use Bun rather than npm. Generate Drizzle migrations with `bun db generate` instead of creating SQL manually.

Before changing synchronized schemas, read [PowerSync, Account & Device Management](./docs/architecture/powersync-account-devices.md). Synced-table deployments require a staged two-PR rollout.

## Good First Contributions

- Add a focused test for an existing utility or parser.
- Improve a component and its Storybook coverage.
- Add or refine a widget using [Widgets](./docs/features/widgets.md).
- Improve setup diagnostics or a documented failure path.
- Trace and simplify duplicated state or data-access logic without crossing a sync boundary.
- Add evidence and failure tests from the [Production Readiness](./docs/architecture/production-readiness.md) backlog.

A new synchronized table is not a good first change because it crosses frontend, backend, migration, configuration, and deployment boundaries.

## Explore by Goal

- **Understand the system:** [Architecture](./docs/architecture/README.md)
- **Run or deploy it:** [Development](./docs/development/quick-start.md) and [Deployment](./deploy/README.md)
- **Understand sync and encryption:** [Production Readiness](./docs/architecture/production-readiness.md)
- **Build interactive AI output:** [Widgets](./docs/features/widgets.md)
- **Evaluate model behavior:** [AI Evaluations](./src/ai/eval/README.md)
- **Understand models and providers:** [Models and Providers](./docs/models-and-providers.md)
- **Compare local and native AI quality:** [Local vs. Native Quality](./docs/local-vs-native-quality.md)
- **Design the adoption experience:** [Five-Minute Demo](./docs/adoption-demo.md)
- **Understand market positioning:** [Competitive Landscape](./competition.md)

## Current Caveats

- Thunderbolt is early and preparing for enterprise production readiness.
- You supply either model credentials or local inference; there is no public Thunderbolt inference service. Hosted provider accounts are optional.
- Authentication is currently required for a locally hosted evaluation.
- Sync, E2EE, and MCP are preview surfaces; ACP is actively developing.
- A hosted consumer release has no announced date.

Start with chat and model configuration. Treat sync, encryption, deployment, and agent protocols as deeper layers to learn when your task requires them.
