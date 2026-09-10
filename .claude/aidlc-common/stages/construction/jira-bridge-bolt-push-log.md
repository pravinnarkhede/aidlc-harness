---
slug: jira-bridge-bolt-push-log
name: Bolt Push and Log
plugin: jira-bridge
phase: construction
execution: CONDITIONAL
condition: >
  Execute after code-generation completes for a unit, when that unit's Bolt
  ran in a git worktree (autonomous swarm / per-Bolt worktree mode). Skip
  (report --result skipped) when the unit did not run in an isolated
  worktree, since there is then no dedicated branch to push.
lead_agent: jira-bridge-bolt-push-agent
support_agents: []
mode: inline
for_each: unit-of-work
produces:
  - jira-bridge-implementation-log
consumes:
  - artifact: code-summary
    required: true
  - artifact: unit-of-work
    required: true
requires_stage:
  - code-generation
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
inputs: "worktree info for the unit's Bolt (aidlc engine worktree info --slug <slug>); the unit's code-summary.md"
outputs: "git push of the unit's bolt-<slug> branch to origin; jira-bridge-implementation-log.md appended at the intent record root"
---

# Bolt Push and Log

Runs once per Unit, immediately after that Unit's `code-generation` stage
completes (`requires_stage: [code-generation]` — a plugin stage may name a
core stage's slug here without editing that core stage's own file; the graph
builder only needs the referenced slug to exist).

## Purpose

AIDLC's construction phase already creates an isolated `bolt-<slug>` git
branch/worktree per Unit (`.claude/tools/aidlc-worktree.ts`), but never
pushes it and never writes a human-readable log of which branch holds what.
Looper does both today (`looper-code/commands/looper-implement.md`: commit,
`git push -u origin <branch-name>`, then append a phase-completion section —
repos changed, branch(es), base branch — to the RePIT `.md`). This stage adds
the same two things to AIDLC's per-unit flow, without touching
`aidlc-worktree.ts` or `code-generation.md`.

## Steps

### Step 1: Resolve the unit's worktree

Run `aidlc engine worktree info --slug <kebab-slug-of-this-unit's-bolt>`
(schema: `.claude/knowledge/aidlc-shared/worktree-info-schema.md`). This
returns `{ slug, path, branch_name, audit_timestamp, merge_held }` for the
most recent `WORKTREE_CREATED` entry for that slug.

- If the command exits non-zero (no worktree for this slug — the unit did
  not run under per-Bolt worktree/swarm mode): run
  `aidlc engine orchestrate report --stage jira-bridge-bolt-push-log --result skipped`
  and stop.
- Otherwise capture `path`, `branch_name`.

### Step 2: Find the base branch and repo

Read the matching `WORKTREE_CREATED` audit entry for this slug (per
`.claude/knowledge/aidlc-shared/audit-format.md`) and extract its `Base
branch` and `Repo` fields — `info` in Step 1 does not surface these, but the
audit entry that `info` itself reads always carries them.

### Step 3: Push the branch

`git -C <path> push -u origin <branch_name>`. If the push fails (no remote
configured, auth failure, rejected — e.g. the base branch moved upstream),
report the failure verbatim to the user and stop this stage without writing
Step 4's log entry for this unit — do not log a push that didn't happen.

### Step 4: Read the unit's implementation summary

Read `<record>/construction/<unit>/code-generation/code-summary.md` (already
produced by the core `code-generation` stage — this stage's `consumes:
code-summary` entry) for a one-line "what changed" summary.

### Step 5: Append to the running implementation log

Append a section to `jira-bridge-implementation-log.md` at the intent's record root
(create it with a one-line header the first time this stage runs for the
intent; append to it on every subsequent unit — never overwrite prior
sections):

```
### Unit <unit-id> — <unit name> — pushed
- Repo: <repo from Step 2>
- Branch: <branch_name from Step 1>
- Base branch: <base branch from Step 2>
- Summary: <one line from code-summary.md>
```

This is the human-browsable "what's implemented in which branch" record the
team's looper harness already produces per phase — today, in AIDLC, that
branch↔unit link only exists buried in `worktree-meta.json`/audit events;
this file surfaces it as one scannable running log.

### Step 6: Report completion

Run `aidlc engine orchestrate report --stage jira-bridge-bolt-push-log --result completed`.
