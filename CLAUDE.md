# CLAUDE.md

## Project

Homebrew tap that automatically builds and distributes pre-built Hell binaries for macOS.
Repository: `Portfoligno/homebrew-hell`. Install: `brew install Portfoligno/hell/hell`.

## Architecture

Fully automated pipeline:

1. Daily cron detects new `chrisdone/hell` tags (`check-upstream.yml`)
2. Builds macOS binaries for ARM64 and x86_64 (`build-and-release.yml`)
3. Creates GitHub Release with tarballed binaries
4. Updates Formula with new version, URLs, and SHA256 hashes (`update-formula.yml`)
5. Weekly smoke test confirms installation works (`verify-install.yml`)

The formula distributes pre-built binaries directly (no `bottle do` block, no `brew test-bot`).
The formula `version` field is the source of truth for tracking which upstream version we've built.

## Languages

- **Ruby**: `Formula/hell.rb` ONLY. Unavoidable -- Homebrew formulas are Ruby.
- **Hell**: `scripts/*.hell` -- all CI orchestration and multi-command logic.
- **YAML**: `.github/workflows/*.yml` and `.github/actions/*/action.yml` -- each `run:` step must be a SINGLE command. No chaining, piping, or multi-line scripts.
- **No shell scripts, no Python.**

## Bootstrap

The `setup-hell` composite action builds Hell from source for CI use. This is the ONE place where multi-step orchestration lives in YAML steps instead of Hell scripts. Pin the CI Hell version in `.hell-version`.

## Constraints

- YAML `run:` steps: one command per step. Multi-command logic goes in Hell scripts.
- Subprocess spawning in Hell scripts: typed argument arrays via `Process.proc`, never string concatenation.
- All git-tracked files end with trailing newline.
- Cache must be extensive: Hell binary, Cabal store.

## Key Files

- `.hell-version` -- pinned Hell tag for CI bootstrap
- `Formula/hell.rb` -- the formula (generated; do not edit by hand)
- `.github/actions/setup-hell/action.yml` -- bootstrap workaround
- `scripts/check-upstream.hell` -- upstream release detection
- `scripts/update-formula.hell` -- formula regeneration
- `scripts/verify-install.hell` -- post-install verification

## Secrets

- `DEPLOY_KEY` -- SSH deploy key for pushing branches that trigger workflows (GITHUB_TOKEN pushes do not trigger other workflows)
- `GITHUB_TOKEN` -- used for GitHub API calls and release creation (automatic)

## Development

- Test formula locally: `brew install --formula ./Formula/hell.rb`
- Run a script: `./scripts/check-upstream.hell`
- Manual build trigger: workflow_dispatch on `build-and-release.yml` with an upstream tag
