# Engineering Quality, CI, Security, and Operations

This guide explains the automated safeguards around a Thunderbolt change. It distinguishes hard merge signals from informational reporting and highlights workflows that can deploy infrastructure or consume paid services.

Repository configuration cannot show whether GitHub branch protection requires every check, whether organization-level secret scanning or Dependabot is enabled, or what spending limits exist. Verify those controls in GitHub before treating them as enforced.

## What Enforces Quality

| Layer                     | What it checks                                                                                  | Enforcement                                                                                             |
| ------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `AGENTS.md`               | Architecture, TypeScript, React, data deletion, testing, sync, E2EE, and deployment conventions | Guidance for contributors and coding agents; not every rule is machine-enforced                         |
| Husky + lint-staged       | Fast checks on staged files                                                                     | Local pre-commit hook; can be bypassed                                                                  |
| `bun run check`           | TypeScript, ESLint, Prettier, and source license headers                                        | Local command; CI currently runs type checking and linting separately rather than this complete command |
| `ci.yml`                  | TypeScript, lint, randomized tests, backend tests, Rust, CLI, and WASM integrity                | Hard workflow failures                                                                                  |
| `e2e.yml`                 | Real-browser OIDC, SAML, ACP, proxy, WebSocket, and artifact flows                              | Hard workflow failures                                                                                  |
| `security.yml`            | Semgrep SAST, OWASP, JWT, and secret-pattern rules                                              | Scan failures are visible; SARIF upload is allowed to fail without failing the scan                     |
| `pr-metrics.yml`          | Coverage, bundle size, Lighthouse, accessibility, and Web Vitals                                | Informational; most collection and assertion steps continue on error                                    |
| `thunder-deep-review.yml` | AI-assisted correctness, security, architecture, and convention review                          | Advisory review, not a substitute for deterministic checks or human approval                            |

PR titles must use Conventional Commit syntax, such as `fix: ...` or `docs: ...`, unless they use the legacy `THU-123: ...` form.

## Test Strategy

The primary TypeScript and backend suites run five times in randomized order in CI. This is intended to expose leaked state, order dependence, races, and incomplete cleanup. Dedicated jobs cover the browser agent core and CLI only when their paths change. Rust changes run build, Clippy with warnings denied, and tests.

Playwright runs two shards with one Chromium worker per shard. It starts real Vite and backend processes and uses mock OIDC and SAML identity providers. Failure artifacts include traces, screenshots, and an HTML report with limited retention.

Important gaps:

- There is no enforced minimum coverage percentage.
- Coverage, bundle size, Lighthouse, accessibility, and Web Vitals are reported rather than merge-blocking.
- Browser E2E covers Chromium, not Firefox or WebKit/Safari.
- There is no equivalent automated Tauri, iOS, or Android E2E matrix.
- Multi-device PowerSync, offline conflicts, E2EE recovery, mixed client versions, and cross-runtime encryption need broader production-like testing.
- Retried WebSocket tests reduce noise but can conceal intermittent defects; repeated retries should be investigated rather than normalized.
- AI evaluations depend on model/provider behavior and cost. They are manually triggered and complement deterministic tests rather than replacing them.

Run the normal local checks with:

```sh
bun run check
bun run test
bun run test:backend
```

Do not run bare `bun test` at the repository root. See [Testing](./testing.md) for scoped commands and test patterns.

## Performance and Quality Baselines

PR Metrics compares coverage and gzipped bundle size with a cached `main` baseline and reports Lighthouse results from a deployed preview. The configured Lighthouse warning targets are:

| Metric                   | Warning target   |
| ------------------------ | ---------------- |
| Performance              | 80               |
| Accessibility            | 90               |
| Best practices           | 90               |
| SEO                      | 80               |
| First Contentful Paint   | 2 seconds        |
| Largest Contentful Paint | 2.5 seconds      |
| Total Blocking Time      | 300 milliseconds |
| Cumulative Layout Shift  | 0.1              |

A coverage drop greater than two percentage points is displayed as red in the PR comment, but it does not fail CI. Bundle measurement also has no configured maximum. Treat these as review signals, not guarantees.

## Workflows That Can Spend Money

Most PR workflows use concurrency cancellation and explicit timeouts. Rust, CLI, agent-core, and WASM jobs use path filters. Test artifacts generally expire after 7–14 days.

The material cost surfaces are:

- Internal PRs build container images and deploy AWS Fargate previews. Fork previews require maintainer approval.
- Preview stacks are destroyed on PR close; an hourly cleanup removes failed-orphan and inactive stacks after three days. The `preview:persist` label opts a stack out.
- Nightly release workflows build desktop, iOS, Android, and CLI artifacts; macOS and mobile builds can consume substantial runner time.
- Nightly image workflows rebuild and publish the container set and Helm chart.
- AI deep review uses a high-effort hosted model with bounded turns and a 30-minute job timeout.
- AI evaluations invoke paid inference providers and can run for substantially longer than ordinary tests.

Before adding a scheduled job or matrix axis, set a timeout, concurrency policy, path filter where appropriate, artifact retention, and an owner-visible cost signal. Infrastructure should have deterministic teardown and external budget alerts. GitHub Actions, AWS, model-provider, and artifact-storage budgets are organization settings and are not defined by this repository.

## Security Detection

Current controls include Semgrep, pinned GitHub Action revisions, frozen dependency installs, a targeted Rust advisory guard, WASM checksum/staleness validation, scoped workflow permissions, and approval-gated fork previews.

### Dependency CVE reporting is not in CI

Semgrep scans source code; it does not audit the resolved packages in the Bun and Cargo lockfiles. No current workflow runs `bun audit`, `cargo audit`, OSV-Scanner, or an equivalent software-composition analysis tool. Consequently, CI has no dependency-vulnerability report or dependency-security baseline.

The existing baseline mechanisms cover different concerns:

| Baseline            | What it compares                                                | Does it block a dependency CVE? |
| ------------------- | --------------------------------------------------------------- | ------------------------------- |
| PR Metrics          | Coverage and gzipped bundle size against a cached `main` result | No                              |
| Semgrep PR scan     | New source-code findings relative to the PR base commit         | No                              |
| AI evaluations      | Model behavior against checked-in expectations                  | No                              |
| Dependency security | Not implemented                                                 | No                              |

A point-in-time `bun audit` on August 19, 2026 reported:

| Package root     | Critical | High | Moderate | Low |
| ---------------- | -------: | ---: | -------: | --: |
| Root application |        2 |   27 |       38 |  10 |
| Backend          |        2 |   42 |       21 |   2 |
| CLI              |        0 |    4 |        5 |   0 |
| Marketing web    |        0 |    4 |       17 |   6 |
| Pulumi           |        2 |   18 |       20 |   2 |

These totals are discovery results, not proof that every advisory is reachable in Thunderbolt. Direct runtime findings—particularly authentication, SSO, proxying, parsing, and externally reachable backend dependencies—require immediate applicability review. Counts become stale as advisories and lockfiles change; rerun the audit rather than treating this table as a live status dashboard.

Repository-visible gaps include:

- No CodeQL workflow
- No general JavaScript or Rust dependency audit gate
- No container-image vulnerability scan or SBOM generation
- No infrastructure-as-code security scan
- No dedicated Gitleaks/TruffleHog workflow
- No checked-in Dependabot or Renovate configuration
- No dynamic security test against preview environments

GitHub may provide dependency alerts, secret scanning, or push protection outside the repository. Confirm their status in repository settings. Report vulnerabilities privately as described in [`SECURITY.md`](../../SECURITY.md).

### Dependency security roadmap

This is required production-readiness work:

1. **Remediate direct critical/high runtime findings.** Start with authentication and SSO dependencies, then externally reachable backend packages.
2. **Add automated update management.** Configure Dependabot or Renovate for every Bun package root, both Cargo manifests, GitHub Actions, and container base images.
3. **Add a required multi-lockfile audit job.** Scan all five Bun lockfiles and both Cargo lockfiles on PRs, `main`, and a weekly schedule.
4. **Establish a differential baseline.** Fail when a PR introduces a critical/high advisory or increases risk relative to `main`; do not permanently grandfather everything already present.
5. **Create expiring exceptions.** Every accepted advisory needs an owner, reachability rationale, mitigation, and expiration date.
6. **Expand supply-chain scanning.** Scan final container images and infrastructure configuration, generate SBOMs, and publish SARIF or another reviewable report.
7. **Harden installs consistently.** Apply the release-age quarantine to backend, web, and Pulumi package roots; use lifecycle-script-free CI installs where builds permit it.
8. **Add Firefox, then WebKit coverage.** Chromium is the only browser currently installed and tested by Playwright CI.

The target PR report should show counts and changes per package root, distinguish runtime from development scope where possible, list expiring exceptions, and fail according to a documented severity/remediation policy. A green Semgrep check must not be interpreted as a clean dependency audit.

## Monitoring and Privacy

The backend provides structured logs, a health endpoint, optional OpenTelemetry export, and body-free inference latency/error telemetry. PostHog supplies optional product, startup, chat-readiness, and sync diagnostics. Self-hosted deployments without a PostHog key do not initialize analytics.

Client analytics defaults disable autocapture, session recording, automatic exception capture, page views, surveys, and performance capture. User consent controls event capture, dynamic route IDs are sanitized, and inference error telemetry is designed not to include prompt or response bodies.

These controls do not make every data path local:

- A selected cloud model receives prompts sent to it.
- MCP tools and external integrations receive the data intentionally passed to them.
- Local SQLite contains readable working data while the application is in use.
- E2EE protects configured synchronized fields, not all sync metadata.
- Preview environments can receive provider credentials and should use test data only.
- Local inference remains local only when the configured model and tools do not route data elsewhere.

Monitoring is instrumentation rather than a complete operational program. The repository does not define production SLOs, alert thresholds, on-call routing, mobile crash reporting, or dashboard provisioning. Operators should establish availability, latency, error-rate, sync-lag, provider-cost, and restore/recovery signals for their deployment.

Read [Telemetry](../../TELEMETRY.md), [Self-hosting Configuration](../self-hosting/configuration.md), and [Sync and E2EE Production Readiness](../architecture/production-readiness.md) for the underlying controls and limitations.

## Review Checklist

Before merging a change:

1. Confirm the relevant hard CI jobs ran; path-filtered jobs may legitimately be skipped.
2. Inspect PR Metrics rather than assuming green CI means no performance or coverage regression.
3. Add tests at the lowest useful level and an E2E test when a cross-process user journey can regress.
4. Treat flaky retries as evidence to investigate.
5. Review new telemetry for consent, identifiers, free text, prompts, responses, secrets, and retention.
6. Review new integrations for authentication, SSRF, CORS, proxy-header, and data-disclosure boundaries.
7. Add timeouts, cancellation, retention, teardown, and cost ownership to new workflows.
8. Apply the production-readiness checklist to sync, encryption, device, schema, or migration changes.
