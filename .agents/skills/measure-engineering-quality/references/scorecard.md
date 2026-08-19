# Engineering Quality Scorecard

Score each domain from 0–4 using repository evidence.

| Score | Meaning                                                               |
| ----- | --------------------------------------------------------------------- |
| 0     | No control or evidence                                                |
| 1     | Documented intent or manual process                                   |
| 2     | Automated reporting; failure does not reliably block                  |
| 3     | Enforced for material paths with known coverage gaps                  |
| 4     | Enforced comprehensively, measured over time, and operationally owned |

## Domains

| Domain                        | Weight | Score 3 evidence                                                                                                   | Score 4 evidence                                                                            |
| ----------------------------- | -----: | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| Dependency security           |     20 | All lockfiles audited; new critical/high findings block                                                            | Runtime reachability, expiring exceptions, remediation SLA, trend reporting                 |
| Dependency maintenance        |     10 | Automated semver updates cover every package root and Actions                                                      | Update latency/SLA measured; grouped updates and ownership are effective                    |
| Install supply chain          |     10 | Frozen locks, release-age policy, reviewed lifecycle-script policy                                                 | Consistent roots, provenance/attestations, trust changes reviewed and tested                |
| Deterministic CI quality      |     15 | Type, lint, tests, builds, and relevant platform jobs are required                                                 | Coverage/performance budgets and flake rate are owned and improving                         |
| Browser/runtime coverage      |     15 | Chromium and Firefox required; WebKit scheduled                                                                    | Required risk-based browser matrix plus real Tauri/mobile evidence                          |
| Security detection            |     10 | SAST, dependency, secret, Rust, container, and IaC coverage                                                        | SBOMs, provenance, DAST, triage SLA, and measured false-positive handling                   |
| Cost/infrastructure hygiene   |      5 | Timeouts, cancellation, retention, teardown, and budget alerts                                                     | Per-workflow/resource cost trends and owners; path-aware avoidance                          |
| Monitoring/reliability        |      5 | Health, logs, traces, errors, dashboards, and alerts cover key paths                                               | SLOs, on-call ownership, recovery drills, and release health are measured                   |
| Privacy/security architecture |      5 | Data flows, consent, telemetry minimization, and threat boundaries documented/tested                               | Independent review, retention enforcement, incident drills, verified controls               |
| Agent context and skills      |      5 | Skills have routing context, validation scenarios, evidence contracts, and provider-neutral canonical instructions | Cross-interface forward-test metrics, regression fixtures, ownership, and freshness reviews |

Calculate the weighted score as:

```text
sum(domain score / 4 × weight)
```

Do not average away a critical failure. Add a red status when any of these holds:

- Known unexcepted critical runtime vulnerability
- A security gate is disabled or silently allowed to fail
- A production deployment lacks deterministic teardown or bounded execution
- A claimed privacy/security control lacks implementation evidence
- A skill reports success when required evidence is unavailable

## Baseline record

Record baselines in the engineering improvement tracking issue or its linked artifact with:

- UTC date and commit SHA
- Tool versions
- Raw machine-readable reports or artifact links
- Domain scores and evidence
- Accepted exceptions and expiry
- Measurement limitations

Never overwrite history. Append a new measurement so trends remain auditable.
