# Testing and Runtime Coverage

## Observed evidence

- A point-in-time inventory found 61 backend test files and 14 Playwright browser spec files.
- Thirty-one backend test files call Elysia's in-process `app.handle()`, providing fast handler coverage without a separate network process.
- [Backend CI](../../../.github/workflows/ci.yml) reruns most backend tests five times in randomized order and isolates WebSocket-heavy tests with bounded retries.
- [Browser CI](../../../.github/workflows/e2e.yml) installs Chromium. The current Playwright projects use Desktop Chrome; Firefox and WebKit projects were not identified.

File counts describe the current branch and do not by themselves measure test effectiveness.

## Options for maintainer consideration

- Add a Firefox smoke suite on pull requests, then evaluate whether its reliability supports making it required.
- Add fuller Firefox and WebKit coverage on `main` or a schedule.
- Classify tests into browser-neutral, browser-sensitive, auth/proxy, sync/storage, and artifact groups to control cost.
- Establish a flake budget and a quarantine policy with owners and expiry dates.
- Add mobile viewports, then evaluate real Tauri desktop and mobile smoke coverage where runners permit.
- Prioritize PowerSync, worker, storage, streaming, auth-cookie, and E2EE behavior that differs by runtime.

## Possible evidence of completion

- Risk-relevant changes receive coverage in the runtimes they can affect.
- Platform-specific gaps remain visible rather than being implied by Chromium results.
- Flake rate and time to signal remain within an accepted budget.
