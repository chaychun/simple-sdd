---
name: wrap-up
description: "Close out a completed feature. Run after implementation and bug-fixing are done — code is working and behavior matches the spec. Spawns a scoped code review agent to catch logic flaws and spec violations, presents real issues to the user, then closes out the spec (Review, Learnings, status: complete) and handles finishing up (commit, push, PR). Trigger on: 'wrap up', 'close out the spec', 'we're done', 'ready to ship', 'finish up', or after /implement-spec completes."
---

# Wrap Up

This skill closes out a completed feature. It runs after implementation and bug-fixing are done — the code is working, behavior matches the spec, and you're ready to ship.

It does three things in order:

1. Spawns a scoped code review agent to catch bugs and spec violations that testing didn't catch
2. Closes out the spec with Review and Learnings
3. Helps you finish up: commit, push, PR

## Step 1: Find the Active Spec

Look for specs with `status: implementing` in `specs/`. If there's one clear candidate, confirm it with the user before proceeding. If multiple, list them and ask.

If no implementing spec exists, check for `status: ready` — the user may have skipped flagging it mid-implementation. Ask which one to close out.

## Step 2: Read the Spec and Understand the Changes

Read the full spec. Focus on:

- `## Intent` — what this was supposed to accomplish
- `## Decisions` — what was intentionally chosen and why
- `## Scope` — especially Out items (things that should NOT have changed)
- `## Progress` — what was actually done
- `## Implementation Notes` — technical specifics (below the `---` divider)

Then understand what actually changed in the codebase:

- **On a feature branch or worktree**: run `git diff main --stat` for scope, then read the diff for changed files
- **On main**: use `git log --oneline` to find commits related to this work (Progress notes help identify the right range), then `git diff <first-commit>^ HEAD` for the full diff

## Step 3: Derive Targeted Review Instructions

Before spawning the code reviewer, extract specific things to check from the spec. These make the review scoped and useful — not generic.

Derive 3–6 targeted instructions from:

- **Decisions**: what choices were made that need to hold? (e.g., "used optimistic rendering — verify rollback path exists on error")
- **Scope Out**: what should definitely NOT have changed? (e.g., "mobile layout excluded — verify no mobile-specific styles were added")
- **Approach**: what architectural pattern was used? Does the implementation follow it faithfully?
- **Known risks from Progress**: anything flagged during implementation as potentially fragile?

## Step 4: Spawn Code Review Subagent

Spawn a subagent with the spec context and the derived instructions. The subagent should read the actual changed files (not just the diff summary) for any file where logic needs to be understood in context.

**Subagent prompt:**

```
You are reviewing code for a recently completed feature. Review for: logic flaws, spec compliance violations, unintended behaviors, and code quality issues.

## Feature intent:
[SPEC ## Intent SECTION]

## Decisions that should be honored:
[SPEC ## Decisions SECTION]

## Scope — what should NOT have changed:
[SPEC ## Scope OUT SECTION]

## Targeted review instructions:
[DERIVED INSTRUCTIONS FROM STEP 3]

## Changes to review:
[GIT DIFF OR CHANGED FILE CONTENTS]

Return structured feedback:

STATUS: PASS | ISSUES_FOUND

ISSUES (if any):
- [BLOCKING] filepath:line — description (logic flaw or spec violation)
- [WARNING] filepath:line — description (concern worth addressing)
- [NIT] filepath:line — description (code quality, minor)

VERIFIED:
- Specific things confirmed correct per the spec intent and decisions
```

## Step 5: Assess and Filter Feedback

Read the reviewer's output. Before presenting anything to the user, filter false positives:

- Issues that contradict an explicit spec Decision (the spec already decided this — dismiss)
- Issues about things explicitly in `## Scope Out` or deferred in `## Future Notes`
- Style preferences that conflict with visible project conventions

For remaining issues, group by severity: BLOCKING → WARNING → NIT.

Present real issues to the user, grouped by severity. For each BLOCKING and WARNING, show the location, explain the issue, and ask: fix now, defer to a follow-up spec, or dismiss with reasoning?

Nits: present as a grouped list and ask if the user wants them fixed or wants to skip them entirely.

## Step 6: Fix Loop

If the user confirms issues to fix: do the fixes. After fixing, briefly confirm what changed.

Don't re-run the full review unless the user asks. The fixes are scoped and confirmed.

## Step 7: Close Out the Spec

Once the code review loop is done:

1. **Write `## Review`**: What diverged from the plan? What was skipped? What changed unexpectedly? What did the code review catch? Be specific and honest — this is the most useful part for future work.

2. **Update `## Learnings`**: Add new patterns and surprises. If an existing learning no longer holds — the pattern turned out to be wrong, or this work contradicts it — update or remove it rather than leaving a contradiction. Learnings should reflect current understanding, not just a log.

3. **Update related specs**: Check `refs.specs` in the frontmatter, plus any other specs that cover the same area of the codebase. If this work introduces a pattern that supersedes or contradicts something in an older spec's Decisions or Learnings, add a brief pointer there — e.g., "See canvas-arrow-links.md — this approach replaced the per-node handle pattern." Don't rewrite the old spec; just leave a forward reference so future readers aren't misled.

4. **Update frontmatter:**
   - `status: complete`
   - `updated: today's date`
   - `refs.files`: add any key files created or significantly changed

5. **Update `## Progress`**: Append "complete." as the final entry.

6. **Remove `## Implementation Notes`**: Delete the section below the `---` divider (the agent-only reference block). Confirm with the user first: "Remove the implementation notes section before committing?"

## Step 8: Finish Up

Check the current git state (uncommitted changes, untracked files, current branch) and summarize it briefly for the user. Suggest a commit message based on the spec's title and intent, then ask how they want to finish up.

Don't prescribe a flow — just present the state and let the user decide. If they're on a non-main branch or in a worktree, mention it so they can decide about merging or opening a PR. For PRs, use the spec's Intent as the description and Review as the summary of what changed.

Follow the user's choice. Confirm before any push or PR.
