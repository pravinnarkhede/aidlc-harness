<<<<<<< HEAD
# Repo Catalog — Golfler / ClubCaddie

Extracted from `looper-code/artifacts/project-structure.md`. Confirm/adjust
roles with the team before treating this as final — see that file for full
per-repo folder structure and additional notes (e.g. the Windows NTFS
checkout caveat on `cc_ios`).

```json
{
  "org": "definelabs",
  "repos": [
    {
      "name": "golfler_asp_2",
      "url": "git@bitbucket.org:definelabs/golfler_asp_2.git",
      "branch": "release_5_7",
      "tags": ["backend", "asp.net", "posapi", "golferwebapi"],
      "role": "Backend — all APIs (PosApi, GolferWebAPI), database, and shared business logic. Owns the API contract every other repo consumes."
    },
    {
      "name": "golfler_pos_2",
      "url": "git@bitbucket.org:definelabs/golfler_pos_2.git",
      "branch": "release_5_7",
      "tags": ["frontend", "desktop", "wpf", "posapi", "staff"],
      "role": "Staff desktop POS terminal (WPF). Consumes PosApi from golfler_asp_2."
    },
    {
      "name": "sgs-cts-angular",
      "url": "git@bitbucket.org:definelabs/sgs-cts-angular.git",
      "branch": "release_v1_7",
      "tags": ["frontend", "web", "angular", "posapi", "staff"],
      "role": "CCOnline — web-based staff management (Angular). Consumes PosApi from golfler_asp_2."
    },
    // {
    //   "name": "cc_mobile_pos_flex",
    //   "url": "git@bitbucket.org:definelabs/cc_mobile_pos.git",
    //   "branch": "release",
    //   "tags": ["mobile", "react-native", "posapi", "staff"],
    //   "role": "Flex App — general-purpose mobile POS for staff (React Native). Consumes PosApi from golfler_asp_2. Same repo as cc_mobile_pos_fnb, different branch."
    // },
    // {
    //   "name": "cc_mobile_pos_fnb",
    //   "url": "git@bitbucket.org:definelabs/cc_mobile_pos.git",
    //   "branch": "fnb_step215",
    //   "tags": ["mobile", "react-native", "posapi", "staff", "f&b"],
    //   "role": "FnB App — food & beverage mobile POS (React Native). Consumes PosApi from golfler_asp_2. Same repo as cc_mobile_pos_flex, different branch."
    // },
    {
      "name": "cc_api_manager",
      "url": "git@bitbucket.org:definelabs/cc_api_manager.git",
      "branch": "release",
      "tags": ["frontend", "web", "php", "golferwebapi", "customer"],
      "role": "iFrames, API Manager, and standalone booking widgets embedded on club websites (PHP CodeIgniter). Consumes GolferWebAPI from golfler_asp_2."
    },
    {
      "name": "cc_membership_portal",
      "url": "git@bitbucket.org:definelabs/cc_membership_portal.git",
      "branch": "release",
      "tags": ["frontend", "web", "php", "golferwebapi", "customer"],
      "role": "Customer Portal — online member self-service (PHP CodeIgniter). Consumes GolferWebAPI from golfler_asp_2."
    }
    // {
    //   "name": "cc_ios",
    //   "url": "git@bitbucket.org:definelabs/cc_ios.git",
    //   "branch": "release_4.1.3_NewSco",
    //   "tags": ["mobile", "ios", "swift", "golferwebapi", "customer"],
    //   "role": "iOS customer mobile app (Swift). Consumes GolferWebAPI from golfler_asp_2. Note: has a Windows NTFS checkout caveat — see project-structure.md."
    // },
    // {
    //   "name": "cc_android",
    //   "url": "git@bitbucket.org:definelabs/cc_android.git",
    //   "branch": "sco_21_may",
    //   "tags": ["mobile", "android", "kotlin", "golferwebapi", "customer"],
    //   "role": "Android customer mobile app (Kotlin). Consumes GolferWebAPI from golfler_asp_2."
    // }
  ]
}
```
=======
# Repo Catalog — PLACEHOLDER, not yet filled in

This is a template, not real data. `auxiliary-project-onboarding` treats
this file as "not yet configured" until you either fill it in yourself
before running your first ticket, or answer its questions interactively (in
which case it overwrites this placeholder with your real answers).

## How to fill this in yourself (optional — skip if you'd rather answer
## onboarding's questions interactively instead)

Replace the JSON block below with your project's real repos. Shape:
`{org, repos: [{name, url, branch, tags, role}]}`.

- `name` — the repo's folder name once cloned (must match its actual repo name)
- `url` — the clone URL (SSH or HTTPS)
- `branch` — the branch to clone/track (its release or default branch)
- `tags` — short free-text labels used to match a ticket's labels/components
  against this repo (e.g. `["backend", "api"]`)
- `role` — **in your own words**: what this repo is and what it owns vs.
  consumes from another repo in the set. This is the part onboarding
  combines with what reverse-engineering actually finds in the code — not
  guessed from the repo name alone.

```json
{
  "org": "your-git-org",
  "repos": [
    {
      "name": "backend-api",
      "url": "git@github.com:your-org/backend-api.git",
      "branch": "main",
      "tags": ["backend", "api"],
      "role": "Backend API and database — owns the contract every other repo consumes"
    },
    {
      "name": "web-app",
      "url": "git@github.com:your-org/web-app.git",
      "branch": "main",
      "tags": ["frontend", "web"],
      "role": "Customer-facing web app — consumes backend-api"
    }
  ]
}
```

Add one entry per repo in your project — there's no fixed count, list
however many repos actually make up your project.
>>>>>>> 72fb5da64bb9e16192d35b9125f8c0a303d2a5fd
