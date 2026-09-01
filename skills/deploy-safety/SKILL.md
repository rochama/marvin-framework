---
name: deploy-safety
description: Verify the real base/deploy branch before merging a PR or claiming a merge will deploy something. Use whenever merging a pull request, opening a PR meant to ship to production, or telling the user a change is live/deploying.
---

# Deploy safety

Before merging any PR, or telling the user a merge "ships" or "deploys" something, verify the actual mechanics rather than assuming standard conventions hold.

## Why this exists

On a real repo, a PR was merged assuming it targeted the branch that triggers deploy. It actually landed on the repo's default branch (`dev`), which was *not* the deploy-triggering branch (`main` was, via a GitHub Actions workflow watching `push: branches: [main]`). The user was told production would update; it didn't, until a second explicit promotion PR was opened and merged. The root cause: `gh pr create` without `--base` silently targets whatever GitHub reports as the repository's default branch — which is not always `main`, and is not always the branch deploy actually watches.

## Checklist

1. **Find the actual deploy trigger** — read the CI/CD config directly (`.github/workflows/*.yml`, `.gitlab-ci.yml`, Vercel/Netlify project settings, etc.). Note exactly which branch(es) trigger a real deploy. Don't assume it's `main` — check.
2. **Find the repo's actual default branch** — `gh api repos/<owner>/<repo> --jq '.default_branch'` (don't trust a local branch named `main` existing; it may be an unrelated/stale local branch that doesn't match the GitHub remote's default).
3. **If they differ**, always pass an explicit `--base <branch>` to `gh pr create` — never rely on the default. State the target branch out loud before opening the PR.
4. **Before merging**, confirm the PR's actual base with `gh pr view <n> --json baseRefName` — don't assume it's what you intended when you opened it, especially if time has passed or someone else could have changed it.
5. **After merging**, if the merge was meant to deploy, verify a deploy actually started (`gh run list --workflow=<file> --limit 1`) rather than assuming the merge alone confirms it. Watch it to completion (`gh run watch <run-id> --exit-status`) before telling the user it shipped.
6. **State plainly, before merging**, what will happen: which branch, whether it triggers deploy, and get confirmation — merging into a deploy-triggering branch is a production-affecting action and deserves the same confirmation as any other hard-to-reverse action, even though it's technically revertable via `git revert` + push.
7. **After a deploy**, a brief window of 502/503/500 during process restart/cold-start is normal on most platforms (Azure App Service, etc.) — don't treat a transient error right after deploy as a regression without checking whether it's still happening a minute or two later.
