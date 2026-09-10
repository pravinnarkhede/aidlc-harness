# PLANT-001 — Multi-Repository Feature Workspace

## 1. Purpose

This directory is the root workspace for implementing a single feature across multiple applications and repositories.

The feature must be treated as **one end-to-end business feature**, not as five independent development tasks.

The root workspace is the orchestration layer.

The repositories below remain independent Git repositories and must retain their own source control, project structure, build configuration, and AIDLC configuration.

---

# 2. Repository Structure

The expected workspace structure is:

```text
PLANT-001/
│
├── CLAUDE.md
│
├── feature-plan/
│   ├── requirements.md
│   ├── api-contract.md
│   ├── architecture.md
│   ├── implementation-plan.md
│   └── test-plan.md
│
├── golfler_asp_2/
├── sgs-cts-angular/
├── desktop-wpf/
├── php/
└── mobile/
```

Repository responsibilities:

| Repository | Responsibility |
|---|---|
| `golfler_asp_2` | Backend / APIs / business logic / database integration |
| `sgs-cts-angular` | Angular web application |
| `desktop-wpf` | Desktop WPF application |
| `php` | PHP application / integration |
| `mobile` | Mobile application |

If the actual repository names differ, inspect the directories and Git remotes before making assumptions.

---

# 3. Core Rule — One Feature, One Plan

The feature must have one overall plan at the root level.

The root-level feature plan is the source of truth for cross-repository implementation.

Before modifying code, understand:

1. Business requirement
2. Existing behavior
3. Affected applications
4. Backend/API changes
5. Database impact
6. Client impact
7. Integration impact
8. Testing requirements
9. Rollout considerations

Do not independently invent requirements for individual repositories.

If a repository requires a change that is not covered by the feature requirements, first determine whether that change is actually necessary.

---

# 4. Development Order

Unless there is a strong technical reason otherwise, implement the feature in this order:

```text
Requirements
    ↓
Architecture / API Contract
    ↓
Backend
    ↓
Backend Testing
    ↓
Angular
    ↓
WPF
    ↓
PHP
    ↓
Mobile
    ↓
End-to-End Integration Testing
```

The backend API contract is the source of truth for all API-consuming applications.

Do not implement client-side assumptions about APIs before confirming the backend contract.

---

# 5. Repository Isolation

Each child repository is an independent Git repository.

IMPORTANT:

- Do not treat all repositories as one Git repository.
- Do not move files between repositories unless explicitly required.
- Do not modify files in another repository merely because they are visible from the root workspace.
- Do not commit changes from one repository into another repository.
- Always check the current repository before editing files.
- Keep commits logically separated by repository.
- Never modify unrelated repositories.

Before making changes inside a repository, inspect:

```bash
git status
git branch --show-current
git remote -v
```

Confirm that the changes belong to the current feature.

---

# 6. Feature Branches

Where appropriate, use a feature branch with the common feature identifier:

```text
feature/PLANT-001
```

Each repository may have its own branch:

```text
golfler_asp_2
    feature/PLANT-001

sgs-cts-angular
    feature/PLANT-001

desktop-wpf
    feature/PLANT-001

php
    feature/PLANT-001

mobile
    feature/PLANT-001
```

Do not create or switch branches automatically if doing so could discard or interfere with existing developer work.

Always inspect the current Git state first.

---

# 7. Existing Code First

Before implementing anything:

1. Inspect the existing implementation.
2. Find related functionality.
3. Find existing APIs.
4. Find existing models/entities.
5. Find existing database access.
6. Find existing client API calls.
7. Find existing UI flows.
8. Find existing tests.
9. Follow established project conventions.

Prefer extending existing functionality over creating duplicate functionality.

Do not create a new abstraction merely because it appears cleaner.

Do not refactor unrelated code.

---

# 8. CRITICAL BACKEND MIGRATION RULE

The backend contains legacy code that is being migrated from .NET Framework to .NET / .NET 10.

For migrated legacy functionality:

## Preserve legacy behavior.

The default migration strategy is:

```text
Legacy implementation
        ↓
Copy / reproduce existing implementation
        ↓
.NET compatibility changes only
        ↓
.NET implementation
```

Do NOT:

- redesign business logic
- optimize business logic
- refactor business logic
- simplify business logic
- rename existing methods unnecessarily
- change response structures unnecessarily
- change API routes unnecessarily
- change validation behavior
- change database behavior
- change calculation logic
- introduce a new architecture unnecessarily
- replace existing implementation with a cleaner implementation
- "improve" legacy code during migration

The goal is functional equivalence with the legacy implementation.

Only make changes required for framework/runtime compatibility or explicitly requested feature behavior.

---

# 9. Compatibility Changes

Whenever legacy code requires a compatibility change for .NET / .NET 10:

1. Keep the change as small as possible.
2. Preserve the original behavior.
3. Do not use compatibility as an excuse to refactor.
4. Document the change.

Compatibility changes must be recorded in:

```text
aidlc/compatibility-changes.md
```

Each entry should contain:

```text
## <Date> — <Component>

Legacy:
<original behavior/code pattern>

Compatibility change:
<what was changed>

Reason:
<why the change was required for .NET compatibility>

Behavior impact:
<expected behavior impact>

Verification:
<how legacy/core equivalence was verified>
```

---

# 10. Legacy vs Core API Behavior

For migrated APIs, compare legacy and Core behavior whenever possible.

Compare:

- HTTP status code
- response structure
- property names
- property values
- null behavior
- validation behavior
- error behavior
- database side effects
- authorization behavior
- calculations
- collections
- nested objects

JSON property ordering must not be treated as a functional difference.

A difference in actual values or structure must be investigated.

Example:

```text
Legacy:
Course_CourseUsers = 200

Core:
Course_CourseUsers = 415
```

This is a functional difference and must not be ignored.

---

# 11. API Contract Rules

The backend owns the API contract.

Before changing an API consumed by another repository:

1. Identify existing consumers.
2. Understand the existing request.
3. Understand the existing response.
4. Determine whether backward compatibility is required.
5. Update `feature-plan/api-contract.md`.
6. Implement backend changes.
7. Verify backend behavior.
8. Only then update consumers.

Do not silently change an API contract.

If a breaking change is unavoidable, explicitly document:

```text
Old contract
New contract
Reason
Affected consumers
Migration strategy
Backward compatibility strategy
```

---

# 12. Database Rules

Before changing database-related code:

1. Identify existing tables.
2. Identify existing relationships.
3. Identify existing stored procedures/functions/views where applicable.
4. Inspect existing Entity Framework models/configuration.
5. Understand existing transaction behavior.
6. Determine whether the feature actually requires a schema change.

Do not modify the database schema simply because a new table/model would be architecturally cleaner.

Preserve existing database behavior unless the feature explicitly requires a change.

---

# 13. Client Application Rules

The Angular, WPF, PHP, and Mobile applications are API consumers.

Do not duplicate backend business logic in clients when the backend already owns that behavior.

Client applications should:

- call the agreed API contract
- handle the documented response
- preserve existing UI behavior where possible
- follow existing application architecture
- avoid unrelated refactoring

If different clients have different existing behavior, document the differences rather than silently normalizing them.

---

# 14. Scope Control

Only modify repositories that are actually affected by the feature.

For example:

```text
Feature
 ├── Backend       → required
 ├── Angular       → required
 ├── WPF           → not affected
 ├── PHP           → required
 └── Mobile        → required
```

Do not modify WPF merely because it exists in the workspace.

At the beginning of implementation, create or update:

```text
feature-plan/implementation-plan.md
```

with an affected-repository matrix.

Example:

```text
| Repository | Affected | Reason |
|------------|----------|--------|
| Backend    | Yes      | New API |
| Angular    | Yes      | UI consumes API |
| WPF        | No       | Existing flow unaffected |
| PHP        | Yes      | Integration uses API |
| Mobile     | Yes      | Mobile flow consumes API |
```

---

# 15. Root Feature Plan

Maintain these documents:

```text
feature-plan/
├── requirements.md
├── api-contract.md
├── architecture.md
├── implementation-plan.md
└── test-plan.md
```

## requirements.md

Contains:

- business requirement
- user/business problem
- expected behavior
- acceptance criteria
- assumptions
- constraints
- out-of-scope items

## api-contract.md

Contains:

- endpoints
- HTTP methods
- request models
- response models
- authentication/authorization
- validation
- errors
- backward compatibility requirements

## architecture.md

Contains:

- affected systems
- data flow
- application flow
- backend flow
- client flow
- database impact
- integration points

## implementation-plan.md

Contains:

- affected repositories
- implementation order
- files/components expected to change
- dependencies
- migration considerations
- deployment considerations

## test-plan.md

Contains:

- unit tests
- integration tests
- API tests
- legacy/Core comparison tests where applicable
- UI tests
- end-to-end scenarios
- regression scenarios
- acceptance criteria verification

---

# 16. Planning Before Coding

Do not immediately start modifying source code when a feature request is first provided.

First:

```text
Understand
    ↓
Inspect
    ↓
Ask only necessary questions
    ↓
Plan
    ↓
Confirm affected repositories
    ↓
Define API contract
    ↓
Implement
```

If information is missing but a safe assumption can be made, clearly document the assumption.

If a missing decision could materially change the implementation, stop that specific implementation step and identify the decision required.

Do not invent business requirements.

---

# 17. AIDLC Usage

Each repository has its own AIDLC project configuration.

Examples:

```text
golfler_asp_2/.claude/
sgs-cts-angular/.claude/
desktop-wpf/.claude/
php/.claude/
mobile/.claude/
```

Use the AIDLC workflow appropriate to the repository being implemented.

The root workspace is the cross-repository orchestration layer.

Do not assume that configuring AIDLC in one repository automatically configures the other repositories.

For backend work, the existing backend-specific AIDLC skills may be available, for example:

```text
/golfler_asp_2:aidlc-feasibility
/golfler_asp_2:aidlc-feature
```

Use them when the work is being performed inside `golfler_asp_2`.

---

# 18. Feasibility First for Complex Features

For a complex feature, perform feasibility/ideation before implementation.

Evaluate:

- existing implementation
- affected repositories
- API dependencies
- database dependencies
- external integrations
- technical constraints
- backward compatibility
- migration risks
- testing complexity
- deployment risks

Do not use feasibility as an opportunity to redesign the system.

The purpose is to determine whether the requested feature can be implemented safely within the existing architecture.

---

# 19. Testing Requirements

Testing is part of implementation, not an optional final step.

For every affected repository:

```text
Build
 ↓
Unit tests where applicable
 ↓
Integration/API tests where applicable
 ↓
Feature verification
 ↓
Regression verification
```

For backend APIs, prefer request/response verification against known legacy behavior when migrating existing functionality.

For cross-repository features, verify the complete flow:

```text
Client
  ↓
API
  ↓
Business Logic
  ↓
Database
  ↓
API Response
  ↓
Client
```

Record important test results in:

```text
feature-plan/test-plan.md
```

---

# 20. Regression Protection

Existing functionality must not be broken by the new feature.

Pay particular attention to:

- high-usage APIs
- frequently used client flows
- authentication
- authorization
- payments
- orders
- tee times
- activities
- events
- F&B
- existing integrations

If a change affects an existing API, identify the regression scenarios before implementation.

---

# 21. Do Not Hide Problems

Never hide or suppress:

- failing tests
- build errors
- API differences
- database errors
- authorization failures
- null/reference errors
- unexpected response differences
- migration compatibility problems

If a problem is discovered:

```text
Problem
↓
Root cause
↓
Impact
↓
Recommended fix
↓
Verification
```

Document significant issues in the appropriate feature-plan document.

---

# 22. No Unnecessary Refactoring

This is especially important for the legacy backend.

Do not turn feature implementation into a cleanup project.

Avoid:

```text
"While we are here..."
```

changes such as:

- renaming unrelated classes
- changing architecture
- extracting unrelated services
- replacing working patterns
- changing dependency injection patterns without need
- changing database queries without requirement
- formatting entire files unnecessarily
- changing unrelated API responses

Only make changes required for:

1. The feature
2. Framework compatibility
3. Required security fixes
4. Required build/runtime fixes
5. Explicitly requested improvements

---

# 23. Preserve Existing Conventions

Before creating new code, inspect nearby code.

Follow the existing:

- naming conventions
- folder structure
- dependency injection pattern
- logging pattern
- exception handling
- API response pattern
- authorization pattern
- database access pattern
- testing pattern
- UI conventions

Do not introduce a new pattern unless the existing pattern cannot support the feature.

---

# 24. Security

Do not expose:

- passwords
- API keys
- tokens
- connection strings
- secrets
- credentials
- production secrets

Do not commit secrets to Git.

If configuration is required, use the project's existing configuration/secret mechanism.

Do not weaken authentication or authorization to make tests pass.

---

# 25. Git Safety

Before modifying a repository:

```bash
git status
```

If there are existing uncommitted changes:

- do not overwrite them
- do not reset them
- do not clean them
- do not revert them
- determine whether they are related to the current feature

Never run destructive Git commands unless explicitly requested.

Avoid:

```bash
git reset --hard
git clean -fd
git checkout -- .
```

unless explicitly authorized.

---

# 26. Change Tracking

At the end of implementation, provide a clear summary:

```text
Feature:
PLANT-001

Repositories changed:
- golfler_asp_2
- sgs-cts-angular
- mobile

Repositories not changed:
- desktop-wpf
- php

Backend:
- APIs changed
- business logic changed
- database changes

Clients:
- Angular changes
- Mobile changes

Tests:
- Tests executed
- Results

Legacy/Core compatibility:
- Verified / Not verified
- Differences found

Known issues:
- ...

Deployment considerations:
- ...
```

---

# 27. Completion Criteria

A feature is not considered complete merely because the code compiles.

The feature is complete when:

- requirements are satisfied
- acceptance criteria are satisfied
- API contract is verified
- affected repositories are implemented
- builds succeed
- applicable tests pass
- integration flow is verified
- regression risks are reviewed
- legacy/Core behavior is verified where applicable
- compatibility changes are documented
- feature-plan documents are updated
- no unrelated changes are introduced

---

# 28. Working Principle

Always think in terms of:

```text
BUSINESS FEATURE
       ↓
CROSS-REPOSITORY IMPACT
       ↓
SINGLE FEATURE PLAN
       ↓
API CONTRACT
       ↓
BACKEND
       ↓
CLIENTS
       ↓
INTEGRATION
       ↓
REGRESSION TESTING
```

The goal is not merely to modify five codebases.

The goal is to deliver **one working business feature consistently across all affected applications** while preserving existing behavior and minimizing unnecessary changes.

---

# 29. First Action for a New Feature

When the user gives a new feature request, do the following:

### Step 1 — Understand the request

Identify:

- business goal
- user flow
- expected result
- acceptance criteria
- constraints

### Step 2 — Inspect repositories

Determine which repositories contain relevant functionality.

### Step 3 — Inspect existing implementation

Search before creating new code.

### Step 4 — Create/update the feature plan

Update:

```text
feature-plan/requirements.md
feature-plan/api-contract.md
feature-plan/architecture.md
feature-plan/implementation-plan.md
feature-plan/test-plan.md
```

### Step 5 — Identify dependencies

Determine:

```text
Database
API
Backend
Angular
WPF
PHP
Mobile
External integrations
Testing
```

### Step 6 — Implement in dependency order

Normally:

```text
Backend
↓
API verification
↓
Clients
↓
Integration testing
```

### Step 7 — Verify

Run appropriate builds and tests.

### Step 8 — Report

Clearly report:

- what changed
- where it changed
- what was tested
- what remains
- known issues
- compatibility changes

---

# 30. Final Rule

When uncertain:

**Preserve existing behavior over introducing a new design.**

**Inspect existing code before writing new code.**

**Plan the complete feature before implementing individual repositories.**

**Keep repositories independent.**

**Keep the backend API contract explicit.**

**Do not modify unrelated code.**

**Do not refactor legacy code unless explicitly required.**

**Never hide failures or behavioral differences.**