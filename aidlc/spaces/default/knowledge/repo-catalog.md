# Project Structure — Golfler / ClubCaddie

The platform spans **9 repositories**. The backend (`golfler_asp_2`) exposes two APIs consumed by 8 other repos.

---

## Repository Overview

| Repo | Type | API | Release Branch | Role |
|---|---|---|---|---|
| `golfler_asp_2` | ASP.NET solution | — | `release_5_7` | Backend — all APIs, database, business logic |
| `golfler_pos_2` | WPF XAML (C#) | PosApi | `release_5_7` | Staff desktop POS terminal |
| `sgs-cts-angular` | Angular (TypeScript) | PosApi | `teesheet_v2_step763` | CCOnline — web-based staff management |
| `cc_mobile_pos` (Flex) | React Native | PosApi | `release` | Flex App — general mobile POS |
| `cc_mobile_pos` (FnB) | React Native | PosApi | `fnb_step215` | FnB App — food & beverage mobile POS |
| `cc_api_manager` | PHP CodeIgniter | GolferWebAPI | `release` | iFrames, API Manager, standalone booking widgets |
| `cc_membership_portal` | PHP CodeIgniter | GolferWebAPI | `release` | Customer Portal — online member self-service |
| `cc_ios` | Swift | GolferWebAPI | `release_4.1.3_NewSco` | iOS customer mobile app |
| `cc_android` | Kotlin | GolferWebAPI | `sco_21_may` | Android customer mobile app |

---

---

# golfler_asp_2 — Backend

```
git clone git@bitbucket.org:definelabs/golfler_asp_2.git
# checkout the release branch listed in the Repository Overview table above
```

**Type:** ASP.NET MVC 5 / Web API 2, .NET Framework 4.5.2–4.7.2, Entity Framework 6, SQL Server  
**Role:** The main backend. Contains all APIs, database schema, and shared business logic.

## Projects

| Project | Type | Role |
|---|---|---|
| **PosApi** | Web API 2 | Internal staff API — orders, payments, inventory, reconciliation |
| **GolferWebAPI** | Web API 2 | Customer/mobile API — tee time booking, memberships, aggregators |
| **Golfler** | ASP.NET MVC 5 | Admin web portal — course config, reporting, management |
| **GolflerDataModel** | Class Library | Entity Framework 6 models and `GolflerDataModelEntities` DbContext |
| **GolflerShared** | Shared Project | Critical shared business logic — payment, booking, utilities |
| **GolflerDB** | SQL DB Project | Database schema — 431+ tables, stored procedures, views |
| **CourseWebApi** | Web API 2 | Tee sheet API — **DEPRECATED**, consolidating into PosApi |
| **CCU** | ASP.NET Core | Modern utility web interface |
| **AzureUtilities** | Azure Functions | GL uploads, file storage, background jobs |
| **HubSpotIntegration** | Azure Function | HubSpot CRM sync |
| **RangeExpress** | Azure Function | Range management automation |
| **VoucherExpirationWindowsService** | Windows Service | Monitors and expires vouchers |
| **MaintenanceConsoleApp** | Console App | Data migration and maintenance utilities |
| **CCACHWebhook** | Console App | CardConnect ACH webhook receiver |
| **POS** | WinForms | Legacy desktop POS client (minimal) |
| **PosApiUnitTest** | Test Project | **EMPTY — no tests exist** |

## Folder Structure

```
golfler_asp_2/
├── Golfler.sln
│
├── PosApi/                          # Internal staff Web API
│   ├── ActionFilters/               # ApiUserTokenMandatoryAuthorizationFilter
│   ├── App_Start/                   # Route, WebApi, filter config
│   ├── Controllers/
│   │   ├── AiChat/
│   │   ├── ChartOfAccount/
│   │   ├── Customers/
│   │   ├── Departments/
│   │   ├── Events/
│   │   ├── GiftCards/
│   │   ├── Invoices/
│   │   ├── Memberships/
│   │   ├── Orders/
│   │   ├── PaymentProcessing/
│   │   ├── Reports/
│   │   ├── Reservations/
│   │   ├── Settings/
│   │   ├── Tax/
│   │   └── CourseController.cs      # Settings, course config (huge)
│   ├── Data/
│   ├── Models/                      # DTOs and request/response bodies
│   └── PaymentReconciliation/
│
├── GolferWebAPI/                    # Customer/mobile Web API
│   ├── ActionFilters/
│   ├── App_Start/
│   ├── Controllers/
│   │   ├── AggregatorIntegration/   # Forefront and other aggregators
│   │   ├── CCInterface/             # Cross-course integration
│   │   ├── HealthCheck/
│   │   ├── Membership/              # Voucher, membership endpoints
│   │   ├── MembershipSale/
│   │   ├── MobileAppBuilder/
│   │   └── Reservation/             # Tee sheet / booking
│   ├── Models/
│   │   ├── CreditVoucher.cs         # Voucher lookup, sale, search logic
│   │   ├── OnlinePaymentsHandler.cs # Online payment validation + processing
│   │   └── TeeBooking.cs            # Core tee time booking logic
│   └── emailtemplates/
│
├── Golfler/                         # Admin MVC Portal
│   ├── Controllers/
│   ├── Views/
│   │   ├── Admin/
│   │   ├── CourseAdmin/
│   │   ├── Golfer/
│   │   ├── Home/
│   │   ├── Membership/
│   │   ├── MenuItems/
│   │   ├── QuickBooks/
│   │   ├── Shared/
│   │   └── Tools/
│   ├── Content/                     # CSS, images, jQuery plugins
│   └── Scripts/                     # Global JS functions (~638k lines)
│
├── GolflerDataModel/                # Entity Framework models
│   ├── Models/                      # 431+ EF entity classes
│   │   ├── GF_CreditVoucher.cs
│   │   ├── GF_CourseInfo.cs
│   │   ├── GF_Order.cs
│   │   ├── GF_Customer.cs
│   │   ├── GF_Settings.cs
│   │   └── ...
│   ├── Modules/
│   │   └── RedisCache/
│   └── Connected Services/
│       └── FirstPayServiceReference/
│
├── GolflerShared/                   # ⚠️ CRITICAL — changes affect ALL projects
│   └── Modules/
│       ├── Payment.cs               # Payment gateways — EXTREME RISK
│       ├── TeeBooking.cs            # Tee time booking logic
│       ├── CommonProperties.cs      # Setting name constants (WebSetting class)
│       ├── CommonFunctions.cs       # Utility helpers
│       └── PaymentType.cs           # Payment type constants
│
├── GolflerDB/                       # SQL Server Database Project
│   ├── dbo/
│   │   ├── Tables/                  # 431+ table DDL files
│   │   ├── Stored Procedures/
│   │   ├── Functions/
│   │   └── Views/
│   ├── DBScripts/                   # Migration scripts
│   └── HangFire/                    # Background job schema
│
├── CourseWebApi/                    # DEPRECATED tee sheet API
│   ├── Controllers/Teesheet/
│   └── Web.*.config                 # 40+ environment configs
│
├── docs/                            # All documentation
│   ├── CODE_CONVENTIONS.md
│   ├── API_CONVENTIONS.md
│   ├── DB_CONVENTIONS.md
│   ├── MOBILE_CONVENTIONS.md
│   ├── UI_CONVENTIONS.md
│   ├── KNOWN_RISKS_AND_TECH_DEBT.md
│   ├── DATABASE.md                  # Full DB schema reference (88 KB)
│   ├── adr/                         # Architecture Decision Records
│   └── api/                         # OpenAPI specs + markdown references
│       ├── posapi-openapi.json       (1.7 MB)
│       └── golferwebapi-openapi.json (542 KB)
│
├── looper-code/                     # AI agent tooling
│   ├── agents/
│   ├── commands/
│   └── artifacts/                   # This file lives here
│
├── AzureUtilities/                  # Azure Functions (.NET Core)
├── HubSpotIntegration/
├── RangeExpress/
├── VoucherExpirationWindowsService/
├── MaintenanceConsoleApp/
├── CCACHWebhook/
├── CCU/                             # ASP.NET Core utility app
├── PosApiUnitTest/                  # EMPTY — no tests
├── Binaries/                        # Pre-compiled third-party DLLs
├── packages/                        # NuGet packages
└── JenkinsAutomation/               # CI/CD pipeline scripts (Groovy)
```

---

---

# golfler_pos_2 — WPF POS App

```
git clone git@bitbucket.org:definelabs/golfler_pos_2.git
# checkout the release branch listed in the Repository Overview table above
```

**Type:** WPF XAML, C#, .NET Framework  
**Role:** Staff desktop point-of-sale terminal. Consumes **PosApi**.

## Projects

| Project | Role |
|---|---|
| **POSApp** | Main WPF application — UI, windows, views |
| **POSApp.Data** | Data models, API request/response contracts, settings |
| **POSApp.Core** | Shared business logic, utilities |

## Folder Structure

```
golfler_pos_2/
├── POSApp/
│   ├── Views/                       # XAML screens (windows, user controls)
│   ├── ViewModels/                  # MVVM view models
│   ├── Controls/                    # Custom WPF controls
│   └── Resources/                   # Styles, icons, themes
├── POSApp.Data/
│   ├── DataModels/
│   │   └── Settings/
│   │       └── IFrameBanners.cs     # iFrame banner data model
│   └── ApiContracts/                # Request/response models for PosApi calls
└── POSApp.Core/
    └── Services/                    # API client services
```

---

---

# sgs-cts-angular — CCOnline Angular App

```
git clone git@bitbucket.org:definelabs/sgs-cts-angular.git
# checkout the release branch listed in the Repository Overview table above
```

**Type:** Angular (TypeScript)  
**Role:** Web-based staff management interface (CCOnline). Consumes **PosApi**.

## Projects

Single Angular app — no sub-projects.

## Folder Structure

```
sgs-cts-angular/
├── angular.json
├── package.json
├── tsconfig.json
├── proxy.config.json                # Dev server proxy to PosApi
│
└── src/
    └── app/
        ├── app-routing.module.ts    # Main routes (55 KB)
        ├── app.module.ts            # Root module (114 KB)
        │
        ├── service/                 # Core services
        │   ├── authentication.service.ts
        │   ├── iframe-setting.service.ts  # iFrame settings API calls
        │   └── ...
        │
        ├── setting/                 # Settings screens
        │   └── i-frames/            # iFrame settings (toggles, colors, URLs)
        │       ├── i-frames.component.ts
        │       └── i-frames.component.html
        │
        ├── customers/               # Customer management
        ├── dashboard/               # Main dashboard
        ├── register/                # POS register
        ├── sales/                   # Sales transactions
        ├── payment/                 # Payment methods (card, cash, voucher, etc.)
        ├── paymentV1/               # Legacy payment module
        ├── reports/                 # Analytics and reporting
        ├── teesheet/                # Tee sheet management
        ├── vouchers/                # Gift/credit voucher handling
        ├── events/                  # Event management
        ├── memberships/             # Membership management
        ├── kitchen/                 # Kitchen display system
        ├── ondemand/                # On-demand ordering
        ├── hubspot/                 # HubSpot CRM integration
        ├── global-view/             # Global club dashboard
        ├── mco-dashboard/           # Multi-club overview
        ├── login-v2/                # Login screen
        ├── terminal-selection-v2/   # Terminal/location selection
        ├── shared/                  # Shared components and utilities
        ├── common-service/          # Common constants, utilities, date helpers
        └── Custom-Pipes/            # Angular pipes
```

---

---

# cc_mobile_pos — FnB App + Flex App

```
git clone git@bitbucket.org:definelabs/cc_mobile_pos.git
# checkout the Flex App or FnB App release branch listed in the Repository Overview table above
```

**Type:** React Native  
**Role:** Two mobile POS apps in one repo — **FnB App** (food & beverage) and **Flex App** (general mobile POS). Both consume **PosApi**.

## Projects

| App | Role |
|---|---|
| **FnB App** | Food & beverage ordering and payment at the point of service |
| **Flex App** | General-purpose mobile POS for staff |

## Folder Structure

```
cc_mobile_pos/
├── package.json
├── App.js                           # Entry point
├── src/
│   ├── screens/                     # UI screens
│   ├── components/                  # Reusable components
│   ├── services/                    # PosApi client services
│   ├── store/                       # State management
│   └── navigation/                  # React Navigation config
└── android/ + ios/                  # Native platform code
```

*(Exact structure not yet locally mapped — clone repo for details)*

---

---

# cc_api_manager — iFrames / API Manager / Booking Widgets

```
git clone git@bitbucket.org:definelabs/cc_api_manager.git
# checkout the release branch listed in the Repository Overview table above
```

**Type:** PHP CodeIgniter  
**Role:** Powers the online tee time booking iFrames, API Manager, and standalone booking widgets embedded on club websites. Consumes **GolferWebAPI**.

## Projects

Single PHP application — no sub-projects.

## Folder Structure

```
cc_api_manager/
├── application/
│   ├── controllers/             # PHP controllers (booking, payment, member lookup)
│   ├── models/                  # Data models, GolferWebAPI client calls
│   ├── views/                   # HTML/PHP templates (booking UI, payment UI)
│   │   └── webapi/
│   │       └── view/            # Tee time booking iframe template
│   ├── config/                  # App config, routes, database
│   └── libraries/               # Shared libraries
├── assets/                      # CSS, JS, images for the iframe UI
└── index.php
```

*(Exact structure not yet locally mapped — clone repo for details)*

---

---

# cc_membership_portal — Customer Portal

```
git clone git@bitbucket.org:definelabs/cc_membership_portal.git
# checkout the release branch listed in the Repository Overview table above
```

**Type:** PHP CodeIgniter  
**Role:** Online member self-service portal — members can view their profile, book tee times, manage memberships, and make payments. Consumes **GolferWebAPI**.

## Projects

Single PHP application — no sub-projects.

## Folder Structure

```
cc_membership_portal/
├── application/
│   ├── controllers/             # Member portal controllers
│   ├── models/                  # GolferWebAPI client models
│   ├── views/                   # Member-facing HTML templates
│   └── config/
├── assets/                      # CSS, JS, images
└── index.php
```

*(Exact structure not yet locally mapped — clone repo for details)*

---

---

# cc_ios — iOS App

```
git clone git@bitbucket.org:definelabs/cc_ios.git
# checkout the release branch listed in the Repository Overview table above
```

**Type:** Swift, iOS  
**Role:** Customer-facing mobile app for iOS. Book tee times, manage memberships, view account. Consumes **GolferWebAPI**.

> ⚠️ App Store deployments take 1–3 days + weeks for user adoption. Never break GolferWebAPI response contracts.

> 🪟 **Windows clone caveat.** On Windows, `git clone` fetches history fine but the **checkout fails**: `invalid path 'Golfie/Configuration/WildHorse /…'`. The folder `WildHorse ` (and its nested `WildHorse .xcassets`) has a **trailing space**, which NTFS forbids, so git's `core.protectNTFS` guard aborts the working tree.
> - **Permanent fix (preferred):** rename the folder at source on macOS/Linux, mirroring the prior LincolnGolf fix — `git mv "Golfie/Configuration/WildHorse " "Golfie/Configuration/WildHorse"` (rename the nested `.xcassets` too), then commit. After this, plain `git clone` works on Windows.
> - **Local workaround (if you must clone on Windows now):** `git -c core.protectNTFS=false read-tree HEAD`, force-remove the `WildHorse ` entries from the index, then `git -c core.protectNTFS=false checkout-index -a -f`. Those 33 files then show as deleted in `git status` — **do not commit that deletion**; it is local-only.

## Projects

| Target | Role |
|---|---|
| **ClubCaddie** | Main iOS app |

## Folder Structure

```
cc_ios/
├── ClubCaddie/
│   ├── AppDelegate.swift
│   ├── Controllers/             # UIViewControllers / SwiftUI views
│   ├── Models/                  # Data models, GolferWebAPI response models
│   ├── Services/                # API client (URLSession calls to GolferWebAPI)
│   ├── Views/                   # UI components
│   └── Resources/               # Assets, storyboards, localisation
├── ClubCaddie.xcodeproj
└── Podfile                      # CocoaPods dependencies
```

*(Exact structure not yet locally mapped — clone repo for details)*

---

---

# cc_android — Android App

```
git clone git@bitbucket.org:definelabs/cc_android.git
# checkout the release branch listed in the Repository Overview table above
```

**Type:** Kotlin, Android  
**Role:** Customer-facing mobile app for Android. Book tee times, manage memberships, view account. Consumes **GolferWebAPI**.

> ⚠️ Play Store deployments take 1–3 days + weeks for user adoption. Never break GolferWebAPI response contracts.

## Projects

| Module | Role |
|---|---|
| **app** | Main Android application module |

## Folder Structure

```
cc_android/
├── app/
│   ├── src/main/
│   │   ├── java/
│   │   │   └── com/clubcaddie/
│   │   │       ├── activities/      # Android Activities (screens)
│   │   │       ├── fragments/       # Fragments (sub-screens)
│   │   │       ├── models/          # GolferWebAPI response models
│   │   │       ├── network/         # Retrofit API client
│   │   │       └── utils/           # Utilities
│   │   └── res/                     # Layouts, drawables, strings
│   └── build.gradle
├── build.gradle
└── settings.gradle
```

*(Exact structure not yet locally mapped — clone repo for details)*

---

---

## Key Architectural Notes

### API Boundaries

```
                    ┌─────────────────────────────┐
                    │       golfler_asp_2          │
                    │  ┌──────────┐ ┌───────────┐ │
  Staff tools ──────┼─▶│  PosApi  │ │GolferWebAPI│◀┼────── Customer tools
                    │  └──────────┘ └───────────┘ │
                    └─────────────────────────────┘

PosApi consumers:          GolferWebAPI consumers:
  golfler_pos_2              cc_api_manager
  sgs-cts-angular            cc_membership_portal
  cc_mobile_pos (FnB)        cc_ios
  cc_mobile_pos (Flex)       cc_android
```

### Multi-Tenant Rule (Mandatory in golfler_asp_2)
Every database query must include `CourseId`. Omitting it leaks data across clubs.

### Mobile App Constraint
iOS and Android apps cannot update instantly — App Store / Play Store review takes 1–3 days, and user adoption takes weeks. **Never remove or rename a GolferWebAPI response field.**

### Settings Pattern (golfler_asp_2)
Per-club settings are key-value rows in `GF_Settings`. Constants defined in `GolflerShared/Modules/CommonProperties.cs` (`WebSetting` class). Saved via `PosApi → Course/SaveAppSetting`. Read back via `GetMembershipSetting`.
