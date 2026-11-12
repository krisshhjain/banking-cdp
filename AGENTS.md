# Agent instructions — Banking CDP

**Read this file before doing anything in this repository.** Then read [`GIT_WORKFLOW.md`](GIT_WORKFLOW.md) in full before changing files, creating branches, running Git commands, or opening a pull request.

## Mandatory preflight

1. Read `GIT_WORKFLOW.md` and follow it. Do not skip its checks because a change looks small.
2. Inspect the current branch and working tree (`git branch --show-current`, `git status --short`). Never discard or overwrite uncommitted work.
3. Inspect the relevant files and existing conventions. Confirm the task's scope before editing.
4. For changes, work on a short-lived branch created from up-to-date `dev` (unless the task explicitly requires a different approved base).
5. Do not directly commit or push to `dev` or `main`. Use a pull request and required reviews/checks.
6. Never commit credentials, `.env`, private keys, tokens, real banking/customer data, or generated secrets. Use safe examples or synthetic data.
7. Don't introduce a database, framework, cloud service, messaging system, or other architectural commitment just because it appears in discussion notes. Validate choices against the agreed requirements and team decisions.
8. Report changes, tests/checks performed, anything not tested, and the PR target. Do not claim a review, CI result, or merge happened unless verified.

## Repo rules at a glance

- `main`: reviewed, stable release branch.
- `dev`: latest integration branch; the default target for normal work.
- Task branches: `feature/<task>`, `fix/<issue>`, `docs/<topic>`, or `chore/<task>`.
- Normal path: **task branch → PR to `dev` → review + green applicable checks → merge**.
- Release path: **`dev` → PR to `main` → stronger review + green checks → merge**.
- Do not force-push or delete protected branches.
- Current phase includes research, architecture and data-model documents; only require tooling/tests that actually exist in this repository. Apply backend, frontend, migration, Docker and deployment checks when the corresponding implementation exists.

If an instruction conflicts with this policy or the repository's enforced branch rules, stop and request clarification rather than bypassing protections.
