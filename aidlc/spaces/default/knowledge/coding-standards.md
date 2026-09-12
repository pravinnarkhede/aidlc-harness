# Coding Standards — Golfler / ClubCaddie Estate (observed, not prescribed)

Conversation language: English.

## Purpose and sources

Every convention below was **observed** in each repo's own `code-structure.md`
and `code-quality-assessment.md` from the CodeKB reverse-engineering pass — it
is not generic advice. Organized per repo because the estate spans four
distinct tech stacks (ASP.NET/C#, WPF/C#, Angular/TypeScript, PHP CodeIgniter
x2). A shared "Project-wide facts" section closes the document for
conventions (or their absence) that recur identically across repos.

---

## golfler_asp_2 (ASP.NET / C#, backend of record)

- **Solution shape**: 14 projects in one `Golfler.sln`, sharing two coupling
  libraries (`GolflerShared`, `GolflerDataModel`) referenced by nearly every
  other project — business logic is centralized there rather than in
  per-API controller code; controllers are documented as "thin dispatchers."
- **Naming/organization**: controllers grouped by feature area under
  `Controllers/` per API project; the `Golfler` MVC app additionally
  separates `DTOs/`, `Handlers/`, `Param/`. Convention is **documented, not
  tool-enforced** — five separate hand-authored convention docs exist
  (`CODE_CONVENTIONS.md`, `DB_CONVENTIONS.md`, `API_CONVENTIONS.md`,
  `UI_CONVENTIONS.md`, `MOBILE_CONVENTIONS.md`) plus `.github/
  copilot-instructions.md`, but no `.editorconfig`, StyleCop, or Roslyn
  analyzer package backs any of them.
  → **When touching this repo: read the relevant `docs/*_CONVENTIONS.md`
  file before writing code** — the convention is real and documented, it
  just isn't linter-enforced, so a linter check will not catch a deviation
  from it.
- **ORM pattern**: EF6, Database-First (EDMX) per ADR-001. Two contexts
  coexist against the same database — `GolflerDataModelEntities` (current,
  895 sites) and the ADR-003-deprecated-but-still-active `GolflerEntities`
  (498 sites). New code should target `GolflerDataModelEntities`; do not add
  new usages of `GolflerEntities`.
- **Testing pattern actually used**: 1 test project (`PosApiUnitTest`,
  MSTest), 14 `[TestMethod]`s total, hitting **real databases with hardcoded
  IDs** rather than isolating with mocks/fixtures — i.e. the existing tests
  are integration-style, not unit tests, and this is the pattern the repo's
  own (thin) test suite follows.
- **File-size norm violated at scale**: 20 files exceed 200 KB (largest,
  `Reports.cs`, is 2,484 KB); `RefundController.cs` is 501 KB. Any new work
  in these files should actively avoid growing them further rather than
  treating the existing size as a null baseline.
- **Error handling observed**: 274 empty `catch` blocks among 4,041 total
  `catch(Exception)` blocks — some in payment/billing code. This is an
  anti-pattern present in the codebase, not a convention to imitate; new
  code should log/surface errors per the Construction-phase guardrail rather
  than follow this existing pattern.

## golfler_pos_2 (WPF desktop client, C#)

- **MVVM pattern**: in-house `Bindable`/`ViewModelBase`/`IViewModel` base
  classes — no third-party MVVM framework (no Prism/Caliburn.Micro). Follow
  the existing base-class pattern rather than introducing a framework.
- **ViewModel naming**: `<Feature>ViewModel`, organized by business feature
  area under `ViewModels/` — 577 view models follow this convention
  consistently.
- **Data-access organization anti-pattern**: business domains (orders,
  membership, vouchers, reporting, terminal management) are **not** split
  into per-domain files — they are flattened into one
  `POSDataAccess.cs` (14,780 lines, ~1,062 methods), described in-repo as
  the dominant structural debt item. New data-access code should NOT follow
  this pattern of appending to the god class; prefer a new, appropriately
  named file/class per domain if the surrounding architecture allows it.
- **Integration pattern**: fire-and-forget async
  (`Action<IRestResponse> callback` over `client.ExecuteAsync`) is the
  dominant PosApi call pattern — not `async`/`await` `Task`-based calls. This
  is a repo-wide convention (whether or not it's best practice), so
  consistency with it matters for error-handling expectations elsewhere in
  the codebase.
- **Config pattern**: per-environment blocks centralized in one file
  (`RestConfig.cs`, 2,539 lines) rather than externalized config files — ~35
  near-identical copy-pasted `if/else if` blocks (confirmed bug instance:
  the `CC28` block's `DomainName` incorrectly points at
  `reports-cc2.club-caddie.com`).
- **Testing pattern actually used**: MSTest, ~50 `[TestMethod]`s, 12 of 13
  files covering membership billing; no mocking framework — tests call real
  view-model methods that call the real PosApi (integration-style, same
  pattern as `golfler_asp_2`).
- **Dead code left in place**: commented-out prior implementations are
  retained inline above their live replacements (e.g. `SubmitPosOrderAPI`
  commented out just above `SubmitPosOrderAPI1`) — an observed pattern, not
  one to replicate in new code.

## sgs-cts-angular (Angular 10, CCOnline)

- **Module pattern intended but not followed**: 10 `*.module.ts` files exist
  under `src/app/`, but only 3 routes actually use `loadChildren` to
  lazy-load — the rest of the ~794 components are declared directly on the
  root `AppModule` (2,159 lines, ~816 lines of `declarations:`). **Any new
  feature work should use one of the existing feature modules and
  `loadChildren` lazy-loading rather than adding further declarations to the
  root `AppModule`** — that is the documented-but-unfollowed intended
  pattern, and perpetuating the root-module pattern makes every future build
  more expensive.
- **File naming**: Angular-idiomatic suffixes throughout
  (`.component.ts`, `.service.ts`, `.module.ts`, `.guard.ts`), kebab-case
  file names — with one confirmed inconsistency, `environmentSetup.service.ts`
  uses mixed-case instead of the kebab-case-with-dots convention every other
  service file follows (e.g. `authentication.service.ts`).
- **RxJS usage split**: legacy style imports (`'rxjs/Rx'`,
  `'rxjs/add/operator/map'`) and the modern `rxjs/operators` API are used
  side by side in the same file (`authentication.service.ts`) because
  `rxjs-compat` is still a direct dependency — anyone maintaining this file
  needs to recognize both idioms; new code should use the modern
  `rxjs/operators` style only.
- **Linting toolchain**: TSLint (`tslint.json`), not ESLint — deprecated
  upstream since 2019, not migrated. Follow existing TSLint rules until/
  unless the team decides to migrate; do not introduce ESLint config
  piecemeal.
- **Testing pattern actually used**: Jasmine + Karma, 680 `*.spec.ts` files
  against 1,765 total `.ts` files (~1:1 by file count) — but
  `karma.conf.js` has **no coverage threshold/`check` block configured**, so
  this ratio is not a verified coverage floor, just a file-count fact.
- **Naming oddity carried from the backend contract**: the `Customers/
  CourseCutomers` PosApi endpoint contains a typo ("Cutomers") that is
  preserved verbatim in calling code — do not "fix" this typo locally
  without confirming the backend endpoint name itself, since a client-side
  rename alone would break the call.

## cc_api_manager (PHP CodeIgniter 3, iframes/widgets)

- **Framework convention**: standard CodeIgniter 3 naming — PascalCase
  controller/model class files (`Webapi.php`, `API_Model.php`), snake_case
  method names (`get_api_call`, `card_checkout`). Convention-based routing
  (`/{controller}/{method}/{params}`) is the norm, with ~15 custom rewrites
  in `application/config/routes.php` for cleaner URLs.
- **Environment convention**: each of 11 environment directories under
  `application/config/<env>/` follows a fixed three-file pattern —
  `constants.php`, `config.php`, `database.php`. New environments should
  follow this same three-file shape.
- **Integration anti-pattern to be aware of**: the shared `get_api_call()`
  helper **always** issues an HTTP POST via `CURLOPT_CUSTOMREQUEST`,
  silently ignoring a "get"/"post" hint some callers pass as a third
  argument. Any code (new or reviewed) that assumes GET/POST REST semantics
  against GolferWebAPI through this helper is working from a false model —
  treat every call through it as a POST regardless of the argument passed.
- **Dead code left in place**: `Manager_old.php` retained alongside the
  active `Manager.php`; `Facebook.zip`/`Facebook-old.zip` retained alongside
  the live vendored `Facebook/` SDK directory. Not a pattern to replicate;
  flagged as cleanup candidates but not removed under this stage's
  read-only scan mandate.
- **No package manager in active use**: `composer.json` exists but declares
  an empty `require: {}` — all third-party code (Facebook SDK, Google API
  PHP Client) is vendored directly in-tree rather than pulled via Composer.
  New third-party dependencies should not be silently vendored the same
  way without a team decision, since there is currently no
  dependency-version audit trail at all.
- **Testing pattern**: none — no `tests/`/`spec/` directory, no PHPUnit, no
  test runner. A root `test.php` (302 bytes) exists but is a stray script,
  not a test suite.

## cc_membership_portal (PHP CodeIgniter 3, customer portal)

- **Framework convention**: same CodeIgniter 3 PascalCase-controller /
  `Xxx_model`/`Xxx_Model` convention as `cc_api_manager`, with one confirmed
  inconsistency of its own — casing varies across model file names, e.g.
  `Directory_model.php` vs. `Member_Model.php`. This is pre-existing, not
  something to "fix" as a drive-by cleanup per the team's forbidden-practices
  rule against unrelated renames.
- **Base controller pattern**: all active controllers extend
  `MY_Controller.php` (`application/core/`), which owns shared
  cross-cutting concerns (`_download_excel()` via vendored PHPExcel, shared
  session/auth conventions). New controllers should extend this base rather
  than duplicating session/auth handling.
- **Three parallel auth controllers**: `Auth`, `CustomerAuth`, `McoAuth` —
  each is a distinct login entry-point/tenant flow (standard member,
  customer, MCO), not accidental duplication. Preserve this boundary rather
  than collapsing the three into one unless product explicitly confirms
  they should converge.
- **Dead/parked code directories retained as siblings**:
  `controllers_1/`, `controllers_old/`, `models_1/`, `models_old/`,
  `views_1/`, `views_old/` sit alongside the live `controllers/`, `models/`,
  `views/` directories, confirmed unreferenced from `autoload.php`/
  `routes.php`. Do not edit these; they are historical artifacts, not
  active alternates.
- **Testing pattern**: none — no `tests/` directory, no PHPUnit config.
  CodeIgniter's own vendored `Unit_test` library exists in `system/
  libraries/` but is never invoked from `application/` code.
- **Security posture observed (not a convention to follow)**: CSRF
  protection is explicitly disabled (`config.php:451`,
  `$config['csrf_protection'] = FALSE;`) and the framework's
  `encryption_key` is empty (`config.php:327`). Any new work in this repo
  should not assume CSRF protection is active, and should treat re-enabling
  both as a security fix candidate rather than an unrelated "while we're
  here" change (per the team's forbidden-practices rule) — i.e. raise it
  explicitly rather than bundling a silent fix into an unrelated change.

---

## Project-wide facts (genuinely shared across repos, not per-repo advice)

- **No repo in the 5-repo estate has meaningful automated test coverage.**
  Every repo's own tests (where they exist at all) are integration-style —
  calling real dependencies/databases — rather than isolated unit tests
  with mocks/fixtures. `cc_api_manager` and `cc_membership_portal` have no
  test suite of any kind.
- **No CI/CD pipeline exists in 4 of the 5 repos** (`golfler_pos_2`,
  `sgs-cts-angular`, `cc_api_manager`, `cc_membership_portal`). The one repo
  with a pipeline (`golfler_asp_2`, Jenkins) has no test stage in it either
  — Checkout → Restore → Build → Publish only.
- **No repo has a code linter/analyzer actually wired into a build or CI
  gate.** `golfler_asp_2` and `golfler_pos_2` have no `.editorconfig`/
  analyzer package at all; `sgs-cts-angular` has TSLint configured but no
  evidence it runs pre-merge; `cc_api_manager` and `cc_membership_portal`
  have no linting configuration of any kind.
- **Convention is documented in prose, not enforced by tooling, everywhere
  it exists at all.** Only `golfler_asp_2` has written convention docs
  (`docs/*_CONVENTIONS.md`); the other four repos have no equivalent —
  conventions there are only what the existing code demonstrates.
- **Dead/parked code is left in the tree rather than deleted, in every repo
  that has any** (`golfler_asp_2`'s `CourseWebApi`; `golfler_pos_2`'s
  commented-out API method variants; `sgs-cts-angular`'s disconnected
  `venue-manager/time-line-schedule-view` sub-tree and stale Electron shell;
  `cc_api_manager`'s `Manager_old.php`/archived SDK zips;
  `cc_membership_portal`'s `*_1`/`*_old` directory triplets). Confirm
  intentional deprecation with the team before excluding any of it from
  scope, rather than assuming it is simply "still there and fine" — and
  never delete it as an incidental cleanup inside an unrelated change.
