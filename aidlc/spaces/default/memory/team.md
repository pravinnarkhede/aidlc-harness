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

## Mandated

<!-- Team-specific mandates -->

## Corrections

<!-- Self-learning loop appends here. -->
