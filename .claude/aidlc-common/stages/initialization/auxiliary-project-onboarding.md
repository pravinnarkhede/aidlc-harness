---
slug: auxiliary-project-onboarding
name: Project Onboarding
plugin: auxiliary
phase: initialization
execution: CONDITIONAL
condition: >
  Execute only on the very first ticket ever run against this project — i.e.
  when the shared `stable-codebase/` mirror (sibling to the harness root)
  does not yet exist, or exists but has no onboarding marker
  (`stable-codebase/.auxiliary-onboarded`). Every ticket after the first
  one finds the marker present and self-reports
  `aidlc engine orchestrate report --stage auxiliary-project-onboarding
  --result skipped` immediately — this stage does real work exactly once per
  project, never per ticket.
lead_agent: auxiliary-project-onboarding-agent
support_agents: []
mode: inline
produces:
  - auxiliary-architecture-overview
  - auxiliary-coding-standards
consumes: []
requires_stage:
  - workspace-detection
sensors: []
scopes:
  - bugfix
  - feature
  - mvp
  - express
  - security-patch
  - enterprise
  - refactor
  - classic
inputs: The repo catalog at aidlc/spaces/<active-space>/knowledge/repo-catalog.md (created here if absent)
outputs: "stable-codebase/ populated for every cataloged repo; shared codekb/ built via core reverse-engineering; auxiliary-architecture-overview.md and auxiliary-coding-standards.md under aidlc/spaces/<active-space>/knowledge/; proposed memory/team.md + memory/project.md entries, written only after human confirmation; stable-codebase/.auxiliary-onboarded marker"
---

# Project Onboarding

Runs once per project — the very first time anyone starts a ticket through
this plugin — and never again after that, on any subsequent ticket, in any
ticket folder. This is deliberately **project-generic**: nothing in this
stage or its agent hardcodes a specific codebase, repo list, or domain. It
works the same way whether it's pointed at this Golfler/ClubCaddie platform
or an entirely different project, because everything it discovers comes from
reading the actual repos in the catalog, not from assumptions baked into
this file.

`auxiliary-ticket-intake` declares this stage in its own `requires_stage`,
so on the very first ticket, onboarding always completes before ticket
intake runs — meaning even ticket #1 gets a warm, freshly-built knowledge
base to seed from, not just ticket #2 onward.

## Steps

### Step 1: Detect first-run

Check for `stable-codebase/.auxiliary-onboarded` (sibling to the harness
root, independent of any per-ticket `AIDLC_PROJECT_DIR` redirect — same
location `auxiliary-ticket-intake` uses for the shared mirror). If present,
run `aidlc engine orchestrate report --stage auxiliary-project-onboarding
--result skipped` and stop — nothing below this step runs.

### Step 2: Load or create the repo catalog — with the user's own role/scope guidance

Same catalog `auxiliary-ticket-intake` Step 3 reads:
`aidlc/spaces/<active-space>/knowledge/repo-catalog.md` — `{org, repos:
[{name, url, branch, tags, role}]}`-shaped table. If it doesn't exist yet,
this is the first place it gets created — and for a project that spans
multiple repos, do not try to infer each repo's purpose from cloning and
guessing alone. Ask the user directly, up front, before cloning anything:

- What is the overall project/product this set of repos implements?
- For each repo: what is its **role** in that project (e.g. "backend API and
  database," "customer-facing web app," "mobile client," "internal admin
  tool") and its **scope** (what it owns vs. what it consumes from another
  repo in the set)?

Record the user's answers in the catalog's `role` field per repo (free text,
in the user's own words) — this is authoritative, human-supplied context
that Step 5's synthesis combines with what reverse-engineering actually
finds in the code, rather than relying on code analysis alone to guess
intent. Do not invent or guess repo names, roles, or relationships — if the
user can point at an existing catalog-shaped file in their own tooling (this
project's team happens to have one at
`looper-code/artifacts/project-structure.md`), offer to import it and still
confirm the role/scope of each entry with the user rather than trusting it
silently, since a pre-existing file may be stale or incomplete.

### Step 3: Clone the shared stable-codebase mirror

For every cataloged repo, shallow clone it into
`aidlc-harness/stable-codebase/<repo>/` at its declared branch:
`git clone --depth 1 --branch <branch> <url> <mirror-path>`. This is a
read-only reference mirror, shared across every ticket — never write to,
branch from, or commit into it.

### Step 4: Build the shared reverse-engineering cache

Run AIDLC's own core `reverse-engineering` stage in isolation (the
`aidlc-reverse-engineering` skill, single-stage mode) against the
stable-codebase mirror itself — i.e. with the session's project directory
resolved to `aidlc-harness/stable-codebase/`. This is the exact same
mechanism this plugin's README documents as the *manual* refresh path; here
it runs once, automatically, as part of first-time setup. No
reverse-engineering logic is reimplemented — core's own stage does the real
analysis and writes `aidlc-harness/stable-codebase/aidlc/spaces/default/codekb/<repo>/`
(9 artifacts per repo: business-overview, architecture, code-structure,
api-documentation, component-inventory, technology-stack, dependencies,
code-quality-assessment, timestamp).

### Step 5: Synthesize cross-repo project knowledge

Read the `codekb/<repo>/` artifacts Step 4 just produced, across every
repo, and write two new space-level knowledge documents (sanctioned
user-owned path, never touched by `aidlc update` or plugin uninstall):

- `aidlc/spaces/<active-space>/knowledge/architecture-overview.md` —
  combines two sources, not code analysis alone: the `role`/scope the user
  gave you per repo in Step 2 (authoritative for *intent* — what each repo
  is supposed to be), plus what Step 4's reverse-engineering actually found
  (authoritative for *reality* — what the code actually does, its real API
  surface, its real dependencies). Where they agree, state it plainly. Where
  they disagree or the code reveals something the user's description didn't
  mention (an undocumented consumer, a repo that's actually a shared
  library, a deprecated component nobody flagged), surface that
  explicitly as a discrepancy rather than silently picking one source. Do
  not assert an architecture that neither the user's input nor the codekb
  artifacts actually support.
- `aidlc/spaces/<active-space>/knowledge/coding-standards.md` — conventions
  actually observed per repo (naming, structure, patterns, testing
  approach) pulled from each repo's `code-structure.md` and
  `code-quality-assessment.md`, not generic advice.

### Step 6: Propose (never silently write) memory entries

From the same synthesis, draft candidate entries for
`aidlc/spaces/<active-space>/memory/project.md` (project-specific
specialisation — e.g. "repo X owns the API contract for repos Y, Z"; a
regression-focus-area list, if the business domain is discoverable from the
artifacts) and, separately, genuinely generic engineering discipline that
isn't specific to this codebase belongs in `memory/team.md` instead (e.g.
"never silently change a consumed API contract"), not `project.md` — keep
the same project-vs-generic split this project's own memory files already
follow.

**Present the draft to the user and wait for explicit confirmation/edits
before writing anything.** Never append to `memory/team.md` or
`memory/project.md` unprompted — these are hand-editable, human-owned
files; this stage only ever proposes, the human decides what actually lands.

### Step 7: Mark onboarding complete

Write `stable-codebase/.auxiliary-onboarded` (a single ISO-8601 timestamp
line). Every later ticket's own `auxiliary-project-onboarding` stage
instance finds this immediately at Step 1 and skips, so this stage's real
cost is paid exactly once per project.

### Step 8: Report completion

Run `aidlc engine orchestrate report --stage auxiliary-project-onboarding --result completed`.
