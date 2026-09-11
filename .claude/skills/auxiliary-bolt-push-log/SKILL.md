---
name: auxiliary-bolt-push-log
generated-by: aidlc-runner-gen
description: >
  Run the auxiliary plugin `auxiliary-bolt-push-log` stage (construction phase) in isolation, without
  advancing the main workflow. Packages `/aidlc --stage auxiliary-bolt-push-log --single`:
  the engine emits one run-stage directive for auxiliary-bolt-push-log and its gate, the
  conductor runs it, then the single-stage run commits a synthetic-id pair and
  stops. The main workflow's Current Stage is never touched.
argument-hint: ""
user-invocable: true
---

# AI-DLC Stage Runner — auxiliary-bolt-push-log

Run the `auxiliary-bolt-push-log` stage from the auxiliary plugin on its own. This is opt-in packaging over
`/aidlc --stage auxiliary-bolt-push-log --single`; the same stage is always reachable via
that flag without this skill.

## Steps

1. Ask the engine for the single-stage directive:

   ```bash
   aidlc engine orchestrate next --stage auxiliary-bolt-push-log --single
   ```

   The engine emits one `run-stage` directive for `auxiliary-bolt-push-log` (carrying the
   lead agent, the resolved consumes/produces paths, the rules and sensors in
   context, and — on this first directive — the conductor persona). Run the stage
   exactly as the directive describes; do not load the conductor persona by hand,
   the engine delivers it.

2. Before acting on the directive, read
   `.claude/aidlc-common/protocols/stage-protocol.md`. Then read every
   `.claude/aidlc-common/protocols/stage-protocol-<module>.md` named by
   `directive.protocol_modules`. Load every listed module before reading the
   stage body or running its topology; skip only a module already loaded earlier
   in this session.

3. When the stage's work is done, commit the single-stage record:

   ```bash
   aidlc engine orchestrate report --single --stage auxiliary-bolt-push-log --result completed
   ```

   This records a STAGE_STARTED / STAGE_COMPLETED pair under a synthetic workflow
   id and stops. It NEVER writes the main workflow's `Current Stage` — a
   single-stage run is isolated by design (the tool refuses to advance the main
   workflow).
