---
description: Safe merge-to-main workflow with pre-merge checks, conventional commits, doc updates, and post-deploy verification. Use when the user asks to merge, ship, land, or release changes to the main branch.
---

# Merge to Main Workflow

A fast, safe workflow for landing changes on `main`. Operating principles: **scope once** (detect tooling and the change set in a single pass, then reuse it), **fail fast** (cheap checks first), **parallelize** (independent checks run concurrently), **skip what can't be affected** (don't run checks the change set can't break), and **one confirmation gate** (batch everything the user must approve into a single question).

## Setup

- **Git auth**: Use GitHub CLI (`gh auth setup-git`) when available — avoid manual PAT or SSH configuration.

## Phase 1 — Scope (one pass, drives everything after)

1. **Branch from fresh remote main** without touching the local `main` checkout: `git fetch origin && git switch -c <type>/<slug> origin/main`. Use a conventional prefix (`feat/`, `fix/`, `docs/`, `chore/`, `refactor/`, `test/`). Always a fresh branch — never reuse one.
2. **Detect tooling once.** In one pass, read `package.json`, `pyproject.toml`, `Makefile`, `Dockerfile`, `docker-compose.yml`, `.github/workflows/`, and any docs app (`apps/docs`, `website/`, Docusaurus, VitePress, OpenAPI spec, Storybook). Note the exact lint/test/build/docs-build commands. Do not re-derive these later.
3. **Compute the change set once**: `git diff --name-only origin/main...HEAD` (plus planned edits). From it, decide up front which checks apply:
   - Source code changed → lint + tests + build (+ container build if a Dockerfile exists and code/deps changed).
   - Behavior that docs describe changed (APIs, CLI, config/env, UI flows, permissions, error codes) → matching doc pages must be updated in this branch.
   - Docs-only change → skip tests/build/container; run only the docs build and link checks if the project has them.
   - Lockfile/dependency-only change → tests + build + container; lint of unchanged source adds nothing.

## Phase 2 — Update docs (in-branch, never a follow-up)

- **Repo docs:** revise `CLAUDE.md`, `README.md`, and relevant `docs/` files; remove outdated content, consolidate duplicates.
- **In-app or routed docs (if detected in Phase 1):** update the matching pages in the same branch. Stale live docs are a merge blocker, same as a broken test.

## Phase 3 — Verify (fail fast, in parallel)

- **Order cheap → expensive** so failures surface in seconds, not minutes: lint first, then tests, then build. Common commands (use what Phase 1 detected):
  - Node: `npm run lint`, `npm test`, `npm run build`
  - Python: `ruff check`, `pytest`, `python -m build`
  - Go: `go vet ./...`, `go test ./...`, `go build ./...`
  - Rust: `cargo clippy`, `cargo test`, `cargo build --release`
- **Run independent checks concurrently** — lint, tests, and the docs build don't depend on each other; launch them in parallel rather than serially.
- **Container build (if applicable):** start `docker compose build` / `docker build .` **in the background** and continue to Phase 4 while it runs; require it green before merging. If the Dockerfile runs the same compile/test steps internally, skip the redundant local build and let the image build serve as the build check.
- Only run checks the Phase 1 change set can affect — a green check on untouched code is wasted time, not safety.

## Phase 4 — Commit & confirm (single gate)

- **Stage docs with the code they describe** in the same commit — `CLAUDE.md`, `.claude/`, `docs/`, doc-site sources, OpenAPI specs, Storybook stories. Check the staged list against the Phase 1 change set before committing.
- **Conventional Commits:** `<type>(<optional scope>): <subject>` — types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `build`, `ci`. Subject under 72 chars; put the *why* in the body when it's not obvious.
- **One confirmation before merge.** Present a single summary — branch, commits, changed files, check results (including the background container build) — and ask once. Don't drip-feed questions across the workflow; batch anything needing user input into this gate.
- **Pre-authorized skip.** If the user's invocation explicitly waived confirmation — "merge without asking", "no confirmation", "ship now" — post the same summary but merge immediately instead of asking. A bare "merge to main" or "ship this" is intent, not authorization: it still gets the gate. Pre-authorization only skips the *ask* — never checks, project `CLAUDE.md` overrides, or pre-merge blockers — and it lapses on any failure or surprise (failed/caveated check, unexpected files in the diff, a blocker that applies): stop and ask regardless.

## Phase 5 — Merge, clean up, monitor

- After approval: merge to `main`, push, delete the branch locally and on the remote.
- **If `main` auto-deploys** (GitHub Actions, Vercel, Fly, etc.): watch the run identified in Phase 1 and confirm health checks pass. Prefer watching in the background (`gh run watch` or polling) over blocking idle.
- **Fix forward immediately.** If runtime errors appear post-deploy, create a `fix/` branch and ship a correction right away — never leave `main` broken.

## Notes

- If the project defines its own merge workflow in `CLAUDE.md` or `docs/`, that takes precedence over this generic flow.
- For protected branches or trunk-based workflows that require PRs, replace the merge in Phase 5 with: open a PR via `gh pr create`, wait for CI + review, then merge via `gh pr merge`. CI re-runs the same checks — don't duplicate slow suites locally if CI is the required gate; run only the fast local checks (lint, affected tests) before pushing.
- In-app docs are part of the product surface; shipping code without updating them is incomplete work, same as skipping tests for touched code paths.
