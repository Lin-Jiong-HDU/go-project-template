# Copilot Repository Instructions

## Project Overview

Go 1.24+ project template with zero third-party dependencies. Ships as either a **CLI/Library** (flat structure) or a **Web** project (Clean Architecture). Full coding conventions are in `AGENTS.md` at the repo root.

## Build & Validation

Always run these commands in order. Every command must pass before moving to the next. Fix failures before proceeding — never skip a step.

```bash
# 1. Static analysis (must produce zero output)
go vet ./...

# 2. Compile check (must produce zero output)
go build ./...

# 3. Tests with race detector (all must pass)
go test -race ./...

# 4. Lint (no new violations)
golangci-lint run
```

`make` shortcuts:

```bash
make build   # go build
make test    # go test ./... -v -cover
make lint    # golangci-lint run
make tidy    # go mod tidy
make clean   # remove bin/
```

## Workflow Rules

1. **Read before writing** — always read the existing code in the area you are changing.
2. **Smallest change** — implement only what the task requires; do not refactor unrelated code.
3. **All gates green** — every gate above must pass before opening a PR.
4. **Fix, never suppress** — do not comment out tests, add `//nolint`, or use `_ = err` to hide errors.
5. **Conventional Commits** — `type(scope): subject` (imperative, ≤72 chars, no period).

## Project Layout

```
.cursorrules          # Cursor AI instructions (same content as AGENTS.md)
AGENTS.md             # GitHub Copilot / OpenAI Agents instructions
CLAUDE.md             # Claude Code instructions (source of truth)
Makefile              # build/test/lint shortcuts
go.mod                # module: github.com/Lin-Jiong-HDU/go-project-template
main.go               # CLI entry point (current project type: CLI/Library)
internal/             # internal packages
```

## Architecture (Web projects)

Dependency direction: `route → controller → usecase → repository → domain`

- `domain` — entities + interfaces only; zero internal imports
- `usecase` — business logic; imports `domain` only; applies `context.WithTimeout`
- `repository` — data access; imports `domain` + drivers
- `controller` — HTTP handlers; imports `domain` interfaces only
- `route` — wires everything together (composition root)

**Never** let an inner layer import an outer layer.

## Key Constraints

- No `interface{}` — use generics (Go 1.24+)
- No `panic` for expected errors — return `error`
- No `time.Sleep` in tests — use channels or sync primitives
- No hardcoded secrets or passwords
- No direct push to `main`
- Third-party dependencies require CVE check + purpose documented in commit message
