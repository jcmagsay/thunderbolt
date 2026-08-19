# Measurement and Agent Guidance

## Observed evidence

- [`AGENTS.md`](../../../AGENTS.md) contains repository-wide contributor guidance.
- The [`measure-engineering-quality` skill](../../../.agents/skills/measure-engineering-quality/SKILL.md) defines an evidence workflow and weighted scorecard.
- [Agent Skill Validation](../agent-skill-validation.md) defines structural, trigger, safety, evidence, portability, and forward-test criteria. The current skill record still marks portability and fresh-agent forward testing as pending.

## Optional measurement cadence

If maintainers adopt the measurement approach, run the repository's `measure-engineering-quality` skill:

- After a relevant architecture decision is accepted
- After material CI, security, browser, telemetry, or infrastructure changes
- Before a production-readiness milestone when requested by its owner
- Periodically while maintainers are actively tracking applicable findings or adopted proposals

Track domain score, baseline delta, applicable finding counts, exception age, patch adoption time, CI duration, flaky-test rate, browser pass rate, preview cost, SLO compliance, recovery-drill status, and agent-skill validation status.

## Proposed agent guidance

- `AGENTS.md` routes quality-baseline requests to the repository skill.
- `.agents/skills/measure-engineering-quality/` defines the evidence workflow and weighted scorecard.
- [Agent Skill Validation](../agent-skill-validation.md) applies to repository-owned skills.
- Canonical guidance is provider and model agnostic; optional adapters cannot be required for execution or redefine the workflow.
- Measurements preserve raw evidence, declare unknowns, and do not produce a passing result when required evidence is unavailable.

## Possible evidence of completion

Two independent, differently configured agent interfaces using the same commit and evidence agree on mechanical facts and domain scores. Any judgment difference is visible in cited rationale.
