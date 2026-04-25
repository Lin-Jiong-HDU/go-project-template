# Go Project Template

A Go 1.24+ project template optimized for AI-assisted development.

## Features

- Zero third-party dependencies
- AI instruction files for Claude Code, GitHub Copilot, and Cursor
- Supports CLI/library (flat) and Web (Clean Architecture) project types
- Go coding standards and best practices baked into AI instructions

## Usage

1. Click **"Use this template"** on GitHub to create a new repository
2. Replace `github.com/example/project` in `go.mod` with your actual module path
3. Choose your project type:
   - **CLI/Library**: keep the flat structure as-is
   - **Web project**: follow the Clean Architecture structure described in `CLAUDE.md`

## Project Types

### CLI / Library

```
├── main.go
├── internal/
├── go.mod
└── Makefile
```

### Web (Clean Architecture)

```
├── cmd/main.go
├── internal/
│   ├── domain/
│   ├── repository/
│   ├── usecase/
│   ├── controller/
│   ├── route/
│   ├── middleware/
│   ├── bootstrap/
│   ├── infra/
│   └── mocks/
├── go.mod
└── Makefile
```

## AI Instruction Files

| File | Tool |
|------|------|
| `CLAUDE.md` | Claude Code |
| `AGENTS.md` | GitHub Copilot |
| `.cursorrules` | Cursor |

## Makefile Commands

```bash
make build     # Build the project
make test      # Run tests with coverage
make lint      # Run linter
make run       # Run the application
make tidy      # Tidy go modules
make clean     # Clean build artifacts
```

## License

MIT
