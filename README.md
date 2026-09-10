# aidlc-harness

This repo is an AWS AI-DLC (AIDLC) install, extended with a plugin —
`plugins/jira-bridge/` — that gives the `/aidlc` workflow a "ticket in,
workspace ready" front door: point it at a Jira ticket key and it fetches the
ticket, clones the repos it impacts into a dedicated folder, seeds a
reverse-engineering cache, and hands off into AIDLC's normal
requirements → design → construction pipeline. No separate command, no core
AIDLC file ever hand-edited — see `plugins/jira-bridge/README.md` for how the
plugin itself is built.

This file is the practical guide: what's here, how to set it up once, and how
to actually work a ticket day to day.

## What's in this repo

```
aidlc-harness/
├── .claude/                    the AIDLC harness itself (agents, stages, tools)
├── plugins/
│   └── jira-bridge/            the plugin source (build/install docs in its own README)
├── stable-codebase/            shared, read-only repo mirror + reverse-engineering cache
│                                (created the first time you run a ticket; see below)
├── GOLF-123/                   one folder per ticket you work — created by you
├── GOLF-345/
└── ...
```

## One-time setup

1. **Install `bun`** (needed to run the plugin tooling — the compiled `aidlc`
   binary alone has a known issue resolving its own bundled compose-hook
   template, so plugin `validate`/`build`/`sync` need the real `bun` runtime):
   ```powershell
   irm bun.sh/install.ps1 | iex
   ```
2. **Build and sync the plugin** (from inside `aidlc-harness/`):
   ```
   bun .claude/tools/aidlc-plugin-validate.ts plugins/jira-bridge
   bun .claude/tools/aidlc-plugin-build.ts plugins/jira-bridge claude plugins/jira-bridge/dist/claude
   ```
   Then, with `CLAUDE_PLUGIN_ROOT` pointed at that build output, sync it in:
   ```powershell
   $env:CLAUDE_PLUGIN_ROOT = "D:\...\aidlc-harness\plugins\jira-bridge\dist\claude"
   bun .claude/tools/aidlc-plugin.ts sync
   ```
   Confirm it landed clean: `aidlc doctor` should report 0 problems. This
   composes the plugin's two stages + two agents into `.claude/` (tracked via
   ownership sidecars, fully reversible with `sync --prune-missing`) — you
   only need to redo this after you change the plugin's own source files.
3. **Fill in your repo catalog once.** The plugin needs to know your team's
   repos (name, clone URL, release branch, which API each one talks to). The
   first ticket you ever run will create a placeholder for you to fill in at
   `aidlc/spaces/default/knowledge/repo-catalog.md` (or seed it automatically
   from `looper-code/artifacts/project-structure.md` if that's reachable from
   this repo) — after that, copy the filled-in file into each new ticket
   folder's own `aidlc/spaces/default/knowledge/` before running `/aidlc`.

## How to use it — one ticket at a time

### The one rule that matters

**One ticket = one folder = one session, and the session must be pointed at
that folder before you type `/aidlc` for the first time.** AIDLC resolves
every path (intent state, cloned repos, worktrees) for the whole session the
moment your very first `/aidlc` command runs. Skip this and everything lands
at the harness root instead of in an isolated ticket folder.

### Folder structure once you're working tickets

```
aidlc-harness/
├── stable-codebase/            shared, read-only mirror of every cataloged repo
│   ├── golfler_asp_2/            at its release branch, refreshed manually
│   ├── sgs-cts-angular/
│   └── aidlc/spaces/default/codekb/...   shared reverse-engineering cache
│
├── GOLF-123/                   ticket folder — your clone + your AIDLC state
│   ├── golfler_asp_2/            (only the repos classified "Change")
│   ├── sgs-cts-angular/
│   ├── repos.json                this ticket's clone manifest
│   └── aidlc/spaces/default/
│       ├── intents/...            this ticket's own state — plans,
│       │                          approvals, artifacts — fully isolated
│       └── knowledge/repo-catalog.md
│
├── GOLF-345/                   a second ticket, fully isolated from GOLF-123
└── GOLF-567/
```

`stable-codebase/` (repos + `codekb/`) is the only thing shared across
tickets. Everything else — cloned repos, `repos.json`, AIDLC intent state —
is per-ticket.

### Starting a new ticket

```powershell
# 1. Create the folder
mkdir aidlc-harness\GOLF-123

# 2. Start a session pointed at it (new terminal/window per ticket)
$env:AIDLC_PROJECT_DIR = "D:\...\aidlc-harness\GOLF-123"
$env:CLAUDE_PROJECT_DIR = "D:\...\aidlc-harness\GOLF-123"
claude

# 3. Inside that session, run the workflow with the ticket key first
/aidlc GOLF-123 add mco tee times across clubs
```

`.claude/` stays physically inside `aidlc-harness/` throughout — only the
*state and repos* redirect to `GOLF-123/`; skills/agents/hooks load normally.

From here, the plugin's `jira-bridge-ticket-intake` stage takes over
automatically:
- Verifies the session is actually pointed at a `GOLF-123`-named folder —
  refuses and tells you if it isn't (this is what stops two tickets' work
  from silently mixing together).
- Fetches the real ticket from Jira.
- Refreshes `stable-codebase/` if reachable, and seeds this ticket's
  `codekb/` from the shared cache if one already exists.
- Shows a Change / Reference / Out-of-scope table for every cataloged repo —
  confirm or adjust it.
- Clones the "Change" repos into `GOLF-123/`.
- Hands off into AIDLC's normal pipeline (reverse-engineering, requirements,
  planning, construction...).

### Working a second ticket at the same time

Repeat in a **different terminal/session** with a different folder and env
var — nothing else changes:

```powershell
mkdir aidlc-harness\GOLF-345
# new terminal/window
$env:AIDLC_PROJECT_DIR = "D:\...\aidlc-harness\GOLF-345"
claude
```
```
/aidlc GOLF-345 add multi-club dashboard filter
```

`GOLF-123` and `GOLF-345` never see each other's repos, branches, or state.

### Resuming a ticket later

Open a session with `AIDLC_PROJECT_DIR` pointed at that ticket's existing
folder, then run `/aidlc` with no arguments to resume the active intent (or
check `aidlc engine intent list` if you're not sure what's in flight there).

### During construction — branches get pushed and logged automatically

Once a ticket reaches construction, `jira-bridge-bolt-push-log` runs after
each unit's code-generation: pushes that unit's `bolt-<slug>` branch to
origin and appends an entry to `implementation-log.md` in the ticket's record
directory — repo, branch, base branch, one-line summary.

### Refreshing the shared stable codebase + analysis (manual, whenever you like)

No scheduler — run this yourself, as often as you want:

```
git -C aidlc-harness/stable-codebase/<repo> fetch --depth 1 origin <branch>
git -C aidlc-harness/stable-codebase/<repo> checkout <branch>
git -C aidlc-harness/stable-codebase/<repo> reset --hard origin/<branch>
```

Then, in a session with `AIDLC_PROJECT_DIR=aidlc-harness/stable-codebase`,
run `/aidlc-reverse-engineering` — AIDLC's own core stage, run in isolation,
updating the shared `codekb/` incrementally. Every ticket started afterward
seeds from it automatically.

## Known gotchas

- **Full clones, not shallow.** Cloning "Change" repos into a ticket folder
  pulls complete history, not `--depth 1` — budget time/disk for large repos.
- **Windows + `cc_ios`.** That repo has a folder with a trailing space in its
  name that breaks `git checkout` on Windows (`core.protectNTFS`). If a
  ticket's scope includes it, expect the same manual workaround documented
  in the repo catalog.
- **Forgetting to set `AIDLC_PROJECT_DIR`.** The stage's safety check catches
  this (refuses to proceed if the folder name doesn't match the ticket key)
  — start a fresh session per the steps above rather than fixing the path
  mid-session.
