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

<!-- Project-specific specialisation. Example: -->
<!-- This monorepo requires package-scoped branch names and a package owner -->
<!-- review in addition to the team's normal merge policy. -->

- Ticket-scoped work (feature or bugfix alike) gets its own folder directly
  under `aidlc-harness/`, named exactly by the ticket key — e.g.
  `aidlc-harness/GOLF-123/` — per `auxiliary-ticket-intake.md`'s documented
  precondition. Create the folder first, then start (or point) the session
  at it via `AIDLC_PROJECT_DIR` (and ideally `CLAUDE_PROJECT_DIR`) set to its
  absolute path before running `/aidlc <TICKET-KEY> <description>` — the
  `.claude/` harness install itself always stays at `aidlc-harness/.claude`
  regardless of this redirect. This isolates each ticket's own repos and
  AIDLC state from every other ticket and from the harness root, and is
  never set up mid-workflow by a stage. (affirmed 2026-09-12)

## Walking Skeleton

<!-- Project-specific specialisation. Example: -->
<!-- The walking skeleton must exercise the legacy service adapter as well -->
<!-- as the new service boundary. -->

## Testing Posture

<!-- Project-specific specialisation. -->

## Deployment

<!-- Project-specific specialisation. -->

## Code Style

<!-- Project-specific specialisation. -->

## Tech Stack

<!-- Technology choices locked for this project. -->

- The Golfler/ClubCaddie estate spans 4 tech stacks across 5 repos:
  ASP.NET Web API 2 / EF6 / C# (`golfler_asp_2`), WPF/C# desktop
  (`golfler_pos_2`), Angular 10 (`sgs-cts-angular`), and PHP CodeIgniter 3
  x2 (`cc_api_manager`, `cc_membership_portal`).

## Decided

<!-- Decisions made in earlier stages that should not be re-asked. -->
<!-- Format: DECIDED: [decision] (Stage [slug], [date]) -->

- `golfler_asp_2` owns the API contract for all other repos in the
  estate — never change PosApi or GolferWebAPI response shapes, endpoint
  paths, or auth headers without checking every known consumer first:
  `golfler_pos_2`, `sgs-cts-angular` (PosApi); `cc_api_manager`,
  `cc_membership_portal` (GolferWebAPI); plus the cataloged-but-unscanned
  `cc_mobile_pos_flex`/`fnb`, `cc_ios`, `cc_android`. (DECIDED: onboarding
  synthesis, 2026-09-12)
- `golfler_asp_2` exposes TWO incompatible response envelopes that are both
  live in production: PosApi's `Result` (lowercase `record`/`Error`) and
  GolferWebAPI's `Response`/`Base.cs` (uppercase `Record`/`Message`). Any
  work touching either envelope must treat both shapes as constraints, not
  assume one is legacy — this is the repo's own risk register finding
  R10 (score 15/27). (DECIDED: onboarding synthesis, 2026-09-12)
- `CourseWebApi` in `golfler_asp_2` is documented deprecated (ADR-004,
  target October 2025) but its 5 controllers are still present in the
  scanned branch as of 2026-09-12, and no known consumer was found. Confirm
  with the team whether it is truly dead before touching or removing it —
  do not assume it is safe to delete without that confirmation. (DECIDED:
  onboarding synthesis, 2026-09-12)
- Payment processing and multi-tenant `CourseId` data isolation both score
  27/27 (the maximum) on `golfler_asp_2`'s own risk register. Any Unit of
  work touching either area requires explicit human sign-off before
  Construction proceeds, per the repo's own stated rule. (DECIDED:
  onboarding synthesis, 2026-09-12)
- `sgs-cts-angular` (CCOnline) serves 40+ distinct club/course tenants
  from one codebase via a hardcoded table in `environmentSetup.service.ts`
  (two hosting families: `jonassynergy.com` and `club-caddie.com`).
  Onboarding a new tenant today requires a code change and redeploy — this
  is a business-agility constraint to flag to product/ops, not assume is
  already handled dynamically. (DECIDED: onboarding synthesis, 2026-09-12)
- `cc_api_manager` uses a single shared API key to authenticate all 11
  club/environment deployments to GolferWebAPI. A credential compromise or
  required rotation is an all-clubs event, not an isolated one — treat any
  credential-related change here as cross-club in scope. (DECIDED:
  onboarding synthesis, 2026-09-12)

**Regression Focus Areas** (from reverse-engineering, cross-repo) — prioritize
for manual verification and/or new test authoring before or during any
Construction-phase work that touches them:
1. Payment processing in `golfler_asp_2` (`Payment.cs`, `PaymentMethodController.cs`,
   `RefundController.cs`) — 8+ coexisting gateways, near-zero test coverage.
2. Multi-tenant/multi-club `CourseId` data isolation in `golfler_asp_2` —
   no base-query filter, no compile-time enforcement, across 1,393 EF
   context instantiations.
3. Any change to PosApi or GolferWebAPI response envelopes or endpoint
   contracts — both shapes are live and consumed by multiple repos.
4. `cc_membership_portal`'s CSRF-disabled, empty-encryption-key
   configuration (`config.php`) — a customer-facing portal handling
   payments and PII with both protections off.
5. `cc_api_manager`'s disabled TLS certificate verification on every
   GolferWebAPI call.
6. The three parallel tee-booking API generations in GolferWebAPI
   (`TeeBookingController`, `CustomerTeeTimesController`,
   `TeeTimesV2Controller`, `TeeTimesV3Controller`) — confirm which are
   still live before consolidating or modifying any one of them.

## Scope Overrides

<!-- Custom scope rules for this project. -->

- `golfler_asp_2` active-project scope (affirmed by the user, 2026-09-12):
  of its 14 .NET projects, only **PosApi**, **GolferWebAPI**,
  **GolflerDataModel**, **GolflerDB**, **AzureUtilities**, and
  **GolflerShared** are actually in active use. The remaining projects
  (`CourseWebApi`, `Golfler`, `CCACHWebhook`, `VoucherExpirationWindowsService`,
  `PosApiUnitTest`, `MaintenanceConsoleApp`, `CCU`, `HubSpotIntegration`,
  `RangeExpress`) are unused or rarely used. Default reverse-engineering,
  Unit boundaries, and design work should focus on the active six; only
  scan or design against a deprioritized project when explicitly asked.

## Forbidden

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: NEVER [behavior] (affirmed [date]) -->
<!-- Example: NEVER throw exceptions across service layer boundaries (affirmed 2026-05-17) -->

- NEVER touch `golfler_asp_2`'s deprioritized projects (`CourseWebApi`,
  `Golfler`, `CCACHWebhook`, `VoucherExpirationWindowsService`,
  `PosApiUnitTest`, `MaintenanceConsoleApp`, `CCU`, `HubSpotIntegration`,
  `RangeExpress`) as part of a bugfix or feature scope — the existing
  reverse-engineering CodeKB for `golfler_asp_2` stays as-is (full-repo
  scan, already published, not being trimmed), but new Units, code changes,
  or focused scans must stay within the active six
  (PosApi, GolferWebAPI, GolflerDataModel, GolflerDB, AzureUtilities,
  GolflerShared) unless the user explicitly names a deprioritized project
  for that piece of work (affirmed 2026-09-12).

## Mandated

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->
