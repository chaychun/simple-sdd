---
name: brainstorm
description: "Spec-driven workflow entry point. Use this for any work request — building a feature, refactoring code, investigating a bug, analyzing a system, migrating something, fixing an issue. Runs a collaborative discussion, explores the codebase as needed, and produces a spec as the decision record and working contract. Trigger on: 'let's build X', 'refactor X', 'investigate X', 'analyze and fix X', 'migrate X to Y', 'how should I approach X', 'brainstorm X', or any request where thinking before doing would help. Always use brainstorm rather than jumping straight to execution — the spec is a record, not just a plan."
---

# Brainstorm

This skill runs the spec-driven workflow. It guides a structured conversation with the user, explores the codebase when relevant, and produces a spec file that serves as the decision record, the working contract, and the agent's reference throughout execution.

The output of a brainstorm session is always a spec. The spec is what gets reviewed before work begins.

## Specs as living truth

Over time, `specs/` becomes the project's institutional memory — not just a log of what was done, but why. Every non-obvious decision, every deliberate tradeoff, every pattern that emerged from hard experience lives here.

This only works if specs stay honest and connected:

- **Update superseded specs.** When a pattern from an old spec no longer applies, add a note pointing to the new one. Don't leave contradictions silently accumulating.
- **Reference what you're building on.** If this work builds on or changes something in a previous spec, link it in `refs.specs`.
- **Keep Learnings real.** Useful Learnings are specific — not "this went well" but "the approach in auth.md works better for stateful flows; avoid it for stateless ones."
- **Scan past specs at the start of every brainstorm.** Patterns and decisions from previous work should inform this one.

The goal: someone reading the specs directory should understand not just what the codebase does, but why it is the way it is.

## Before you begin

If `specs/` exists in the project root, spawn a subagent to explore it before the discussion starts. This keeps your context clean — the subagent does the reading, you get a summary.

**Subagent prompt:**

```
Explore specs/ in the current directory. The user wants to work on: [describe the request briefly].

1. List all spec files with their frontmatter status.
2. Flag any with status: ready, implementing, or reviewing — these are in-progress.
3. For specs that seem relevant to this request, read their ## Decisions and ## Learnings sections.
4. Return a concise summary:
   - In-progress specs (name + status)
   - Relevant past decisions that apply to this work
   - Relevant learnings worth knowing before the discussion
   - Any specs this work is likely to build on (for refs.specs)

Be selective — only surface what's actually relevant. If nothing is relevant, say so briefly.
```

Use the subagent's summary to:

- Surface any in-progress spec that matches this request ("I see there's an existing spec for X — want to continue that one?")
- Bring relevant past decisions and learnings into the discussion naturally
- Pre-populate `refs.specs` candidates in the spec you'll write

## Phase 1: Discussion

Start by understanding what the user wants and why. You're not just gathering requirements — you're thinking alongside them.

The discussion has two goals:

- **Clarity**: what exactly needs to happen, why, and what does success look like?
- **Alignment**: confirm the approach makes sense before any work begins

Things to surface naturally during the conversation:

- The "why" — what problem does this solve, or what goal does it serve?
- What's in scope vs. explicitly out of scope
- Technical or preference-based constraints
- Tradeoffs between approaches
- Patterns from past specs that might apply

Keep it conversational. Don't front-load all your questions. Let the discussion develop organically, and explore the codebase when it helps clarify something — discovery and discussion can happen at the same time.

**When to stop discussing and write the spec:** When you have enough clarity to describe what will happen, how, and why. Ambiguity on details is fine — that's what `## Approach` is for. Ambiguity on intent and scope is not.

## Phase 2: Codebase Exploration

Explore the codebase when it informs the discussion — not as a separate phase, but as a natural part of it. Look at:

- What already exists that this touches or changes
- Conventions and patterns in use
- Potential conflicts or dependencies
- For refactors/fixes: the current state of what's being changed

Share what you find as you discover it. Often this changes the direction of the conversation.

## Phase 3: Writing the Spec

When there's enough clarity, write the spec.

**File location:** `specs/` in the project root. Start flat (`task-name.md`). Create subdirectories only when multiple related specs exist (e.g. `auth/login.md`, `auth/refresh.md`). Use kebab-case, task-focused names — not dates.

**Existing spec:** If one already exists for this task, update it rather than creating a new one. Preserve existing content, revise what's changed, update `updated` date and `status`.

**New spec:** Use the template at `assets/spec-template.md`. Fill in what you know. Leave `## Approach` for after exploration if you haven't done it yet.

**Sections to fill before handing off:**

- `## Intent` — always required
- `## Decisions` — always required (even if brief)
- `## Scope` — always required
- `## Approach` — 3–5 sentences, written for the user to evaluate the strategy. Cover what will be done and why this way — not file paths, token names, or step-by-step notes. If the approach is simple, one sentence is fine.
- `## Implementation Notes` — technical specifics from codebase exploration: file paths, token/variable names, API signatures, step-by-step notes. This goes below the `---` divider at the bottom of the spec, under the "agent reference" label. Only include if there's substantive technical detail worth capturing.
- `## Future Notes` — optional; use when the discussion covered more ground than this spec implements. Capture deferred ideas with enough context that future specs can build on what was already worked through here.
- `## Progress` — set to "spec written, pending review"
- `## Review` and `## Learnings` — leave as template placeholder text

**Frontmatter:**

- `status`: set to `ready` once the user approves, `brainstorming` while in progress
- `updated`: update whenever the spec changes
- `refs.specs`: link to related specs this builds on
- `refs.files`: key files this task touches

## Phase 3.5: Spec Review

Before presenting the spec to the user, spawn a subagent to review whether the described approach is actually implementable. This is a blocking step — the user only sees a spec that has passed review.

The reviewer checks the **approach**, not the intent. It explores the codebase to verify that the patterns, files, components, and APIs described in the spec actually exist and are compatible with how things currently work.

**Subagent prompt:**

```
You are reviewing the technical approach described in a spec before it's shown to the user. Your job is to verify that the approach is actually implementable — not to judge the intent or decisions.

## Spec title: [TITLE]

## Approach to review:
[SPEC ## Approach SECTION]

## Decisions made:
[SPEC ## Decisions SECTION]

## Scope:
[SPEC ## Scope SECTION]

Explore the codebase as needed to:
1. Confirm that files, components, APIs, patterns, or libraries mentioned in the approach actually exist
2. Check whether the described pattern is compatible with how those parts of the codebase currently work
3. Identify architectural conflicts, missing dependencies, or steps that won't work as described

Return structured feedback:

STATUS: PASS | NEEDS_REVISION

ISSUES (if NEEDS_REVISION):
- [BLOCKING] Description — what's wrong and what to change in the spec
- [WARNING] Description — potential concern worth addressing

VERIFIED:
- What you confirmed exists and is compatible with the approach

NOTES:
- Anything that couldn't be fully verified from static analysis alone
```

**After receiving feedback:**

- **PASS**: proceed directly to Phase 4
- **NEEDS_REVISION**: apply all feedback to the spec, then assess whether the changes were substantive (approach section significantly changed) or trivial (minor clarification, one small gap filled). Re-run the reviewer only if changes were substantive. If trivial, proceed to Phase 4.

## Phase 4: Handoff

Present the spec to the user and ask them to review it. This is the checkpoint.

- If they approve: update `status` to `ready`. They can run `/implement-spec` now or in a future session.
- If they want changes: update the spec in place, don't just describe the changes verbally.

The spec is a contract — once it's `ready`, work should follow it.
