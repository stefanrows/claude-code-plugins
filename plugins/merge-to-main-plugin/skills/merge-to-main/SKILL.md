---
description: Safe merge-to-main workflow with pre-merge checks, conventional commits, doc updates, and post-deploy verification. Use when the user asks to merge, ship, land, or release changes to the main branch.
---

# Merge to Main Workflow

A repeatable, safe workflow for landing changes on `main`. Adapt each step to the project's actual tooling — skip Docker if there's no `Dockerfile`, use whatever lint/test/build commands the project defines, etc. Detect what applies by reading `package.json`, `pyproject.toml`, `Makefile`, `Dockerfile`, `.github/workflows/`, or equivalent before assuming.

## Setup

- **Git auth**: Use GitHub CLI (`gh auth setup-git`) when available — avoid manual PAT or SSH configuration.

## Workflow

1. **Start fresh.** Pull latest `main` and create a new branch with a conventional prefix (`feat/`, `fix/`, `docs/`, `chore/`, `refactor/`, `test/`). Always create a fresh branch — never reuse an existing one for a new change.

2. **Update docs.** Revise `CLAUDE.md`, `README.md`, and any relevant files under `docs/`. Reflect the changes being shipped, remove anything outdated, and consolidate duplicates.

3. **Verify the build.** Run the project's lint, test, and build commands. Detect them from the project files; common examples:
   - Node: `npm run lint && npm test && npm run build`
   - Python: `ruff check && pytest && python -m build`
   - Go: `go vet ./... && go test ./... && go build ./...`
   - Rust: `cargo clippy && cargo test && cargo build --release`

4. **Verify container build (if applicable).** If the repo has a `Dockerfile` or `docker-compose.yml`, run `docker compose build` or `docker build .` to catch container-specific issues that local builds miss.

5. **Always stage docs alongside code.** In every commit, include `CLAUDE.md`, `.claude/`, and `docs/` if they changed. Run `git diff --name-only` before committing to confirm nothing is left out.

6. **Commit with Conventional Commits.** Format: `<type>(<optional scope>): <subject>` — types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `build`, `ci`. Keep the subject under 72 chars; put the *why* in the body when it's not obvious.

7. **Confirm before merging.** Always ask the user for explicit confirmation before merging to `main`. Summarize what's being merged so they can sanity-check.

8. **Merge and clean up.** After approval: merge the branch to `main`, push, then delete the branch locally and on the remote.

9. **Monitor the deploy (if applicable).** If `main` auto-deploys (GitHub Actions, Vercel, Fly, etc.), watch the run and confirm health checks pass. Inspect `.github/workflows/` to find the relevant workflow.

10. **Fix forward immediately.** If runtime errors appear post-deploy, create a `fix/` branch and ship a correction right away — never leave `main` broken.

## Notes

- If the project defines its own merge workflow in `CLAUDE.md` or `docs/`, that takes precedence over this generic flow.
- For protected branches or trunk-based workflows that require PRs, replace step 8 with: open a PR via `gh pr create`, wait for CI + review, then merge via `gh pr merge`.
