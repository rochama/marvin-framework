# MarVinFramework

Marcus Rocha's personal toolkit of Claude Code skills. Grown from real project work rather than written speculatively.

## Skills

### web-audit

A practical checklist for auditing a personal website/portfolio — SEO, accessibility, security, and code hygiene — distilled from a real audit. See [skills/web-audit/SKILL.md](skills/web-audit/SKILL.md).

### deploy-safety

Verify the real base/deploy branch before merging a PR or claiming a merge will deploy something — written after a merge landed on the wrong branch and didn't actually ship. See [skills/deploy-safety/SKILL.md](skills/deploy-safety/SKILL.md).

### changelog

Write a dated changelog entry (`changelog/YYYYMMDD-name.md`) after finishing notable work, and keep a living `CLAUDE.md` current — separates "what happened" from "how things work now." See [skills/changelog/SKILL.md](skills/changelog/SKILL.md).

## Installation

```
/plugin marketplace add rochama/marvin-framework
/plugin install marvin-framework
```

## Usage

```
/marvin-framework:web-audit
/marvin-framework:deploy-safety
/marvin-framework:changelog
```
