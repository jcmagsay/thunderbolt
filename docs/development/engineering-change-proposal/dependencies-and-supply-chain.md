# Dependencies and Supply Chain

## Observed evidence

- A point-in-time inventory found five Bun lockfiles and two Cargo lockfiles across the application, backend, CLI, web, Pulumi, Tauri, and standalone ACP crate.
- The repository review did not identify a checked-in Dependabot or Renovate configuration or a general multi-lockfile vulnerability audit in CI.
- [`backend/package.json`](../../../backend/package.json) and the root [`package.json`](../../../package.json) demonstrate that targeted overrides are already available for transitive dependency remediation.

These observations do not establish exploitability or whether organization-level dependency alerts are enabled.

## Options for maintainer consideration

- Privately review direct authentication, SSO, and externally reachable dependencies according to the maintainers' severity process.
- Review applicable critical/high direct runtime findings for reachability before publishing details.
- Add scheduled and pull-request audits for all Bun and Cargo lockfiles.
- Initially report existing findings while applying a maintainer-approved policy to newly introduced risk.
- Add owned, expiring exceptions where immediate remediation is not possible.
- Configure Dependabot or Renovate for Bun, Cargo, GitHub Actions, and container bases.
- Apply frozen lockfiles and the release-age quarantine consistently to every Bun package root.
- Use lifecycle-script-free CI installs where builds pass and document any trusted scripts.
- Add Rust advisory, container-image, secret, and infrastructure-as-code scans.
- Generate SBOMs and attestations for published artifacts.
- Measure update latency and vulnerability-remediation performance.

## Possible evidence of completion

- Every manifest is covered by update automation.
- Applicable findings and exceptions have owners, rationale, mitigation, and expiry.
- New risk is handled according to an accepted merge policy.
- Published images are scanned and supply-chain evidence is retained.
