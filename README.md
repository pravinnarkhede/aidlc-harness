# aidlc-harness

This repo is an AWS AI-DLC (AIDLC) install extended with one plugin —
`plugins/auxiliary/` — that closes the gap between "I have a ticket" and
"AIDLC is analyzing the right code." It is generic: nothing in it is tied to
any one project, team, or repo list. Use this same harness for every project
you work on; only the *content* it gathers (repos, architecture, standards)
is project-specific, and that content lives per-project, not in the harness
itself.

## Why this exists

Out of the box, AIDLC starts from a freeform typed description and assumes
the code you're working on is already sitting on disk, already understood,
with no connection to your ticket tracker. In practice, real work usually
starts from a ticket, touches several repos at once, and needs the AI to
actually understand the codebase before it can plan anything useful. Doing
that by hand every time — find the ticket, figure out which repos it
touches, clone them, explain the architecture to the AI, remember to push
and log what got built — is exactly the kind of repetitive setup tax that
should be automated once and reused forever.

The `auxiliary` plugin automates that tax without changing a single AIDLC
core file, so it survives `aidlc update` cleanly and can be dropped into any
AIDLC project.

## What you get

- **A ticket key becomes a working, isolated environment automatically.**
  Type `/aidlc GOLF-123 <description>` and the plugin fetches the real
  ticket, works out which repos it affects, clones exactly those, and hands
  off into AIDLC's normal requirements → design → construction pipeline —
  no manual repo hunting, no manual cloning.
- **The AI actually understands your codebase before it plans anything.**
  A one-time onboarding pass reverse-engineers every repo in the project and
  writes an architecture overview and coding-standards doc that every
  subsequent ticket's planning stages can cite — instead of every ticket
  starting from a blank slate.
- **Multiple tickets can be worked at once without stepping on each other.**
  Each ticket gets its own folder, its own clone of the repos it touches,
  its own AIDLC state — fully isolated, verified by a safety check that
  refuses to proceed if a session isn't pointed at the right folder.
- **Construction branches get pushed and logged without being asked.**
  Every unit of work's branch is pushed to origin automatically, with a
  running human-readable log of which branch holds what — no digging
  through worktree metadata to find out what happened.
- **One harness, reused across every project.** The generic base (this
  repo, minus any project-specific content) never needs to be rebuilt per
  project — only re-pointed at a new set of repos.

## How it fits together — one harness, many projects

```
aidlc-harness/                (this repo — the generic base)
├── .claude/                  the AIDLC engine itself
├── plugins/auxiliary/        the plugin — project-agnostic, reusable as-is
├── aidlc/spaces/default/
│   ├── memory/                org.md + team.md: generic defaults, no
│   │                          project specifics — safe to keep as-is
│   │                          across every project
│   └── knowledge/             starts empty; onboarding fills this in
│                              per project
├── stable-codebase/          created by onboarding, per project
└── <TICKET-KEY>/              created per ticket, per project
```

Everything that's genuinely project-specific — the repo catalog, the
architecture overview, coding standards, and any `project.md` entries —
gets written *after* onboarding, never shipped with the harness. If you
work multiple distinct projects out of copies of this repo, keep this base
(harness + plugin + generic memory) identical across all of them, and let
each project's own onboarding pass populate its own `knowledge/`,
`stable-codebase/`, and `project.md` independently. (How you keep multiple
projects' populated state separate — separate clones, separate branches,
whatever fits your setup — is up to you; this repo doesn't prescribe it.)

## Initial setup (once, for this harness)

1. **Install `bun`** — needed to run the plugin tooling. The compiled
   `aidlc` binary alone has a known issue resolving its own bundled
   compose-hook template, so plugin `validate`/`build`/`sync` need the real
   `bun` runtime:
   ```powershell
   irm bun.sh/install.ps1 | iex
   ```
2. **Build and sync the plugin** (from inside `aidlc-harness/`):
   ```
   bun .claude/tools/aidlc-plugin-validate.ts plugins/auxiliary
   bun .claude/tools/aidlc-plugin-build.ts plugins/auxiliary claude plugins/auxiliary/dist/claude
   ```
   Then, with `CLAUDE_PLUGIN_ROOT` pointed at that build output, sync it in:
   ```powershell
   $env:CLAUDE_PLUGIN_ROOT = "D:\...\aidlc-harness\plugins\auxiliary\dist\claude"
   bun .claude/tools/aidlc-plugin.ts sync
   ```
   Confirm it landed clean: `aidlc doctor` should report 0 problems. This
   composes the plugin's three stages + three agents into `.claude/`
   (tracked via ownership sidecars, fully reversible with
   `sync --prune-missing`). Redo this only after changing the plugin's own
   source files — not per project, and not per ticket.
3. **Nothing else to prepare by hand.** No repo list, no catalog, no
   architecture doc — all of that gets gathered by onboarding, described
   next.

## Starting on a project — onboarding

"Onboarding" is the one-time-per-project step where the plugin learns what
your project actually is. It runs automatically, inside the very first
ticket you start for that project — there's no separate onboarding command.

### If this is a brand-new project (repos already exist, never onboarded here)

1. Create a folder for your first ticket and point a session at it (see
   "Working a ticket" below) — same steps as any ticket.
2. Run `/aidlc <TICKET-KEY> <description>` as usual.
3. Before ticket intake runs, `auxiliary-project-onboarding` fires
   automatically and walks you through:
   - **What repos make up this project?** Give it a name, clone URL, and
     branch per repo — or point it at an existing catalog-shaped file if
     you already have one; it will still confirm the details with you.
   - **What's each repo's role and scope?** In your own words — "this is
     the backend API," "this is the customer-facing web app," and so on.
     This is the part that isn't guessed: your description of intent is
     combined with, not replaced by, what the code turns out to actually
     do.
   - It then clones every repo into a shared, read-only `stable-codebase/`
     mirror and runs AIDLC's own reverse-engineering stage against it.
   - Finally it writes an **architecture overview** and a **coding
     standards** doc under `aidlc/spaces/default/knowledge/`, combining
     your stated roles with what reverse-engineering found — and flags any
     place the two disagree instead of silently picking one.
   - Any proposed additions to `memory/project.md` or `memory/team.md` are
     shown to you for confirmation before anything is written — onboarding
     never edits those files silently.
4. Once onboarding finishes, it marks itself done (a marker file under
   `stable-codebase/`) and your first ticket continues straight into
   normal ticket intake. Every ticket after this one finds onboarding
   already complete and skips it instantly.

### If this project has no code yet (greenfield)

Onboarding's reverse-engineering step needs actual code to analyze, so for
a truly greenfield project there's nothing to reverse-engineer yet. In that
case, either let onboarding run with an empty or partial repo set (it will
simply have less to synthesize), or skip straight to AIDLC's own normal
greenfield flow (`workspace-detection` already handles "no existing code"
correctly on its own) and let onboarding populate the architecture/standards
docs later, once there's a first repo worth analyzing.

### If this project has already been onboarded (by you, earlier, or by a teammate)

Nothing to do — just start your ticket folder and go (see below).
`auxiliary-project-onboarding` checks for its completion marker first and
skips immediately if the project's `stable-codebase/` is already set up,
so re-running it costs nothing.

## Working a ticket, day to day

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
│   ├── <repo-a>/                 at its release branch, refreshed manually
│   ├── <repo-b>/
│   └── aidlc/spaces/default/codekb/...   shared reverse-engineering cache
│
├── <TICKET-KEY>/                ticket folder — your clone + your AIDLC state
│   ├── <repo-a>/                  (only the repos classified "Change")
│   ├── repos.json                 this ticket's clone manifest
│   └── aidlc/spaces/default/
│       ├── intents/...             this ticket's own state — plans,
│       │                           approvals, artifacts — fully isolated
│       └── knowledge/repo-catalog.md
│
├── <ANOTHER-TICKET-KEY>/        a second ticket, fully isolated from the first
└── ...
```

`stable-codebase/` (repos + `codekb/`) and `aidlc/spaces/default/knowledge/`
(the catalog, architecture overview, coding standards) are the only things
shared across every ticket in a project. Everything else — cloned repos,
`repos.json`, per-ticket AIDLC intent state — is isolated per ticket.

### Starting a new ticket

```powershell
# 1. Create the folder
mkdir aidlc-harness\GOLF-123

# 2. Start a session pointed at it (new terminal/window per ticket)
$env:AIDLC_PROJECT_DIR = "D:\...\aidlc-harness\GOLF-123"
$env:CLAUDE_PROJECT_DIR = "D:\...\aidlc-harness\GOLF-123"
claude

# 3. Inside that session, run the workflow with the ticket key first
/aidlc GOLF-123 <description of the ticket>
```

`.claude/` stays physically inside `aidlc-harness/` throughout — only the
*state and repos* redirect to `GOLF-123/`; skills/agents/hooks load normally.

What happens next depends on whether this is the project's first ticket
(onboarding fires first, see above) or a later one — in which case
`auxiliary-ticket-intake` runs directly:
- Verifies the session is actually pointed at a `GOLF-123`-named folder —
  refuses and tells you if it isn't (this is what stops two tickets' work
  from silently mixing together).
- Fetches the real ticket from your tracker (Jira, via the Atlassian MCP
  connector).
- Refreshes `stable-codebase/` if reachable, and seeds this ticket's
  `codekb/` from the shared cache if one already exists.
- Shows a Change / Reference / Out-of-scope table for every cataloged repo
  — confirm or adjust it.
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
/aidlc GOLF-345 <description of the second ticket>
```

`GOLF-123` and `GOLF-345` never see each other's repos, branches, or state.

### Resuming a ticket later

Open a session with `AIDLC_PROJECT_DIR` pointed at that ticket's existing
folder, then run `/aidlc` with no arguments to resume the active intent (or
check `aidlc engine intent list` if you're not sure what's in flight there).

### During construction — branches get pushed and logged automatically

Once a ticket reaches construction, `auxiliary-bolt-push-log` runs after
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

## Quick reference

| I want to... | Do this |
|---|---|
| Set up this harness for the first time | Install `bun`, build + sync the plugin (see Initial setup) |
| Start work on a project for the first time | Create a ticket folder, point a session at it, run `/aidlc <TICKET-KEY> <desc>` — onboarding fires automatically |
| Start a ticket on an already-onboarded project | Same as above — onboarding detects it's already done and skips instantly |
| Work two tickets at once | Separate folder + separate session + separate `AIDLC_PROJECT_DIR` per ticket |
| Resume a ticket | Point a session at its existing folder, run `/aidlc` with no arguments |
| Refresh the shared codebase mirror/analysis | See "Refreshing the shared stable codebase" above — manual, on demand |
| See what's implemented in which branch | `implementation-log.md` in the ticket's record directory |
| Check everything is wired correctly | `aidlc doctor` |

## Known gotchas

- **Full clones, not shallow.** Cloning "Change" repos into a ticket folder
  pulls complete history, not `--depth 1` — budget time/disk for large repos.
- **Repos with unusual filenames can break `git checkout` on Windows** (e.g.
  a path with a trailing space triggers `core.protectNTFS`). If onboarding
  or a clone fails with an NTFS/checkout error, this is almost always the
  cause — check for such a file at the source and rename it, or apply the
  documented `core.protectNTFS=false` workaround locally.
- **Forgetting to set `AIDLC_PROJECT_DIR`.** The stage's safety check catches
  this (refuses to proceed if the folder name doesn't match the ticket key)
  — start a fresh session per the steps above rather than fixing the path
  mid-session.
- **A future `aidlc update` renaming a core stage this plugin depends on**
  (e.g. `workspace-detection`, `code-generation`) would silently break the
  plugin's `requires_stage` references. Run `aidlc doctor` after any future
  update to catch this early.
