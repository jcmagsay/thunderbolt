# Thunderbolt [![CI](https://github.com/thunderbird/thunderbolt/actions/workflows/ci.yml/badge.svg)](https://github.com/thunderbird/thunderbolt/actions/workflows/ci.yml)

**AI You Control: Choose your models. Own your data. Eliminate vendor lock-in.**

![Thunderbolt Main Dashboard](./docs/screenshots/main.png)

> [!IMPORTANT]
> ⚠️ **We are excited about the amount of interest Thunderbolt has been getting and want to clarify that it is still early and under active development**. Currently, we are targeting enterprise customers that want to deploy it on-prem. We encourage you to self-host it and try it out, but there are a few caveats we are still working on:
>
> - While we eventually plan to make Thunderbolt fully offline-first, it currently depends on authentication and search functionality (though you can disable search on the integrations screen in the app). You can [deploy your own backend with Docker](./deploy/README.md) and sign up in order to test it locally.
> - You’ll need to add your own model providers - we don’t yet have a public inference endpoint. We recommend using Thunderbolt with [Ollama](https://ollama.com) or [llama.cpp](https://github.com/ggml-org/llama.cpp) if you want free local inference, or you can add API keys for any OpenAI-compatible model provider in the settings.

Thunderbolt is an open-source, cross-platform AI client that can be deployed on-prem anywhere.

- 🌐 Available on all major desktop and mobile platforms: web, iOS, Android, Mac, Linux, and Windows.
- 🧠 Compatible with frontier, local, and on-prem models.
- 🙋 Enterprise features, support, and FDEs available.

**Thunderbolt is under active development, currently undergoing a security audit, and preparing for enterprise production readiness.**

## Why Thunderbolt?

Thunderbolt is for people and organizations that want a useful AI workspace without handing control of the product, models, and data to one vendor.

- **Choose your models and providers:** select the AI model that performs the work and the cloud service, local runtime, or on-prem endpoint that serves it.
- **Own the deployment:** run the backend, identity, proxy, synchronization, and data infrastructure yourself.
- **Work locally:** each client uses local SQLite, so the application can read and write without waiting on a server round trip.
- **Extend the workspace:** connect tools and company context through MCP, and package reusable behavior as skills.
- **Use it across devices:** one React application supplies the shared interface and product behavior for browsers, macOS, Windows, Linux, iOS, and Android. Tauri packages that web application inside a small native shell for desktop and mobile, adding access to capabilities such as native SQLite, deep links, notifications, haptics, native file access, and application updates.
- **Optionally protect synced content:** preview E2E encryption encrypts supported fields before they pass through the sync infrastructure.

Practical uses include a private company AI workspace, a local-model assistant, a provider-neutral model evaluation environment, and a governed interface to organizational context and tools.

## Choose Your Thunderbolt Path

Thunderbolt is the workspace and control plane around AI; it is not itself a model host. Ollama, llama.cpp, a cloud API, or an on-prem inference cluster performs the inference. This creates two valid ways to use the project:

|                 | Workspace adopter                                             | Infrastructure operator                                                    |
| --------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Goal            | Get one useful interface for chat, context, skills, and tools | Control where models run, where data lives, and which services are allowed |
| Inference       | Add one local or hosted provider                              | Operate Ollama, llama.cpp, or an OpenAI-compatible inference service       |
| Data            | Keep working data in Thunderbolt's local client database      | Self-host the backend, database, sync, identity, and optional integrations |
| Best first step | Add an Ollama model in **Settings → Models**                  | Start locally, then replace or disable each optional service deliberately  |

For a private single-machine setup, Thunderbolt can send model requests directly from the client to Ollama on `localhost`; no Fireworks, Anthropic, OpenAI, Render, or Thunderbolt inference key is required. The Thunderbolt backend is still required today for application configuration and authentication, and the complete development stack uses locally running PostgreSQL and PowerSync for sync. That is a current product boundary, not a requirement to buy hosted accounts.

Render is one deployment target, Linear is project-management tooling, and Fireworks is one optional hosted model provider. They are not prerequisites. Likewise, PostHog, Resend, Exa, Tinfoil, and enterprise identity providers enable specific optional capabilities rather than the core local-model path.

Thunderbolt also does not download models, schedule GPUs, or operate inference clusters. Ollama or another inference runtime owns that layer; Thunderbolt gives users a portable workspace and gives operators a consistent place to select models, connect organizational context and tools, and govern the surrounding experience.

Read [Contributor Onboarding](./onboarding.md#two-ways-to-adopt-thunderbolt) for the full local-only data flow, service matrix, setup path, and current limitations.

### One application, several runtimes

Thunderbolt does not maintain a separate product implementation for every operating system:

```text
Shared React application
├── Browser → runs directly as a web application
├── Desktop → Tauri shell on macOS, Windows, and Linux
└── Mobile  → Tauri shell on iOS and Android
```

Most routes, components, chat behavior, model configuration, skills, and data-access code are shared. Tauri is the native container around that code—not an AI model and not a separate frontend framework. Its Rust layer integrates the shared application with operating-system features and packages it as an installable native app.

“One application” does not mean every runtime is identical. Browsers use WA-SQLite and browser storage APIs, while Tauri uses native SQLite and native plugins. Worker support, embedded web views, file access, notifications, updates, and mobile resource limits also differ by platform. Thunderbolt contains runtime-specific adapters for those boundaries while keeping the product interface and most business logic common.

The benefit is that a feature can usually be implemented once and delivered across web, desktop, and mobile. The tradeoff is that database, sync, encryption, and native integration changes still require testing on each supported runtime.

### Sync and E2EE, briefly

**Sync** copies locally stored chats, settings, skills, and other supported data between your devices. **End-to-end encryption (E2EE)** encrypts protected fields before they leave one approved device and decrypts them after they reach another. The sync service stores ciphertext instead of readable protected content.

E2EE protects the synchronization path; it does not hide prompts from a cloud model selected to process them, secure an already-compromised device, or encrypt all synchronization metadata. Sync and E2EE are preview features. Read [Production Readiness](./docs/architecture/production-readiness.md) before relying on them for sensitive or critical workloads.

## Find Your Way Around

- [Contributor Onboarding](./onboarding.md) — a short product, architecture, repository, and development tour.
- [Architecture](./docs/architecture/README.md) — components and data flows.
- [Production Readiness](./docs/architecture/production-readiness.md) — sync/E2EE behavior, repository-visible risks, and proposed trust-building work.
- [Models and Providers](./docs/models-and-providers.md) — what each term means and what portability does and does not guarantee.
- [Local vs. Native Quality](./docs/local-vs-native-quality.md) — when Thunderbolt can match local or hosted native AI experiences, and when it cannot.
- [Five-Minute Adoption Demo](./docs/adoption-demo.md) — proposed company-context demonstration and success criteria.
- [Competitive Landscape](./competition.md) — direct competitors, commercial substitutes, and positioning.

## Get Started Locally

```sh
make doctor    # verify your tools — prints exact install commands for anything missing
make setup     # install frontend + backend dependencies, wire up agent symlinks
make up        # start Postgres + PowerSync in Docker
make run       # start the backend (:8000) and frontend (:1420)
```

For self-hosting with Docker Compose or Kubernetes, see [`deploy/README.md`](./deploy/README.md). For full dev-environment details, see [`docs/development/quick-start.md`](./docs/development/quick-start.md).

## Need Help?

Found a bug? Have an idea?

- We're actively working on our docs, community, and roadmap. For now, the best way to get in touch is to [File an issue](https://github.com/thunderbird/thunderbolt/issues).

## Contributing

We welcome contributions from everyone.

- **Development**: The [development guide](./docs/development/quick-start.md) will help you get started.
- Make sure to check out the [Mozilla Community Participation Guidelines](https://www.mozilla.org/about/governance/policies/participation/).

## Documentation

- [FAQ](./docs/faq.md) - Frequently asked questions
- [Deployment](./deploy/README.md) - Self-host with Docker Compose or Kubernetes
- [Development](./docs/development/quick-start.md) - Quick start, setup, and testing
- [Architecture](./docs/architecture/README.md) - System architecture and diagrams
- [Storybook](./docs/dev-tooling/storybook.md) - Build, test, and document components
- [Vite Bundle Analyzer](./docs/dev-tooling/vite-bundle-analyzer.md) - Analyze frontend bundle size
- [Tauri Signing Keys](./docs/features/tauri-signing-keys.md) - Generate and manage signing keys for releases
- [Release Process](./RELEASE.md) - Instructions for creating and publishing new releases
- [Telemetry](./TELEMETRY.md) - Information about data collection and privacy policy

## Code of Conduct

Please read our [Code of Conduct](./CODE_OF_CONDUCT.md). All participants in the Thunderbolt community agree to follow these guidelines and [Mozilla's Community Participation Guidelines](https://www.mozilla.org/about/governance/policies/participation/).

## Security

If you discover a security vulnerability, please report it responsibly via our [vulnerability reporting form](https://github.com/thunderbird/thunderbolt/security/advisories/new). Please do **not** file public GitHub issues for security vulnerabilities.

## License

Thunderbolt is licensed under the [Mozilla Public License 2.0](./LICENSE).
