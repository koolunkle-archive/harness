---
name: deploy-check
description: Runs a pre-deploy readiness check on the current project — verifies git status, environment/config, and build/test/lint all pass before shipping. Use this whenever the user asks to deploy, ship, release, or push to production, or asks "is this ready to deploy/ship?" or "can we deploy this?" even if they don't name the skill explicitly. Works across any project type (Node, Python, Go, Rust, etc.) by auto-detecting the right commands.
---

## What this does

Before a deploy, three classes of things silently break releases: uncommitted or unpushed changes, missing/misconfigured environment variables, and code that doesn't actually build/test/lint clean. This skill runs all three checks and reports a checklist so the user can see exactly what's blocking (or clearing) the deploy — it does not perform the deploy itself.

## Step 1: Detect the project

Look for these signals to figure out what commands apply (check in this order, use whichever exist):

- **CLAUDE.md** in the project root — if it documents specific build/test/lint/deploy commands, prefer those over guessing.
- **package.json** — read `scripts` for `build`, `test`, `lint`, `typecheck`. Run via the project's package manager (check for `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json` to pick `pnpm`/`yarn`/`npm`).
- **pyproject.toml** / **requirements.txt** — look for `pytest`, `ruff`/`flake8`, `mypy`, or a `tox.ini`/`noxfile.py` with defined sessions.
- **Makefile** — targets named `build`, `test`, `lint`, `check` are common; `make -n <target>` shows what it would run without executing.
- **go.mod** — `go build ./...`, `go vet ./...`, `go test ./...`.
- **Cargo.toml** — `cargo build`, `cargo test`, `cargo clippy`.

If none of these are found, say so plainly instead of guessing at a command that might not exist — a wrong command reads as a false failure.

## Step 2: Git status

Run `git status --porcelain`, `git branch --show-current`, and `git status -sb` (or `git rev-list --left-right --count HEAD...@{upstream}` if it has an upstream) to check:

- Any uncommitted or untracked changes (these won't be in the deploy unless committed).
- Whether the current branch is the one that actually gets deployed (e.g. not accidentally on a feature branch when `main`/`master` is expected — infer this from context/CLAUDE.md rather than assuming).
- Unpushed local commits, or the local branch being behind its remote.

## Step 3: Environment / config

- If both `.env.example` (or `.env.sample`) and `.env` exist, diff their variable names and flag any missing from `.env` — a var that's documented but unset is a common deploy-day surprise.
- If only `.env.example` exists and no `.env`, flag that config likely needs to be set up on the target environment.
- Check `.gitignore` actually covers `.env` and other local secret files — if it doesn't, flag it as a real risk (secrets could get committed), not just a style nit.
- Skim the current diff (`git diff HEAD`) for anything that looks like a hardcoded credential, API key, or connection string being added.

## Step 4: Build / test / lint

Run whatever commands Step 1 identified. Run them in the order build → lint → test unless the project's own tooling implies otherwise (e.g. a Makefile target that chains them itself). Capture the actual failure output for anything that fails — don't just report pass/fail, include enough of the error to act on.

Only run commands that are read-only or produce local artifacts (building, testing, linting). Never run an actual deploy/publish/push-to-prod command as part of this check, even if one exists (e.g. `npm publish`, `terraform apply`, `make deploy`) — this skill verifies readiness, it doesn't ship.

## Step 5: Report

Present a single checklist-style summary, grouped by category, each item marked ✅ / ⚠️ / ❌. For anything not ✅, give a one-line reason and, where obvious, how to fix it. End with one overall verdict line:

```
## Deploy Check

### Git
✅ No uncommitted changes
✅ On `main`, up to date with origin

### Environment / Config
⚠️ `.env` is missing `STRIPE_SECRET_KEY` (present in `.env.example`)

### Build / Test / Lint
✅ Build passed
❌ Test failed — 2 failures in `tests/auth.test.ts` (see below)
✅ Lint passed

### Verdict
❌ Not ready to deploy — fix the failing tests and missing env var above.
```

Use ✅ ready / ⚠️ ready with caveats / ❌ not ready as the three possible verdicts. If literally everything passed, keep the report short rather than padding it out.
