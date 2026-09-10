# jira-bridge

An AIDLC plugin that gives the `/aidlc` workflow the same "ticket in, workspace
ready" front door the team's in-house looper harness already has — without a
new command and without editing any AIDLC core file.

## What it adds

- **`jira-bridge-ticket-intake`** (Initialization phase, conditional on a ticket key
  like `GOLF-123` appearing in the request): fetches the ticket via the
  Atlassian MCP connector, refreshes a read-only `stable-codebase/` mirror of
  the whole platform on every run, classifies and clones the repos the ticket
  impacts (reusing the team's existing `project-structure.md` repo catalog),
  and records the parsed ticket content for downstream planning stages.
- **`jira-bridge-bolt-push-log`** (Construction phase, per unit of work): after a
  unit's code-generation finishes in its own Bolt worktree, pushes that
  branch to origin and appends a human-readable entry to a running
  `jira-bridge-implementation-log.md` — repo, branch, base branch, one-line summary —
  so you can see what's implemented in which branch without digging through
  `worktree-meta.json`.

## Working a ticket — one folder, one session, per ticket

Each ticket gets its own fully isolated folder directly inside `aidlc-harness/`
— its own cloned repos **and** its own AIDLC intent state, e.g.:

```
aidlc-harness/
  stable-codebase/          (shared, read-only, one copy for every ticket)
  GOLF-123/
    golfler_asp_2/  sgs-cts-angular/  ...
    aidlc/spaces/default/intents/...  (this ticket's own AIDLC state)
  GOLF-345/
    ...
```

This isolation is set up **before** the session starts, not by a stage —
AIDLC resolves every path for the whole session from `AIDLC_PROJECT_DIR`
(falls back to the harness root otherwise), and that resolution happens the
moment the very first `/aidlc` command runs, before any stage (core or
plugin) executes. So per ticket:

1. `mkdir aidlc-harness/GOLF-123`
2. Start a session with `AIDLC_PROJECT_DIR` set to that folder's absolute
   path (`.claude/` itself stays physically at `aidlc-harness/.claude` —
   the harness install path resolves independently of this variable, so
   skills/agents/hooks still load normally).
3. Run `/aidlc GOLF-123 <description>` inside that session.

From here on, `jira-bridge-ticket-intake` operates entirely against whatever
`resolveProjectDir()` returns for the session — which is now `GOLF-123/` —
so the ticket's repos and its AIDLC state land there consistently with no
further steps. The stage also verifies this precondition was actually
followed (the resolved project folder's name must contain the ticket key)
and refuses to proceed otherwise, to avoid silently mixing one ticket's
clone/state into another ticket's folder or into the harness root.

## Refreshing the shared reference analysis (manual, on-demand)

`jira-bridge-ticket-intake` seeds each new ticket's `codekb/` from a shared,
harness-level reverse-engineering cache so every ticket after the first one
starts from a warm analysis instead of a full rescan — but it never
refreshes that shared cache itself, by design (an on-every-ticket refresh
would make ticket intake slower, not faster). Refresh it yourself, whenever
you want, with no scheduler required:

1. Refresh the mirror's repos to their release-branch tips (the same
   fetch/checkout/reset routine `jira-bridge-ticket-intake` Step 4 runs
   automatically on every ticket anyway, so this is usually already current):
   ```
   git -C aidlc-harness/stable-codebase/<repo> fetch --depth 1 origin <branch>
   git -C aidlc-harness/stable-codebase/<repo> checkout <branch>
   git -C aidlc-harness/stable-codebase/<repo> reset --hard origin/<branch>
   ```
2. Point a session at the mirror itself (`AIDLC_PROJECT_DIR=aidlc-harness/stable-codebase`)
   and run the existing `/aidlc-reverse-engineering` skill there. This is
   AIDLC's own core reverse-engineering stage, run in isolation — no
   reimplementation here, and its own `codekb-scope-diff` freshness check
   means only what actually changed since the last run gets re-analyzed, not
   every file from scratch. Output lands at
   `aidlc-harness/stable-codebase/aidlc/spaces/default/codekb/<repo>/`.

Every ticket started after that automatically seeds from whatever's there —
run this as often or as rarely as you like; a ticket started before you ever
run it just falls back to a full scan on its own, exactly like today.

## Install / test locally

```
aidlc engine plugin validate .
aidlc engine plugin build claude dist/claude
```

Then, to try it in a project without publishing to a marketplace, set
`CLAUDE_PLUGIN_ROOT` (or `AIDLC_PLUGIN_ROOT`) to `dist/claude` and run
`aidlc engine plugin sync` from inside the target project. This composes
`stages/` and `agents/` into that project's `.claude/aidlc-common/stages/`
and `.claude/agents/`, and recompiles `stage-graph.json`/`scope-grid.json` —
no hand-edit of any file in the target project, ever. `aidlc engine plugin
sync --prune-missing` cleanly removes everything this plugin added.

## User-owned files this plugin depends on (not shipped, not composed)

- `aidlc/spaces/<space>/knowledge/repo-catalog.md` — the repo/branch/clone-URL
  catalog, seeded once from `looper-code/artifacts/project-structure.md` if
  reachable, otherwise created as a placeholder for you to fill in.
- `<ticket-root>/repos.json` (e.g. `aidlc-harness/GOLF-123/repos.json`) —
  read by the existing core `aidlc-workspace-sync.ts` tool; this plugin only
  writes it.
