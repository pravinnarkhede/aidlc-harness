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
