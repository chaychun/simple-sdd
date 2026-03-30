---
name: implement-spec
description: "Execute work from a spec file. Use after brainstorm has produced a spec, or when resuming work from a previous session. Works for any kind of task — building features, refactoring, investigating bugs, running migrations, fixing issues. Reads the spec, does the work, and updates progress and learnings throughout. When done, run /wrap-up to do the code review, close out the spec, and handle the commit. Trigger on: 'execute the spec', 'implement the spec', 'do the work', 'continue [task name]', 'resume the refactor/migration/fix', or when a specs/*.md file exists with status 'ready' or 'implementing' and the user wants to proceed."
---

# Implement Spec

This skill picks up where brainstorm left off. It reads a spec file, executes the work described in it, and keeps the spec updated as a living record of progress and learnings throughout.

Closing out the spec — Review, Learnings, and finishing up — is handled by `/wrap-up`. Run that when implementation and bug-fixing are done.

The spec is the source of truth. If something is unclear during execution, refer back to `## Decisions` and `## Intent` before making assumptions.

## Step 1: Find the Spec

Look at `specs/` in the project root.

**If the user named a spec or task:** find the matching file. If ambiguous (e.g. the name matches multiple specs), present the candidates and ask the user to confirm.

**If no spec is named:** look for specs with `status: ready` or `status: implementing`. If there's one clear candidate, confirm it with the user before proceeding. If there are multiple, list them and ask the user to pick.

**If no ready spec exists:** tell the user and suggest running `/brainstorm` first.

Always confirm the spec with the user before starting.

## Step 2: Read and Understand the Spec

Read the full spec carefully before doing any work. Pay attention to:

- `## Intent` — what this is supposed to accomplish
- `## Decisions` — non-obvious choices that should be honored
- `## Scope` — especially the "Out" section; don't do things listed there
- `## Approach` — the intended strategy; if this section is sparse, explore the codebase to fill in gaps before starting
- `## Progress` — if this is a resumed session, understand what's already been done
- `## Implementation Notes` — if present (below the `---` divider at the bottom), read it for technical specifics: file paths, token names, step-by-step notes from brainstorm exploration. This is the primary reference for implementation details.

Update `status` to `implementing` and set a brief `## Progress` note before starting work.

## Step 3: Execute

Do the work as described in the spec. As you go:

- **Update `## Progress`** at meaningful milestones — after each significant piece is done. Write in plain language: "refactored auth module, working on token service, rate limiting deferred to follow-up." Progress is **additive** — append new entries, never replace or clear what's there.
- **Append to `## Learnings`** when you discover something worth noting — unexpected behavior, a pattern that worked well, a constraint discovered mid-implementation. Learnings are also additive. Don't wait until the end; capture them as they surface.
- **Honor `## Decisions`** — if you hit a situation where the spec's decisions don't apply cleanly, note the divergence in `## Progress` rather than silently doing something different
- **Stay in scope** — if something in the "Out" section turns out to be necessary, stop and flag it to the user rather than expanding scope unilaterally. Check `## Future Notes` too — it may already capture the context around why something was deferred, which helps frame the conversation

The spec is updated in place — not in a separate file, not in comments. It's the living record.

## Step 4: Hand Off

When implementation is done (including fixing any bugs surfaced during testing), tell the user what was completed and suggest running `/wrap-up` to do the code review, close out the spec, and handle the commit.

Don't write `## Review`, don't set `status: complete`, don't remove Implementation Notes — that's all `/wrap-up`'s job.
