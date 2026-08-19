# Operations, Privacy, and Security

## Observed evidence

- The backend includes structured logging, a health endpoint, optional OpenTelemetry export, and inference observability.
- [`TELEMETRY.md`](../../../TELEMETRY.md) documents product and sync events, while [`SECURITY.md`](../../../SECURITY.md) defines private vulnerability reporting.
- The repository review did not identify production SLO definitions, alert thresholds, on-call routing, provisioned dashboards, or documented recovery-drill results. These controls may exist outside the repository.

## Options for maintainer consideration

- Define SLIs and SLOs for availability, latency, inference failures, authentication, sync lag, and recovery.
- Provision dashboards and actionable alerts with an owning response path.
- Add dependency/provider cost signals and budget alerts.
- Test backup restore, device loss, E2EE recovery, mixed versions, and failed deployments.
- Maintain a telemetry inventory covering consent, retention, identifier classification, and prompt/response exclusion.
- Seek independent security and cryptography review for production claims where appropriate.

## Possible evidence of completion

- SLOs and alerts are exercised rather than only documented.
- Restore and recovery drills retain evidence.
- The privacy inventory matches implemented behavior.
- Production claims cite applicable independent review and operational results.
