---
name: web-audit
description: Audit a personal website or portfolio for SEO, accessibility, security, and code-hygiene issues, and propose prioritized fixes. Use when asked to review a personal site, check what can be improved on a site, or do a pre-launch/periodic health check on one.
---

# Website audit

A practical checklist for auditing a personal website/portfolio, distilled from a real audit of a FastAPI + Jinja2 site. Framework-agnostic — apply whichever items are relevant to the stack in front of you.

Work through the categories below, note concrete findings (file + line, not vague statements), then present them grouped by category with a suggested priority before making changes. Don't fix everything unprompted — a site can span SEO, a11y, cleanup and security at once, and the owner may not want all of it done in one pass; ask which categories to act on first.

## SEO

- `<title>` — present, non-empty, ideally distinct per page/route, and if the site is auto-translated/bilingual it should differ per locale.
- Open Graph tags (`og:title`, `og:description`, `og:image`, `og:type`, `og:url`) and Twitter Card tags (`twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`) — check for a link-preview when shared, not just tag presence.
- `<link rel="canonical">` pointing at the real production URL — get the actual domain from README/deploy config/DNS, never invent one.
- `robots.txt` and `sitemap.xml` — confirm they're actually reachable at `/robots.txt` and `/sitemap.xml`, not just present as static files that never get routed.
- For multi-language sites: `<link rel="alternate" hreflang="...">` pairing every locale, plus an `x-default`.
- JSON-LD structured data appropriate to the page (`Person` for a personal/portfolio site, `Organization`/`Article` etc. elsewhere). Build it as a real object server-side and render with a JSON-safe serializer (e.g. Jinja's `tojson`, not raw string interpolation) — string-interpolating into a `<script type="application/ld+json">` block silently corrupts it, because HTML auto-escaping turns `&` into `&amp;` inside what must be valid JSON. Validate the rendered output actually parses as JSON before calling it done.

## Accessibility

- Icon-only buttons (hamburger menus, close buttons) need `aria-label` **and**, if they toggle visible state, `aria-expanded` kept in sync by the JS that drives them.
- Decorative icons/emoji (paired with adjacent visible text) should be `aria-hidden="true"` — otherwise screen readers announce the Unicode glyph name as noise.
- Any place JS updates a status message (form success/error, async feedback) needs `aria-live="polite"` (or `role="status"`) on the container, or screen-reader users never hear the update.
- Check for a blanket `outline: none` on form inputs/buttons — a very common a11y regression. Prefer `:focus-visible` with a real visible outline over removing focus indication entirely.
- A skip-to-content link as the first focusable element, for sites with a fixed/sticky nav.
- Active-state toggles (language switcher, tabs) should expose state via `aria-current`, not CSS class alone.

## Security (even on a low-traffic personal site)

- Any user-submitted input (contact form, comments) that gets echoed into an email body, log, or page must be HTML-escaped at the point of interpolation — check *both* directions: what's shown back to the visitor, and what's sent to the owner's inbox. An unescaped contact-form field is a real (if low-severity) HTML-injection vector into someone's email client.
- Grep for raw string interpolation building HTML/email bodies (f-strings, template literals) — that's where escaping gets skipped.
- Template `| safe` / `dangerouslySetInnerHTML`-equivalent filters: fine when the source is owner-controlled content (YAML/CMS the owner edits), worth flagging (not necessarily changing) if that content could ever come from a less trusted source.

## Code hygiene / cleanup

- Cross-reference declared dependencies against actual imports — a dependency the code stopped using after a refactor (e.g. swapping a mail API for raw SMTP) often lingers in `requirements.txt`/`package.json`.
- `git status --porcelain` to catch untracked files sitting in the working tree that aren't referenced anywhere — orphaned assets that got added but never wired up or never cleaned up.
- Grep content/config data files (YAML/JSON) for keys the templates/components never read — a common failure mode is a duplicate block shadowing the one actually in use (e.g. a `certifications:` list nobody renders because only `badges:` is used), which then silently drifts out of sync with what's live.
- Hardcoded values duplicated from a config/content source (e.g. numbers hand-typed into a template that also exist in YAML) — wire the template to read from the single source instead of carrying two copies that can diverge.
- Fragile assumptions on data shape, e.g. `name.split()[0]`/`[1]` to derive first/last name — breaks on a single-word or three-word name.

## Process (worth flagging, often out of scope for a quick pass)

- No tests, no linting, and a CI/CD pipeline that deploys straight from `main` with zero checks — flag this but treat it as a separate, larger effort from the content/SEO/a11y/security pass; don't bundle it in without asking.

## How to close out an audit

1. List findings per category with file:line references.
2. Ask the owner which categories to act on now (don't assume "all of it").
3. For anything you're about to delete (unused assets, unused config), confirm it's genuinely unreferenced (grep templates/components, check git tracked status) before removing.
4. After changes, actually run the app locally and hit the routes you touched (`/`, `/robots.txt`, `/sitemap.xml`, any JSON-LD block) rather than trusting the diff alone — templating bugs (like the `tojson` issue above) only show up when rendered.
