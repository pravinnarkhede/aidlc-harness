---
name: aux-project-onboarding
depth: minimal
keywords: []
skeleton: off
---

# Auxiliary Project Onboarding

A minimal, single-stage bootstrap run for onboarding an auxiliary project
into the workspace: clones the cataloged repositories, reverse-engineers
their structure, and synthesizes `architecture-overview.md`,
`coding-standards.md`, and proposed memory entries for human review.

This scope runs only the initialization spine
(`workspace-scaffold`, `workspace-detection`, `auxiliary-project-onboarding`,
`state-init`) and skips every ideation, inception, construction, and
operation stage — there is no application code to design, build, or ship
here, only the onboarding artifacts the auxiliary-project-onboarding stage
produces.

**Skeleton default: off.** This is a single-stage bootstrap run, not
application code, so there is no walking-skeleton Bolt to gate.

Composed via `/aidlc compose`; not keyword-inferable — resolve it explicitly
with `--scope aux-project-onboarding`.
