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

## Walking Skeleton

<!-- Project-specific specialisation. Example: -->
<!-- The walking skeleton must exercise the legacy service adapter as well -->
<!-- as the new service boundary. -->

## Testing Posture

- Regression focus areas across this platform: payments, orders, tee times,
  activities, events, F&B, authentication, authorization, and any other
  high-usage API or frequently used client flow. Changes touching these
  need explicit regression scenarios identified before implementation.

## Deployment

<!-- Project-specific specialisation. -->

## Code Style

<!-- Project-specific specialisation. -->

## Tech Stack

<!-- Technology choices locked for this project. -->

## Decided

<!-- Decisions made in earlier stages that should not be re-asked. -->
<!-- Format: DECIDED: [decision] (Stage [slug], [date]) -->

## Scope Overrides

<!-- Custom scope rules for this project. -->

## Forbidden

- NEVER change the `golfler_asp_2` database schema just because a new
  table/model would be architecturally cleaner — only when the feature
  actually requires it.

## Mandated

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->
