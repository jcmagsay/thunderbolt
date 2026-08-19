# Governance and Baseline

## Observed evidence

- The repository defines CI workflows, permissions, timeouts, concurrency, and artifacts under [`.github/workflows/`](../../../.github/workflows/).
- Repository files cannot establish branch-protection requirements, organization-level security settings, external budgets, or current ownership.

## Options for maintainer consideration

- Consider separate architecture-decision proposals for dependency security and cross-browser compatibility gates.
- Assign owning areas for dependency security, CI/browser quality, infrastructure cost, observability, privacy/security, and agent quality.
- Record the first scorecard using the repository's `measure-engineering-quality` skill.
- Define runtime/development reachability labels, remediation SLAs, and an expiring exception schema.
- Confirm GitHub branch protection, required checks, Dependabot/security settings, budget alerts, and environment reviewers outside the repository.

## Possible evidence of completion

- A dated baseline includes the commit SHA and raw evidence.
- Each accepted control and critical finding has an owner.
- Maintainers have accepted the severity and exception policies.
