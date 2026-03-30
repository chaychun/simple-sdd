# simple-sdd

A very simple spec-driven development workflow for [Claude Code](https://claude.com/product/claude-code). Three skills and a hook that turn every work request into a structured workflow: discuss first, write a spec, then execute from it.

## Why it exists

I very much like the idea of spec-driven development. You stay at a higher level while your agent takes care of the code. The core idea is that specs become your equivalent of the source code over time, and the agent makes sure the real code follows them.

But I find existing SDD workflows to be too much. They often produce dense, hard-to-read artifacts which put mental friction on the user. The result is that you are less throurough with them. So I want to create my own workflow where the spec file is easy to read, while maintaining the accuracy of the implementing agent.

## What it does

Instead of jumping straight to code, every task flows through:

1. **`/brainstorm`** — Collaborative discussion that produces a spec file in `specs/` as the decision record
2. **`/implement-spec`** — Executes work from a spec, updating progress and learnings as it goes
3. **`/wrap-up`** — Code review, closes out the spec, handles commit/push/PR

The **SessionStart hook** reminds Claude to use this workflow at the start of every conversation.

Over time, `specs/` becomes your project's institutional memory — not just what was done, but why.

## Install

### As a Claude Code plugin

```
/plugin marketplace add chaychun/simple-sdd
/plugin install simple-sdd@simple-sdd
```

Once installed, the skills are available as `/simple-sdd:brainstorm`, `/simple-sdd:implement-spec`, and `/simple-sdd:wrap-up`.

### Manual install

Copy the skills into your project's `.claude/skills/` directory:

```bash
# From your project root
mkdir -p .claude/skills
cp -r path/to/simple-sdd/skills/* .claude/skills/
```

For the hook, merge the contents of `hooks/hooks.json` into your project's `.claude/settings.json`. If you don't have one yet, copy it directly:

```bash
cp hooks/hooks.json .claude/settings.json
```

If you already have a `.claude/settings.json`, add the `SessionStart` hook entry to your existing `hooks` object.

## Usage

Start any task with `/brainstorm`:

```
/brainstorm add user authentication
```

Once the spec is written and approved, execute it:

```
/implement-spec
```

When the code is working, close it out:

```
/wrap-up
```

## Spec lifecycle

```
brainstorming → ready → implementing → complete
```

Each spec lives in `specs/` at the project root and tracks intent, decisions, scope, approach, progress, review, and learnings. Specs reference each other, so patterns and decisions accumulate across the project.

## License

MIT
