# Quality Gates and API Contracts

## Observed evidence

- [PR Metrics](../../../.github/workflows/pr-metrics.yml) collects coverage, bundle, Lighthouse, accessibility, and Web Vitals signals, with several collection and assertion steps configured as `continue-on-error`.
- Swagger is generated at runtime when `SWAGGER_ENABLED=true`; [`swagger.test.ts`](../../../backend/src/swagger.test.ts) verifies exposure is enabled or disabled but does not test contract completeness.
- The repository review did not identify a checked-in OpenAPI baseline, contract-diff check, Postman collection, Newman job, or equivalent external API scenario suite.
- [Playwright](../../../playwright.config.ts) already provides a possible API-request runner, so closing the scenario gap does not inherently require a new tool.

## Existing API testing layers

- Bun tests provide fast handler-level coverage of authentication decisions, validation, database behavior, error mapping, streaming, and proxy policy. Most route tests construct a `Request` and call Elysia's in-process `app.handle()`.
- Backend CI runs most tests five times in randomized order and handles WebSocket-heavy tests separately with bounded retries.
- Playwright starts real backend processes for selected OIDC, SAML, ACP, proxy, and browser journeys.

These layers provide implementation feedback but do not establish that a built or deployed backend satisfies a reusable external API contract. A Postman/Newman suite could add that perspective, but it would complement rather than replace the Bun suite.

## Options for maintainer consideration

- Make formatting and license headers explicit CI gates.
- Define coverage policy by critical subsystem rather than one repository-wide percentage.
- Export and lint the generated OpenAPI document, report breaking contract changes, and apply a maintainer-approved review policy.
- Add a minimal black-box API scenario suite against a separately running backend.
- Generate or validate any Postman collection from OpenAPI rather than maintaining a second manual contract.
- Run API scenarios against an ephemeral local stack first; evaluate preview and release-candidate runs after defining credential, test-data, teardown, and cost controls.
- Establish bundle and Lighthouse regression budgets with an exception mechanism.
- Replace retry-normalized flakes with tracked root-cause fixes.
- Avoid full preview deployments for documentation-only changes where workflow architecture permits.
- Verify that PR Metrics uses the intended preview URL and collects Lighthouse reports.

## Candidate external scenarios

A small black-box suite could evaluate:

- Health and configuration
- Anonymous sign-in and session handling
- Expected authorization failures
- CORS preflight and cookie behavior
- Validation and safe error responses
- PowerSync token issuance
- Device and encryption lifecycle boundaries
- Proxy header and SSRF policy
- One streaming response

Mock upstreams and ephemeral test data would keep automated runs independent of paid providers and production credentials.

## Possible evidence of completion

- Reliable quality and contract regressions fail according to documented policies.
- Every bypass is owned and expires.
- CI duration, flake rate, and preview cost remain visible and within accepted budgets.
