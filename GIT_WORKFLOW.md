# Banking CDP — Git workflow and pull request policy

> **Applies to everyone, including coding agents.** Read `AGENTS.md` first. This policy is for the Banking CDP repository; it has no module or individual ownership assignments.

## 1. Branches

| Branch | Purpose | How changes arrive |
|---|---|---|
| `main` | Latest **reviewed, stable** version | Reviewed release PR from `dev` |
| `dev` | Latest integrated team work (may be ahead of `main`) | Reviewed PR from a task branch |
| `feature/<task>` | New functionality or schema design | Branch from updated `dev` |
| `fix/<issue>` | Bug fix | Branch from updated `dev` |
| `docs/<topic>` | Research, architecture, docs | Branch from updated `dev` |
| `chore/<task>` | Maintenance/configuration | Branch from updated `dev` |

**Do not commit or push directly to `main` or `dev`.** Do not force-push or delete protected branches. One focused task per PR.

## 2. Before starting work

```bash
git status --short
git branch --show-current
git fetch origin
git switch dev
git pull --ff-only origin dev
git switch -c docs/short-task-name
```

Choose the appropriate prefix for the work. If there are uncommitted changes, a divergent branch, or other unexpected state, do not reset, clean, or overwrite anything automatically: resolve with the contributor first.

## 3. Before pushing

- Check the diff and keep unrelated edits out of the commit.
- Do not commit `.env`, credentials, access tokens, personal/customer data, generated secrets or large sample datasets containing real records.
- Use synthetic examples for banking, account and customer information.
- For **documentation/data-model work**: verify Markdown links and formatting, ensure naming and relationships are consistent, and review assumptions versus confirmed decisions.
- For **code changes**: run the actual applicable linters, type checks, tests and build commands documented in the repo. Add tests for changed behavior and failure cases.
- For **database migrations**: review schema changes with the team and run the project's actual migration checks, once implemented.
- Do not invent CI commands: if a check is unavailable, note that clearly in the PR.

```bash
git diff --check
git status --short
git diff
git add <relevant-files>
git commit -m "docs: describe the change"
git push -u origin docs/short-task-name
```

Use conventional, readable commit messages such as `docs: add event schema`, `feat: add event parser`, `fix: handle duplicate events`, `chore: update workflow`.

## 4. Pull request process

1. Open a PR **from your task branch into `dev`** (not directly to `main`).
2. Include: a short summary, related issue/task (if any), changed files or interfaces, checks run and results, assumptions/risks, and screenshots/examples when useful.
3. Wait for required review and applicable CI checks. Address feedback and resolve conversations.
4. Merge only after the repository's branch rules are satisfied. Do not self-approve as a substitute for independent review.
5. For a stable release, open a separate PR **from `dev` into `main`**.

### Recommended branch protections

| Protection | `dev` | `main` |
|---|---|---|
| PR required | Yes | Yes |
| Approvals | At least 1 | At least 2 |
| Review conversations resolved | Yes | Yes |
| Block force pushes | Yes | Yes |
| Prevent deletion | Yes | Yes |
| Required status checks | When configured | When configured |

Enforcement depends on repository settings and GitHub plan. The ruleset is the actual source of enforcement; this document explains the expected process. Do not bypass a protection because GitHub allows a direct action.

## 5. Current development stage

The team is presently aligning on CRM/account-module data models and CDP architecture. **Do not begin implementation or create a master database solely because this workflow exists.** Treat proposed technologies (including PostHog, NiFi, pub/sub, PostgreSQL, FastAPI, Azurite/S3) as subject to the latest confirmed team decisions and requirements.

For now, documentation and schema-design PRs can be checked with lightweight validation and human review. Add stronger CI checks when working code, migration tooling, Docker services or frontend projects are actually introduced.

## 6. Rules for coding agents

- Read `AGENTS.md`, then this file, **before planning or changing the repository**.
- Check the current state and avoid changing unrelated work.
- Follow existing names and interfaces; ask before altering shared contracts or introducing new services.
- Never bypass reviews, CI checks, branch protection or secret handling.
- Never claim a PR was merged or tests passed without actually checking.
- At the end, report the branch, changed files, checks performed, outstanding questions and requested PR target.
