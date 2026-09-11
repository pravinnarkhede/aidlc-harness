---
slug: workspace-ticket-intake
name: Jira Ticket Intake
plugin: workspace
phase: initialization
execution: CONDITIONAL
condition: >
  Execute when the intent's originating request text (the WORKFLOW_STARTED
  audit entry / the "Request:" line recorded at intent creation) contains a
  ticket key matching [A-Z]{2,10}-\d+ (e.g. GOLF-123). If no ticket key is
  found, call `aidlc engine orchestrate report --stage workspace-ticket-intake
  --result skipped` and stop.
lead_agent: workspace-ticket-intake-agent
support_agents: []
mode: inline
produces:
  - workspace-ticket-context
consumes: []
requires_stage:
  - workspace-detection
  - workspace-project-onboarding
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
inputs: The intent's original request text (WORKFLOW_STARTED audit entry / aidlc-state.md header)
outputs: "workspace-ticket-context.md (parsed ticket fields); <ticket-root>/repos.json; aidlc/spaces/<active-space>/knowledge/repo-catalog.md (created on first run if absent); cloned/refreshed repos under the active ticket root; codekb seeded from the shared stable-codebase reference analysis, if present"
---

# Jira Ticket Intake

Runs once per intent, immediately after the three core bootstrap stages
(`state-init`, `workspace-scaffold`, `workspace-detection`) have finished, and
before any Ideation/Inception stage (`intent-capture`, `reverse-engineering`,
...) sees the workspace. Because this stage declares `phase: initialization`
and `requires_stage: [workspace-detection]`, the compiled stage graph places
it first in the DAG without editing any core stage's own `requires_stage`
list.

## Purpose

Give AIDLC the same "ticket in, workspace ready" front door the team's looper
harness already has (`looper-code/commands/looper-plan.md` steps 0.1–0.4,
`looper-code/commands/looper-artifacts-setup.md`), driven entirely through the
normal `/aidlc <request>` entrypoint — no separate command.

## Precondition — one ticket, one session, pointed at its own folder

AIDLC resolves every path for the whole session (intent state, cloned repos,
worktrees — see `resolveProjectDir()` in `.claude/tools/aidlc-lib.ts`) from
`AIDLC_PROJECT_DIR` (or `CLAUDE_PROJECT_DIR`) if set, falling back to the
harness's own install directory otherwise. That resolution happens the moment
the *very first* `/aidlc` command runs — before any stage, including this
one, ever executes. So per-ticket isolation (each ticket's repos **and**
AIDLC state living under its own `aidlc-harness/<TICKET-KEY>/` folder) has to
be set up **before** starting the session, not by a stage mid-workflow:

1. Create the ticket folder once: `mkdir aidlc-harness/GOLF-123`.
2. Start (or point) the Claude Code session at that folder with
   `AIDLC_PROJECT_DIR` (and ideally `CLAUDE_PROJECT_DIR`) set to its absolute
   path, e.g. `AIDLC_PROJECT_DIR=/…/aidlc-harness/GOLF-123 claude` (the
   `.claude/` harness install itself stays physically at `aidlc-harness/.claude`
   — `harnessDir()` resolves independently of `AIDLC_PROJECT_DIR`, so the
   skills/agents/hooks still load normally).
3. Run `/aidlc GOLF-123 <description>` as usual inside that session.

Every subsequent step in this stage operates on **whatever `resolveProjectDir()`
currently returns for this session** — it never hardcodes a path — so as long
as the precondition above was followed, that already IS `aidlc-harness/GOLF-123/`,
and everything (intent state, repos this stage clones, later worktrees) lands
there consistently with zero extra plumbing in this stage.

## Steps

### Step 1: Detect the ticket key and verify the session is pointed at it

1. Read the intent's original request text (the `WORKFLOW_STARTED` audit
   entry, or the header of `<record>/aidlc-state.md` if the audit line has
   rolled off).
2. Match against `[A-Z]{2,10}-\d+`. If no match: run
   `aidlc engine orchestrate report --stage workspace-ticket-intake --result skipped`
   and stop — the rest of this stage does not run.
3. If matched, capture the ticket key (e.g. `GOLF-123`) for the remaining
   steps.
4. **Safety check** — compare the ticket key against the basename of the
   session's currently-resolved project directory (`aidlc engine config flags
   --show`, or any tool output that echoes the resolved project root). If the
   basename does not contain the ticket key (e.g. the session is still
   pointed at the harness root, or at a *different* ticket's folder — most
   likely cause: the precondition above wasn't followed, or this is a reused
   session left over from a previous ticket), **stop immediately** and tell
   the user: the session isn't pointed at this ticket's folder, so continuing
   would clone repos and create state in the wrong place, potentially mixing
   this ticket's work into another ticket's (or the harness root's) tree. Ask
   them to start a new session per the precondition before retrying — do not
   proceed, and do not attempt to fix the path yourself.

### Step 2: Fetch the ticket

Call the Atlassian MCP tool `getJiraIssue` for the ticket key. Extract:
summary, description, acceptance criteria, labels, components, priority,
reporter, assignee. (No REST/env-var fallback is needed here — unlike
looper's `jira-repit-updater`, this harness's Claude Code session already has
the Atlassian MCP connector configured.)

If the fetch fails (ticket not found, connector unavailable), report the
failure to the user, do **not** guess ticket content, and stop the stage —
do not fall through to steps 3+ with fabricated data.

### Step 3: Load the repo catalog

Read `aidlc/spaces/<active-space>/knowledge/repo-catalog.md` — `{org, repos:
[{name, url, branch, tags, role}]}`-shaped, where `role` is the user's own
description of that repo's purpose/scope (recorded during onboarding — see
`workspace-project-onboarding` Step 2). This file is **user-owned**, never
touched by `aidlc update` or by uninstalling this plugin. `requires_stage`
above guarantees `workspace-project-onboarding` has already run before
this step, so on a normal run the catalog already exists (onboarding either
found it or created it on the very first ticket). If it's somehow still
absent — the onboarding stage was skipped some other way, or the catalog was
deleted since — create it the same way onboarding would: ask the user for
their repo list rather than assuming any specific project's repos; do not
invent names.

### Step 4: Refresh the stable codebase mirror (every run — not just on request)

The mirror is **shared across every ticket**, not per-ticket — it's read-only
reference, so duplicating it per ticket folder would just waste disk and
network. Keep it at a fixed location relative to the harness install itself
(independent of the per-session `AIDLC_PROJECT_DIR` redirect): the parent of
`.claude/` — i.e. `aidlc-harness/stable-codebase/<repo>/`, sibling to the
`aidlc-harness/<TICKET-KEY>/` folders, never inside one of them.

For every repo in the catalog, maintain that read-only mirror:

- If the mirror directory does not exist: shallow clone it —
  `git clone --depth 1 --branch <release-branch> <clone-url> <mirror-path>`.
- If it already exists: refresh it to the tip of its release branch —
  `git -C <mirror-path> fetch --depth 1 origin <release-branch>` then
  `git -C <mirror-path> checkout <release-branch>` then
  `git -C <mirror-path> reset --hard origin/<release-branch>`.

This is the same clone-or-refresh routine already proven in
`looper-code/commands/looper-artifacts-setup.md` Step 4 — the only change is
that it now runs automatically on every ticket intake instead of requiring a
manual `/looper-setup` re-run. Never write to, branch from, or commit into
this mirror — it is read-only reference for downstream stages
(`reverse-engineering`, `requirements-analysis`) to consult repos the ticket
does not touch.

### Step 5: Classify impacted repos

For each cataloged repo, pre-suggest a classification by matching the
ticket's labels/components against the catalog's `role`/`tags` fields — the
user-supplied role from onboarding is the primary signal (e.g. a ticket
about a customer-facing screen points at the repo whose `role` says
"customer-facing web app," not at an internal admin tool), not a guess from
the repo name alone:

- **Change** — the repo is what this ticket modifies.
- **Reference** — read-only context; resolved from the stable mirror, never
  cloned per-intent.
- **Out-of-scope** — ignore entirely.

Present the table to the user for confirmation/adjustment before proceeding
— do not silently commit to a classification the user hasn't seen, mirroring
the gate in `looper-code/commands/looper-plan.md` step 2a.

### Step 6: Seed this ticket's reverse-engineering cache from the shared reference analysis

AIDLC's core `reverse-engineering` stage already supports incremental reuse —
its own condition text says a rerun's "Step 1 guard checks store freshness
(codekb-scope-diff) — verified-CURRENT stores may be reused... anything else
rescans." The problem this step solves: each ticket folder is its own AIDLC
project root, so each one starts with an *empty* `codekb/` and would rescan
every repo from zero — no ticket ever benefits from another ticket's prior
analysis.

The fix is to seed, not to re-analyze — this stage never runs
reverse-engineering logic itself, it only copies an existing cache so core's
own freshness check has something to compare against:

1. Check whether `aidlc-harness/stable-codebase/aidlc/spaces/default/codekb/<repo>/`
   exists for each repo classified Change or Reference in Step 5. This is a
   shared, harness-level cache the user refreshes manually and independently
   (see this plugin's README — "Refreshing the shared reference analysis") by
   running the existing `aidlc-reverse-engineering` skill against the
   stable-codebase mirror itself; this stage never triggers that refresh
   automatically, so a normal ticket intake stays fast regardless of how
   stale or fresh that cache currently is.
2. For each such repo that has a shared cache: copy
   `aidlc-harness/stable-codebase/aidlc/spaces/default/codekb/<repo>/` into
   this ticket's own `aidlc/spaces/<active-space>/codekb/<repo>/` — but
   **only if the ticket's own `codekb/<repo>/` doesn't already exist**. Never
   overwrite a codekb this ticket's own reverse-engineering stage has already
   started or finished writing.
3. If no shared cache exists yet for a repo (first time anyone has run the
   reference analysis), skip silently — the ticket's own `reverse-engineering`
   stage will just do its normal full scan, exactly as it does today without
   this plugin.

This means the *first* ticket to touch a given repo still pays for a full
scan (via the shared reference analysis, whenever the user chooses to run
it), but every ticket after that starts from a warm, same-or-recent-vintage
cache instead of zero — and core's own `codekb-scope-diff` freshness check
still runs normally on top of the seeded copy, so drift between the seeded
snapshot and the ticket's actual checked-out code is still caught and
rescanned incrementally, never silently trusted.

### Step 7: Write the repos manifest at the ticket root

Write `repos.json` at the session's current project root (which the
precondition above already established as `aidlc-harness/<TICKET-KEY>/`) with
exactly the confirmed "Change" repos:

```json
{
  "org": "<org from the catalog>",
  "repos": [
    { "name": "<repo>", "branch": "<release-branch-or-user-chosen-branch>", "url": "<clone-url>" }
  ]
}
```

This is the exact schema already validated by the existing core tool
`aidlc-workspace-sync.ts` — nothing new to invent here.

### Step 8: Clone the impacted repos

Run `bun .claude/tools/aidlc-workspace-sync.ts --project-dir <resolved-ticket-root>`
(the harness's own tool, invoked with `.claude/` still resolved against its
real physical install path — only `--project-dir` points at the ticket
folder). This reuses the harness's own existing clone/reconcile engine —
staged clone, branch checkout verification, `.gitignore` gate-block
management, `aidlc.code-workspace` generation, and full rollback safety —
instead of re-implementing looper's PowerShell/bash clone loop a second time.
Handle its exit codes:
- `0` — fully synced, continue.
- `1` — blocked or errored; show the tool's diagnostic to the user and stop
  this stage.
- `2` — synced with branch warnings; show the warnings, continue.

### Step 9: Produce the ticket-context artifact

Write `workspace-ticket-context.md` (this stage's declared `produces` artifact)
under the intent's Initialization record directory, containing the parsed
ticket fields from Step 2 (summary, description, acceptance criteria, labels,
components, priority) so that `intent-capture`, `reverse-engineering`,
`requirements-analysis`, and `delivery-planning` can cite the real ticket
content instead of only the one-line request text the intent was created
with.

### Step 10: Report completion

Run `aidlc engine orchestrate report --stage workspace-ticket-intake --result completed`.
