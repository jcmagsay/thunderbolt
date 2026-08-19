# Engineering Quality Improvement Proposal

This document translates repository-visible observations from [Engineering Quality, CI, Security, and Operations](./engineering-quality.md) into one possible sequence of measurable work.

It is a contributor proposal, not an approved roadmap, commitment, priority order, staffing request, or statement that maintainers are unaware of these topics. Maintainers may have additional controls, private security work, and roadmap context. Each area can be accepted, rejected, reordered, or tracked independently through focused issues and PRs. Potentially applicable security findings should follow the private process in [`SECURITY.md`](../../SECURITY.md) rather than wait for adoption of this proposal.

## Proposed outcomes

1. Prevent known critical/high runtime vulnerabilities from shipping without an owned, expiring exception.
2. Keep every dependency root current through automated, reviewable updates.
3. Enforce deterministic CI quality while tracking coverage, performance, flakes, and cost.
4. Establish risk-based Chromium, Firefox, WebKit, Tauri, and mobile coverage.
5. Make monitoring, privacy, security, and recovery controls measurable.
6. Give agents durable context and a validated skill for recomputing the scorecard.

## Candidate phase 0: Governance and baseline

- Consider separate architecture-decision proposals for dependency security and cross-browser compatibility gates.
- Assign owners for dependency security, CI/browser quality, infrastructure cost, observability, privacy/security, and agent quality.
- Record the first scorecard using the repository's `measure-engineering-quality` skill.
- Define runtime/development reachability labels, remediation SLAs, and the expiring exception schema.
- Confirm GitHub branch protection, required checks, Dependabot/security settings, budget alerts, and environment reviewers outside the repository.

**Evidence of completion if adopted:** dated baseline with commit SHA and raw evidence; named owners; accepted severity policy; no unknown owner for a critical finding.

## Candidate phase 1: Triage immediate dependency risk

- Privately review direct authentication, SSO, and externally reachable dependencies according to the maintainers' severity process.
- Review applicable critical/high direct runtime findings for reachability before publishing details.
- Add a scheduled and PR-triggered audit for five Bun and two Cargo lockfiles.
- Publish a combined artifact/PR summary; initially report existing findings while blocking newly introduced critical/high findings.
- Add owned, expiring exceptions only where remediation is not immediately possible.

**Evidence of completion if adopted:** zero unowned critical findings; no new applicable critical/high finding can merge under the accepted policy; all exceptions have owner, rationale, mitigation, and expiry.

## Candidate phase 2: Reduce supply-chain recurrence

- Configure Dependabot or Renovate for Bun, Cargo, GitHub Actions, and container bases.
- Apply frozen lockfiles and the release-age quarantine consistently to every Bun package root.
- Use lifecycle-script-free CI installs where builds pass; explicitly document any trusted script.
- Add Rust advisory, container-image, secret, and IaC scans.
- Generate SBOMs and attestations for published artifacts.
- Define update latency and vulnerability remediation dashboards.

**Evidence of completion if adopted:** every manifest is owned by update automation; final images are scanned; trust exceptions are explicit; median safe patch adoption and overdue SLA counts are measurable.

## Candidate phase 3: Expand browser and runtime confidence

- Add a Firefox smoke suite on every PR, then promote it to required after measuring flakes.
- Add full Firefox and WebKit suites on `main` or nightly.
- Classify tests into browser-neutral, browser-sensitive, auth/proxy, sync/storage, and artifact groups to control cost.
- Establish a flake budget and quarantine policy with owner and expiry.
- Add mobile viewports, then real Tauri desktop and mobile smoke coverage where runners permit.
- Prioritize PowerSync, worker, storage, streaming, auth-cookie, and E2EE paths that differ by runtime.

**Evidence of completion if adopted:** Chromium and Firefox required for risk-relevant PRs; WebKit scheduled and owned; flake rate and time-to-signal remain within the accepted budget; platform-specific gaps are visible.

## Candidate phase 4: Evaluate quality gates

- Make formatting and license headers explicit CI gates.
- Define coverage policy by critical subsystem rather than one repository-wide percentage.
- Establish bundle and Lighthouse regression budgets with an exception mechanism.
- Replace retry-normalized flakes with tracked root-cause fixes.
- Add path-aware preview deployment so documentation-only changes do not build and deploy the full stack.
- Correct preview URLs used by PR Metrics and verify Lighthouse reports are collected.

**Evidence of completion if adopted:** quality regressions fail according to documented budgets; every bypass is owned and expiring; CI duration, flake rate, and preview cost trend downward or remain within budget.

## Candidate phase 5: Operational trust, monitoring, and privacy

- Define SLIs/SLOs for availability, latency, inference failures, authentication, sync lag, and recovery.
- Provision dashboards and actionable alerts with an owning on-call path.
- Add dependency/provider cost signals and budget alerts.
- Test backup restore, device loss, E2EE recovery, mixed versions, and failed deployments.
- Maintain telemetry inventory, consent behavior, retention, identifier classification, and prompt/response exclusion tests.
- Commission independent security and cryptography review for production claims.

**Evidence of completion if adopted:** SLOs and alerts are exercised; restore/recovery drills have evidence; privacy inventory matches code; production claims cite independent review and operational results.

## Scorecard cadence

If maintainers adopt the measurement approach, run the repository's `measure-engineering-quality` skill:

- After a relevant architecture decision is accepted
- After material CI, security, browser, telemetry, or infrastructure changes
- Before a production-readiness milestone, when requested by its owner
- Monthly while maintainers are actively tracking applicable critical/high findings or adopted proposal phases

Track domain score, baseline delta, critical/high counts, exception age, patch adoption time, CI duration, flaky-test rate, browser pass rate, preview cost, SLO compliance, recovery-drill status, and agent-skill validation status.

## Agent context and skill success

- `AGENTS.md` routes quality-baseline requests to the repository skill.
- `.agents/skills/measure-engineering-quality/` defines the evidence workflow and weighted scorecard.
- [Agent Skill Validation](./agent-skill-validation.md) applies to every current and future repository-owned skill.
- Canonical guidance is provider and model agnostic; optional adapters cannot be required for execution or redefine the workflow.
- Skill measurements must preserve raw evidence, declare unknowns, and never produce a false pass when tools or permissions are missing.

The measurement system succeeds when two independent, differently configured agent interfaces using the same commit and evidence agree on mechanical facts and domain scores, while any judgment difference is visible in cited rationale.
