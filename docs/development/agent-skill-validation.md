# Agent Skill Validation Standard

Every repository-owned agent skill must have evidence that it triggers correctly, follows project safety rules, and produces useful results. A valid folder structure alone is not sufficient.

Skills and agent guidance must support Thunderbolt's bring-your-own (BYO) approach. The canonical instructions must be usable across local and hosted agents without requiring a particular model, provider, proprietary invocation syntax, or paid service. Provider-specific adapters may exist only as optional metadata; they must not contain behavior that is absent from the canonical skill.

## Required validation

| Area                | Acceptance criterion                                                                                                                                             |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Structure           | `SKILL.md` has valid YAML with only `name` and `description`; skill and directory names match; referenced files exist; the skill validator passes                |
| Trigger             | At least two realistic prompts that should invoke the skill cause the intended workflow to be selected                                                           |
| Negative trigger    | At least two nearby but out-of-scope prompts do not cause unnecessary invocation                                                                                 |
| Instructions        | A fresh agent can complete the workflow without undocumented tribal knowledge or access to the author’s conversation                                             |
| Evidence            | Material conclusions cite repository files, commands, reports, or external primary sources; unavailable evidence is reported as unknown                          |
| Output contract     | Required fields, severity/score definitions, and failure states are present and unambiguous                                                                      |
| Safety              | The skill does not deploy, publish, delete, spend money, expose secrets, or mutate external state without explicit authorization                                 |
| Failure posture     | Missing tools, network access, permissions, or baselines produce a visible incomplete/blocked result rather than a false pass                                    |
| Determinism         | Repeated runs over the same fixture agree on mechanical facts; thresholds and formulas are explicit                                                              |
| Portability         | The canonical skill runs with at least one local-capable and one hosted-capable agent interface, or equivalent test harnesses, without changing its instructions |
| Provider neutrality | Instructions describe required capabilities and outcomes rather than assuming a vendor, model, proprietary command, or hosted service                            |
| Forward test        | A fresh agent completes at least one representative task using only the skill and raw task artifacts                                                             |
| Regression          | Fixes for observed skill failures add or update a reusable scenario or fixture                                                                                   |
| Freshness           | The skill names an owner or owning area and a review trigger, such as workflow, schema, provider, or policy changes                                              |

## Validation record

Store the validation record beside the skill as `references/validation.md` when it is specific to that skill. Keep it concise and include:

```markdown
# Validation

- Skill version: <commit SHA>
- Date: <UTC date>
- Validator: <person or agent>

| Scenario              | Expected | Result    | Evidence |
| --------------------- | -------- | --------- | -------- |
| Trigger: ...          | ...      | Pass/Fail | ...      |
| Negative trigger: ... | ...      | Pass/Fail | ...      |
| Safety: ...           | ...      | Pass/Fail | ...      |
| Failure posture: ...  | ...      | Pass/Fail | ...      |
| Portability: ...      | ...      | Pass/Fail | ...      |
| Forward test: ...     | ...      | Pass/Fail | ...      |
```

Do not mark a skill production-ready while a required scenario is failing. If a forward test could invoke paid services, mutate infrastructure, or take substantial time, request approval first and use a safe fixture where possible.

Portability validation must not require calling a paid model. A deterministic harness, a local model, or two independently configured agent interfaces may be used. Record the interfaces and capabilities tested, not a preferred vendor or model ranking.

## Review expectations

Skill PRs should explain:

- What user requests trigger the skill
- What it deliberately does not cover
- What deterministic checks or scripts it uses
- Which capabilities it requires and how provider-specific adapters remain optional
- How success is measured
- Which validation scenarios ran
- What change should trigger revalidation

Review skill instructions as executable policy. A vague instruction that can silently produce a false green result is a correctness defect.
