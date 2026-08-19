---
name: measure-engineering-quality
description: Measure Thunderbolt's repository-visible CI enforcement, dependency security, browser coverage, automated tests, workflow cost controls, monitoring, privacy, and agent-skill quality against documented criteria. Use when auditing engineering health, updating security or test baselines, evaluating an accepted improvement proposal, reviewing CI changes, or producing an evidence-backed quality scorecard.
---

# Measure Engineering Quality

Produce a point-in-time scorecard from repository evidence. Do not infer GitHub organization settings, deployed monitoring, budgets, or branch protection from workflow files.

This skill is provider and model agnostic. It must work with local or hosted agents and must not require a specific model vendor, agent product, inference API, or paid service. Treat tool names as capabilities: when a named tool is unavailable, use an equivalent read-only repository or shell capability and report any resulting limitation.

## Workflow

1. Read `AGENTS.md`, `docs/development/engineering-quality.md`, `docs/development/engineering-improvement-plan.md`, and `docs/development/agent-skill-validation.md` completely. Treat proposed work as contributor input unless maintainers have accepted it.
2. Inspect manifests, lockfiles, `bunfig.toml` files, test configuration, `.github/workflows/`, `SECURITY.md`, and `TELEMETRY.md`.
3. Read [scorecard.md](references/scorecard.md) and evaluate every domain. Cite paths and line numbers for every control credited.
4. Run only read-only local checks by default. Ask before network-backed audits. Never deploy, publish, update dependencies, modify GitHub settings, invoke paid model evaluations, or create external resources during measurement.
5. Compare results with the most recent recorded baseline. Mark a value `unknown` when the evidence is unavailable; never convert missing evidence into a passing score.
6. Separate controls into `enforced`, `reported`, `documented`, and `missing`. A workflow that uses `continue-on-error` is not an enforced gate.
7. Report current score, change from baseline, highest-risk gaps, expiring exceptions, and the next three measurable actions.

## Required evidence

Collect, when applicable:

- Dependency audits for every Bun and Cargo lockfile
- Automated update configuration and package-root coverage
- Lifecycle-script and release-age policy by package root
- CI timeouts, permissions, concurrency, retention, and deployment triggers
- Hard versus advisory test, coverage, performance, and security gates
- Playwright browsers, projects, shards, retries, and failure artifacts
- Unit, backend, E2E, Rust, and story inventory
- Security scanners, SBOMs, container/IaC scans, and exception policy
- Monitoring signals, health depth, telemetry consent, and data minimization
- Agent-skill structural validation and forward-test evidence

## Output contract

Return:

1. Date, commit SHA, branch, and measurement limitations
2. Domain score table with evidence links and baseline delta
3. Critical/high findings requiring immediate action
4. Controls added, removed, or weakened since the baseline
5. Exception inventory with owners and expiry dates
6. Three prioritized next actions with a measurable definition of done
7. Commands executed and checks skipped

Do not claim a CVE is exploitable solely because an audit reports it. Distinguish direct/transitive and runtime/development scope, then request a reachability review for material findings.

## Skill self-validation

Before changing this skill, apply `docs/development/agent-skill-validation.md`. A change is incomplete until structural validation passes and its trigger, negative-trigger, evidence, safety, and output-contract scenarios have recorded results.
