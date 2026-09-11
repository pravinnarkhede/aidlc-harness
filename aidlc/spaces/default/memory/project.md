# Project-Level Rules

> Project-specific specialisation and corrections. Loaded after `org.md` and
> `team.md` as strict-additive guidance; contradictions with broader policy
> are rejected. Populated by practices-discovery and the self-learning loop.
>
> Use sparingly: most teams don't need a project layer. Reach for it
> only when this specific project needs stable, durable guidance beyond the
> team practice (for example, package-specific release checks or an additional
> regression suite for a legacy component).

## Way of Working

- The backend (`golfler_asp_2`) owns the API contract for every consumer
  repo (`golfler_pos_2`, `sgs-cts-angular`, `cc_mobile_pos`, `cc_api_manager`,
  `cc_membership_portal`, `cc_ios`, `cc_android`). Before changing an API:
  identify existing consumers, confirm whether backward compatibility is
  required, implement and verify the backend change first, only then update
  consumers. Do not let clients assume API shape before the backend contract
  is confirmed.
- Follow existing naming, folder structure, DI, logging, exception-handling,
  and API-response conventions already present in the repo being touched —
  inspect nearby code before writing new code, rather than introducing a new
  pattern the codebase doesn't already use.
- Legacy `golfler_asp_2` code is being migrated .NET Framework → .NET/.NET 10.
  For migrated functionality, preserve legacy behavior: copy/reproduce the
  existing implementation, apply only .NET-compatibility changes, then
  functionally verify against the legacy version. Record every compatibility
  change (legacy behavior, what changed, why, verification) in
  `aidlc/compatibility-changes.md`.

## Walking Skeleton

<!-- Project-specific specialisation. Example: -->
<!-- The walking skeleton must exercise the legacy service adapter as well -->
<!-- as the new service boundary. -->

## Testing Posture

- When migrating a `golfler_asp_2` endpoint legacy → Core, compare HTTP
  status, response structure/property names/values, null handling,
  validation, error behavior, DB side effects, authorization, and
  calculations against the legacy implementation. JSON property ordering is
  not a functional difference; a value or structure difference is and must
  be investigated, not dismissed.
- Regression focus areas across this platform: payments, orders, tee times,
  activities, events, F&B, authentication, authorization, and any other
  high-usage API or frequently used client flow. Changes touching these
  need explicit regression scenarios identified before implementation.

## Deployment

<!-- Project-specific specialisation. -->

## Code Style

<!-- Project-specific specialisation. -->

## Tech Stack

- `golfler_asp_2` backend is mid-migration from .NET Framework to
  .NET/.NET 10 — see "Way of Working" and "Forbidden" for the preservation
  rules that apply to any change touching migrated code.

## Decided

<!-- Decisions made in earlier stages that should not be re-asked. -->
<!-- Format: DECIDED: [decision] (Stage [slug], [date]) -->

## Scope Overrides

<!-- Custom scope rules for this project. -->

## Forbidden

- NEVER redesign, optimize, refactor, or simplify `golfler_asp_2` business
  logic as a side effect of a .NET Framework → .NET/.NET 10 migration —
  functional equivalence with the legacy implementation is the goal; only
  framework/runtime-compatibility changes or explicitly requested behavior
  changes are in scope.
- NEVER silently change an API contract another repo consumes — document
  old contract, new contract, reason, affected consumers, and migration/
  backward-compatibility strategy before changing it.
- NEVER change the `golfler_asp_2` database schema just because a new
  table/model would be architecturally cleaner — only when the feature
  actually requires it.
- NEVER turn a feature change into a "while we're here" cleanup — no
  renaming unrelated classes, no unrelated refactors, no reformatting whole
  files, no unrelated API response changes.
- NEVER commit across independent repos or move files between them when
  working a ticket that touches multiple cloned repos at once — keep
  commits scoped to the repo they actually belong to.

## Mandated

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->
