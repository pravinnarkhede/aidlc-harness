# Architecture Overview — Golfler / ClubCaddie Estate

Conversation language: English.

## Sources

- [catalog] `aidlc/spaces/default/knowledge/repo-catalog.md` — authoritative for **intent** (declared role/scope per repo)
- [scan] Per-repo CodeKB (9-artifact reverse-engineering output) under
  `stable-codebase/aidlc/spaces/default/codekb/<repo>/` for all 5 cataloged
  repos — authoritative for **reality** (what the code actually does). Cited
  inline as `[scan:<repo>]`.

This document combines both. Where they agree, it says so plainly. Where they
disagree — or the scan found something the catalog doesn't mention — it is
called out explicitly as a **discrepancy**, not silently resolved in favor of
either source.

---

## 1. The five repos, catalog role vs. scan reality

| Repo | Catalog role (intent) | Scan reality | Agreement? |
|---|---|---|---|
| `golfler_asp_2` | "Backend — all APIs (PosApi, GolferWebAPI), database, and shared business logic. Owns the API contract every other repo consumes." | Confirmed: a 14-project modular monolith hosting **three** API surfaces (PosApi, GolferWebAPI, and the deprecated CourseWebApi), the shared `GolflerShared`/`GolflerDataModel` libraries, EF6-over-SQL-Server, and 8+ payment gateway integrations. [scan:golfler_asp_2] | Agrees, with one addition: the catalog doesn't mention CourseWebApi still existing, or that PosApi and GolferWebAPI are contractually **incompatible** with each other (see §2). |
| `golfler_pos_2` | "Staff desktop POS terminal (WPF). Consumes PosApi from golfler_asp_2." | Confirmed: pure WPF client, ~1,062 PosApi call sites in a single `POSDataAccess.cs` god class, no business logic or data of its own beyond local SQLite/EF6 caching. [scan:golfler_pos_2] | Agrees fully. |
| `sgs-cts-angular` (CCOnline) | "CCOnline — web-based staff management (Angular). Consumes PosApi from golfler_asp_2." | Confirmed: single-module Angular 10 SPA, 1,119 PosApi call sites, serving **40+ tenants** (two hosting families: `jonassynergy.com` "Synergy" and `club-caddie.com` "CC") from one codebase via a hardcoded tenant table. [scan:sgs-cts-angular] | Agrees, with an important addition: the catalog entry doesn't mention the multi-tenant fan-out (40+ club/course tenants sharing one deployable) or the app's direct payment-gateway integrations (Spreedly, First Data/eMerchant) alongside PosApi. |
| `cc_api_manager` | "iFrames, API Manager, and standalone booking widgets embedded on club websites (PHP CodeIgniter). Consumes GolferWebAPI from golfler_asp_2." | Confirmed: CodeIgniter 3 monolith, one codebase deployed per club/environment (11 environment configs), all GolferWebAPI traffic funneled through a single `get_api_call()` helper that always issues HTTP POST regardless of caller intent. [scan:cc_api_manager] | Agrees fully on role; scan adds that a **single shared API key** authenticates every club/environment to GolferWebAPI — a cross-club blast-radius fact the catalog doesn't state. |
| `cc_membership_portal` | "Customer Portal — online member self-service (PHP CodeIgniter). Consumes GolferWebAPI from golfler_asp_2." | Confirmed: CodeIgniter 3 monolith, 139 GolferWebAPI call sites, three parallel auth controllers (`Auth`, `CustomerAuth`, `McoAuth`), no persistence of its own. [scan:cc_membership_portal] | Agrees fully on role; scan adds CSRF protection is disabled and the encryption key is empty — a security posture fact with no catalog counterpart. |

### Repos named in the catalog but commented out / not yet scanned

The catalog file also lists (commented out, i.e. **not yet part of this
5-repo synthesis**): `cc_mobile_pos_flex`/`cc_mobile_pos_fnb` (React Native,
PosApi consumers), `cc_ios` (Swift, GolferWebAPI consumer), `cc_android`
(Kotlin, GolferWebAPI consumer). These are real, cataloged consumers of both
API surfaces that this synthesis has no code-level visibility into. Any
architectural decision made here about PosApi or GolferWebAPI contracts
should account for these undocumented-in-scan consumers before being treated
as complete.

---

## 2. Cross-repo architectural fact: two incompatible API contracts, both live

`golfler_asp_2` is confirmed by every downstream repo's scan as the single
backend of record, exactly as the catalog states. But the scans reveal a
fact **no single downstream repo's catalog entry surfaces on its own**,
because it only becomes visible by reading `golfler_asp_2` together with its
consumers: **`golfler_asp_2` exposes two live, incompatible JSON response
envelope shapes for what is nominally one API contract**, and both shapes are
actively depended upon by multiple downstream repos today.

| | PosApi | GolferWebAPI |
|---|---|---|
| Envelope type | `Result` | `Response` / `Base.cs` |
| Casing | lowercase `record` | uppercase `Record` |
| Error field | `Error` | `Message` |
| Consumed by (in this 5-repo set) | `golfler_pos_2` (WPF POS), `sgs-cts-angular` (CCOnline) | `cc_api_manager` (widgets), `cc_membership_portal` (customer portal) |
| Consumed by (cataloged, not scanned) | `cc_mobile_pos_flex`/`fnb` (mobile POS) | `cc_ios`, `cc_android` (mobile customer apps) |

This is `golfler_asp_2`'s own risk register finding (KNOWN_RISKS R10,
score 15/27), and it is a **cross-repo architectural fact, not a
`golfler_asp_2`-internal detail**: any future work to standardize, version,
or consolidate the API contract must treat both envelope shapes as
simultaneously live constraints across the whole estate — a fix that
"cleans up" one shape by aligning it with the other would silently break
every consumer of the shape being changed. [scan:golfler_asp_2 →
api-documentation.md, business-overview.md]

## 3. `CourseWebApi` — documented deprecated, still present

`golfler_asp_2`'s own ADR-004 (in-repo) targets removal of `CourseWebApi` by
October 2025; as of the scan date (2026-09-12) its 5 controllers are still
present in the `release_5_7` branch. [scan:golfler_asp_2] Neither the repo
catalog nor any downstream repo's scan names an active caller of
`CourseWebApi` — it does not appear as a dependency in `golfler_pos_2`,
`sgs-cts-angular`, `cc_api_manager`, or `cc_membership_portal`'s API/
dependency documents. This is consistent with a stalled-but-safe deprecation
(traffic already cut over, cleanup pending) but is **not confirmed** either
way; flagged as an open question for the team (see §6).

## 4. Cross-repo component diagram

```mermaid
graph TB
    subgraph Staff-facing clients
        POS["golfler_pos_2\nWPF desktop POS\n~1,062 PosApi calls"]
        CCOnline["sgs-cts-angular\nCCOnline (Angular 10)\n1,119 PosApi calls\n40+ tenants (Synergy / CC)"]
    end

    subgraph Customer-facing clients
        ApiMgr["cc_api_manager\nPHP CodeIgniter 3\niframes/widgets, 40 endpoints\n11 environments"]
        MemberPortal["cc_membership_portal\nPHP CodeIgniter 3\nmember self-service\n139 endpoints"]
    end

    subgraph Not scanned this pass, cataloged only
        MobilePOS["cc_mobile_pos_flex/fnb\nReact Native"]
        iOS["cc_ios\nSwift"]
        Android["cc_android\nKotlin"]
    end

    subgraph golfler_asp_2["golfler_asp_2 — backend of record"]
        PosApi["PosApi\n294 controllers\nResult envelope"]
        GolferWebAPI["GolferWebAPI\n71 controllers\nResponse/Base.cs envelope"]
        CourseWebApi["CourseWebApi\nDEPRECATED (ADR-004)\n5 controllers, still present"]
        Shared["GolflerShared + GolflerDataModel\n(coupling hub)"]
    end

    SQL[("SQL Server\nGolflerDB")]

    POS -->|REST, ApiKey+Token+Version headers| PosApi
    CCOnline -->|REST, per-tenant baseUrl| PosApi
    MobilePOS -.->|cataloged, not scanned| PosApi

    ApiMgr -->|REST, shared APIKey, TLS verify OFF| GolferWebAPI
    MemberPortal -->|REST, 139 call sites| GolferWebAPI
    iOS -.->|cataloged, not scanned| GolferWebAPI
    Android -.->|cataloged, not scanned| GolferWebAPI

    PosApi --> Shared
    GolferWebAPI --> Shared
    CourseWebApi --> Shared
    Shared --> SQL
```

## 5. Cross-repo security and tech-debt findings

Each finding below recurs across multiple repos or has cross-repo blast
radius; per-repo detail lives in each repo's own `code-quality-assessment.md`.

| Finding | Repos affected | Cross-repo implication |
|---|---|---|
| **Hardcoded secrets committed to source** | `cc_api_manager` (GolferWebAPI `APIKey`, one value shared across all 11 club environments), `cc_membership_portal` (`FB_SECRET`, `G_SECRET`, `G_DEVKEY`, `RECAPTCHA_SECRET_KEY`, `IP_API_KEY` in `constants.php`), `sgs-cts-angular` (Spreedly and First Data payment-gateway credentials in `authentication.service.ts`), `golfler_pos_2` (~35 environment blocks in `RestConfig.cs`, each with plaintext `ApiKeyValue` and shared production Spreedly credentials) | A credential leak or required rotation in any one of these is effectively an **all-clubs, all-environments event** for that integration — there is no per-club/per-tenant credential isolation in the client repos scanned. |
| **TLS certificate verification disabled** | `cc_api_manager` (`CURLOPT_SSL_VERIFYPEER = false` on every GolferWebAPI call) | Every request from this repo to GolferWebAPI — the shared backend of record — is exposed to MITM; this is a live, exploitable gap on a production traffic path, not a theoretical one. |
| **Zero effective automated test coverage** | All 5 repos. `golfler_asp_2`: 14 `[TestMethod]`s across ~380 controllers, hitting real DBs with hardcoded IDs. `golfler_pos_2`: ~50 tests, no mocking, integration-style against a live/configured environment. `sgs-cts-angular`: 680 spec files exist but no coverage floor enforced, ratio-only, not depth-verified. `cc_api_manager`: no test directory, no framework, at all. `cc_membership_portal`: no test directory, no framework, at all. | Every repo in the estate can be changed today with **no regression safety net**. This is the single most consistent finding across the whole 5-repo scan and should be treated as a first-class Construction-phase risk for any Unit touching any of these repos, not folded into a generic legacy-code caveat. |
| **No CI/CD pipeline found** | `golfler_pos_2`, `sgs-cts-angular`, `cc_api_manager`, `cc_membership_portal` — 4 of 5 repos have **no** CI/CD configuration of any kind (no GitHub Actions, GitLab CI, Jenkinsfile, Travis, Bitbucket Pipelines). Only `golfler_asp_2` has any pipeline (Jenkins), and even that pipeline has **no test stage** (Checkout → Restore → Build → Publish only). | Nothing in the estate mechanically blocks a broken or insecure change from reaching a deployable artifact. Combined with the test-coverage finding above, this means the org's own "linter/tests run in CI, failure blocks the PR" default (`org.md`) is **not currently met anywhere in this estate** — a gap to surface to delivery/devsecops before any Construction work proceeds, not something to assume is covered elsewhere without confirmation. |
| **Payment processing — highest risk, concentrated in `golfler_asp_2`** | `golfler_asp_2` scores this 27/27 on its own risk register (8+ coexisting gateways, near-zero test coverage, `TransactionScope` used in only 4 files). Every client repo (`golfler_pos_2`, `sgs-cts-angular`, `cc_api_manager`, `cc_membership_portal`) additionally makes its **own** direct payment-gateway calls (Spreedly, First Data/eMerchant, CardConnect, Clover, 1stPayGateway) that bypass `golfler_asp_2` entirely. | Payment risk is not contained to one repo — it is distributed across at least 5 separate, independently-maintained payment integration surfaces with no shared hardening, and no repo in the set has meaningful automated coverage over its payment code paths. |
| **Multi-tenant / multi-club data isolation not structurally enforced** | `golfler_asp_2` (no base-query `CourseId` filter across 1,393 EF context instantiations, scored 27/27), `sgs-cts-angular` (40+ tenants resolved via a hardcoded table, not a tenant-resolution service), `golfler_pos_2` (~35 environment blocks, one club per configured environment), `cc_api_manager` (11 environment configs, one shared API key across all of them) | Tenant/club isolation in this estate is achieved almost entirely by **deployment-time configuration** (which environment file gets loaded), not by any enforced runtime boundary. A bug in a shared query, or a misconfigured environment block, has cross-club blast radius by construction, not merely by accident. |

## 6. Open questions surfaced by this synthesis (not resolved by any single repo's scan)

- Is `CourseWebApi`'s deprecation (ADR-004, targeted Oct 2025) actually
  complete — i.e. has traffic already moved to PosApi — or is removal simply
  pending? No consumer of `CourseWebApi` was found in any of the 4 client
  repos scanned. [golfler_asp_2 architecture.md Q1]
- What consumes the Azure OpenAI / Azure Search / gRPC dependencies present
  in `golfler_asp_2`'s PosApi project? Not traced to a calling module or a
  downstream consumer anywhere in this 5-repo scan. [golfler_asp_2
  dependencies.md Q1]
- Are all API endpoints in the catalog above (particularly the three
  coexisting tee-booking generations in GolferWebAPI, and the 40/139-endpoint
  surfaces catalogued from `cc_api_manager`/`cc_membership_portal`) still
  actively called by the cataloged-but-unscanned mobile clients (`cc_ios`,
  `cc_android`, `cc_mobile_pos_flex`/`fnb`)? None of those 3 repos were part
  of this scan pass, so their actual call patterns against PosApi/
  GolferWebAPI are unverified.
- Does `cc_membership_portal`'s `API_URL_Invoice` constant (referenced in
  `Member_Model.php` but undefined in the scanned `constants.php`) point at
  a distinct, unscanned backend/environment, or is this a latent
  configuration bug? Needs confirmation against deployment-time config.
- Is a central/shared CI or QA gate enforced *outside* the 5 scanned repos
  for the 4 repos with no in-repo CI/CD evidence? This synthesis can only
  say no such gate was found *in-repo* — it cannot confirm or deny an
  external one.
