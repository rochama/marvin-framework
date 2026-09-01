---
name: changelog
description: Write a dated changelog entry after finishing notable work on a project, and keep a living CLAUDE.md current. Use when significant changes just shipped (a feature, an audit pass, a fix) and are worth recording for future reference.
---

# Changelog entries

Keep two things separate rather than merging them into one file:

- **`CLAUDE.md`** at the repo root — the *current, living* reference: tech stack, non-obvious gotchas (deploy branch quirks, env var requirements, known transient errors), what the project is, what's deferred. Keep this updated in place; it should always describe *now*, not history. This filename is fixed — Claude Code auto-loads it when working in the repo, so never rename or move it.
- **`changelog/YYYYMMDD-<short-name>.md`** — an append-only, dated log entry per notable change. Never edit a past entry to reflect new reality; write a new dated entry instead. `<short-name>` is the project/repo name, or a short topic slug if multiple entries land on the same project the same day.

## When to write a changelog entry

After completing something worth remembering later: a feature, a multi-area fix/audit pass, a migration, an incident and its resolution. Not for every trivial commit — use judgment; if it's the kind of thing you'd want to explain to someone joining the project cold, it's changelog-worthy.

## Entry structure

```markdown
# YYYY-MM-DD — <short title>

## What changed
<bullet list, grouped by area if there are several>

## Shipped via
<PR links / commit refs>

## Incidents during rollout
<anything that went wrong and how it was resolved — omit section if nothing did>

## Deferred
<anything flagged but intentionally not done this pass>
```

## After writing an entry

Update `CLAUDE.md` if the change affects anything it documents (new gotcha, new env var, changed deploy behavior, etc.) — the changelog records *that it happened*, `CLAUDE.md` records *how things work now*. Don't let `CLAUDE.md` go stale by only ever appending to the changelog.
