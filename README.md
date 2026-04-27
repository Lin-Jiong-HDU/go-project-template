# Go Project Template

A Go 1.24+ project template optimized for **AI-assisted development**. Ships with ready-to-use instruction files for Claude Code, GitHub Copilot, and Cursor — so AI tools understand your project conventions from day one.

## Why This Template?

When AI coding assistants don't understand your project's rules, they generate inconsistent code, break architectural boundaries, and introduce unwanted dependencies. This template solves that by embedding your conventions directly into AI instruction files:

- **Coding standards** — naming, error handling, context usage, logging
- **Architecture rules** — Clean Architecture layering for web projects, flat structure for CLI
- **Testing strategy** — table-driven tests, mock generation, coverage expectations
- **Hard boundaries** — explicit "NEVER" rules that prevent common AI mistakes
- **Git workflow** — Conventional Commits, branch naming, PR format

## Features

- Zero third-party dependencies — pure Go standard library skeleton
- AI instruction files for **3 major tools** (Claude Code, Copilot, Cursor)
- Two project modes: **CLI/Library** (flat) and **Web** (Clean Architecture / DDD)
- Go 1.24+ with modern idioms (generics over `interface{}`, `log/slog`, etc.)
- Makefile with common commands out of the box

## Quick Start

1. Click **"Use this template"** on GitHub to create a new repo
2. Update the module path in `go.mod`:
   ```
   module github.com/YOUR_USERNAME/YOUR_REPO
   ```
3. Choose your project type and start coding

## Project Types

### CLI / Library

Keep the flat structure as-is:

```
├── main.go
├── internal/
├── go.mod
└── Makefile
```

### Web (Clean Architecture)

Follow the structure described in `CLAUDE.md`:

```
├── cmd/
│   └── main.go              # Entry point
├── domain/                  # Entities, repository interfaces, usecase interfaces
│   ├── entity.go
│   ├── repository.go
│   └── usecase.go
├── usecase/                 # Business logic (implements domain.XxxUsecase)
│   └── xxx_usecase.go
├── repository/              # Data access (implements domain.XxxRepository)
│   └── xxx_repository.go
├── api/
│   ├── controller/          # HTTP handlers (calls domain usecase interfaces)
│   │   └── xxx_controller.go
│   ├── middleware/          # HTTP middleware
│   └── route/               # Route registration and dependency wiring
│       └── route.go
├── bootstrap/               # App initialization (config, DB setup)
├── internal/                # Shared utilities (tokenutil, etc.)
├── mocks/                   # Generated mocks (mockery)
├── go.mod
└── Makefile
```

Dependency direction is strictly enforced: `route → controller → usecase → repository → domain`, with `route` as the composition root.

## AI Instruction Files

| File | Tool | Status |
|------|------|--------|
| `CLAUDE.md` | [Claude Code](https://claude.ai/code) | Ready |
| `AGENTS.md` | [GitHub Copilot](https://github.com/features/copilot) | Ready |
| `.cursorrules` | [Cursor](https://cursor.sh) | Ready |
| `.github/copilot-instructions.md` | GitHub Copilot (cloud agent) | Ready |

`CLAUDE.md` is the source of truth; `AGENTS.md` and `.cursorrules` contain the same rules.

### What's in the AI Instructions?

- **AI Workflow Protocol** — five mandatory phases: Understand → Plan → Implement → Verify → Commit
- **Pipeline Gates** — four local gates (vet, build, test, lint) every AI change must clear before a PR
- **Debug Protocol** — structured self-correction loop; rules against suppressing errors or tests
- **Project type detection** — AI reads your structure and adapts its behavior
- **Go coding standards** — naming, error handling, context, concurrency, logging
- **Architecture conventions** — dependency direction, interface segregation, DI patterns
- **Testing strategy** — table-driven tests, mockery mocks, black-box testing
- **Git workflow** — Conventional Commits, branch naming, PR format
- **Hard boundaries** — 20+ explicit "NEVER" rules preventing AI from breaking your project
- **Recommended libraries** — curated list for when you need third-party packages

## CI/CD Pipeline

The `.github/workflows/ci.yml` workflow runs on every push and pull request to `main`:

| Stage | Command | Failure action |
|-------|---------|----------------|
| Tidy check | `go mod tidy && git diff --exit-code go.sum` | Fix go.sum drift |
| Vet | `go vet ./...` | Fix reported issues |
| Build | `go build ./...` | Fix compile errors |
| Test | `go test -race ./...` | Fix failing tests |
| Lint | `golangci-lint run` | Fix lint violations |

**AI agents must replicate these gates locally** (see Pipeline Gates in `AGENTS.md`) before opening a pull request. The CI pipeline acts as the final arbiter.

### Copilot Cloud Agent Setup

`.github/copilot-setup-steps.yml` pre-installs `mockery` and `golangci-lint` in Copilot's ephemeral environment so it can run all four pipeline gates without installing tools on every run.

## Makefile Commands

```bash
make build     # Build the project
make test      # Run tests with coverage
make lint      # Run linter (requires golangci-lint)
make run       # Run the application
make tidy      # Tidy go modules
make clean     # Clean build artifacts
```

## Customization

- Edit `CLAUDE.md` to adjust rules, then sync to `AGENTS.md`, `.cursorrules`, and `.github/copilot-instructions.md`
- Add your preferred libraries from the recommended list in the AI instructions
- Extend the architecture sections to match your specific domain patterns
- Adjust pipeline gates in `.github/workflows/ci.yml` to add project-specific checks (e.g., integration tests, security scanning)

## License

MIT
