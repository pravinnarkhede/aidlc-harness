# Team-Level Rules

> This team's affirmed practices and corrections. Loaded after `org.md` as
> strict-additive guidance; contradictions with broader policy are rejected.
> Populated by the practices-discovery affirmation gate. Edit at the gate,
> not directly.

## Way of Working

<!-- Affirmed during practices-discovery. Example: -->
<!-- We use GitHub Flow with feature branches. Branches live 3-5 days max. -->
<!-- Hotfixes branch from main and merge back via expedited review. -->

- Follow existing naming, folder structure, DI, logging, exception-handling,
  and API-response conventions already present in the repo being touched —
  inspect nearby code before writing new code, rather than introducing a new
  pattern the codebase doesn't already use.

## Walking Skeleton

<!-- Affirmed during practices-discovery. Example: -->
<!-- We don't run a walking skeleton — our deployment pipeline is mature -->
<!-- and the slice cost outweighs the value at our maturity stage. -->

## Testing Posture

<!-- Affirmed during practices-discovery. Example: -->
<!-- We use BDD. Specifications drive scenarios; scenarios drive code. -->
<!-- Each Unit ships with feature files in /features/. -->

## Deployment

<!-- Affirmed during practices-discovery. -->

## Code Style

<!-- Team-specific conventions beyond the linter. Example: -->
<!-- - Prefer named exports over default exports -->
<!-- - All async functions return Result<T, E>, never throw -->

## Forbidden

<!-- Team-specific forbidden patterns -->

- NEVER silently change an API contract another repo/service consumes —
  document old contract, new contract, reason, affected consumers, and
  migration/backward-compatibility strategy before changing it.
- NEVER turn a feature change into a "while we're here" cleanup — no
  renaming unrelated classes, no unrelated refactors, no reformatting whole
  files, no unrelated response/API changes.
- NEVER commit across independent repos or move files between them when a
  change touches multiple cloned repos at once — keep commits scoped to the
  repo they actually belong to.
- NEVER disable TLS/SSL certificate verification on any outbound HTTP
  call, in any language or client library, regardless of environment
  (confirmed live in this estate: `cc_api_manager`'s GolferWebAPI client;
  affirmed 2026-09-12).
- NEVER commit secrets, API keys, or credentials to source — even a
  "shared/non-production-looking" key — use environment variables or a
  secrets manager instead (confirmed live in this estate across
  `cc_api_manager`, `cc_membership_portal`, `sgs-cts-angular`, and
  `golfler_pos_2`; affirmed 2026-09-12).
- NEVER disable CSRF protection or ship an empty encryption key on a
  customer-facing application that handles payments or PII, without an
  explicit, documented, time-bound security exception signed off by
  devsecops/compliance (affirmed 2026-09-12).

## Mandated

<!-- Team-specific mandates -->

- ALWAYS write a human-reviewed test plan before any Construction-phase
  work against a repo found to have zero automated test coverage and no
  CI/CD gate — do not assume the org's default 80% coverage-floor language
  alone covers this; the floor presumes an existing suite and pipeline to
  build on, and a codebase with neither needs that foundation established
  first, scoped explicitly as first-class work (affirmed 2026-09-12).
- ALWAYS confirm actual removal/traffic-cutover status with the team when
  a codebase's own documentation (ADRs, risk registers, known-issues files)
  states a component is deprecated, before treating it as dead — a
  documented deprecation and an actual one are not the same fact, and code
  scans can only observe presence, not intent (affirmed 2026-09-12).
- ALWAYS treat every response-shape or endpoint change to a shared backend
  contract as requiring an explicit downstream-consumer impact check before
  it ships, when multiple client applications depend on that contract —
  "which repos call this" is not optional research, it is a precondition
  for the change (affirmed 2026-09-12).

## Corrections

<!-- Self-learning loop appends here. -->
