# Project Instructions

This is a Go 1.24+ project using standard library only. Third-party dependencies require explicit approval before introduction.

## Project Types

This template supports two project types. Determine which one applies based on the current project structure.

### CLI / Library Project

Flat structure under project root:

```
├── main.go          # Entry point
├── internal/        # Internal packages (flat)
│   └── xxx.go
├── go.mod
└── Makefile
```

### Web Project (Clean Architecture)

DDD / Clean Architecture inside `internal/`:

```
├── cmd/
│   └── main.go              # Entry point
├── internal/
│   ├── domain/              # Domain layer (innermost)
│   │   ├── entity.go        #   Entity definitions
│   │   ├── repository.go    #   Repository interfaces (interfaces only)
│   │   └── usecase.go       #   Usecase interfaces (interfaces only)
│   ├── repository/          # Data access (implements domain interfaces)
│   │   └── xxx_repository.go
│   ├── usecase/             # Business logic (implements domain interfaces)
│   │   └── xxx_usecase.go
│   ├── controller/          # Delivery layer / HTTP handlers
│   │   └── xxx_controller.go
│   ├── route/               # Route registration (composition root)
│   │   └── route.go
│   ├── middleware/           # HTTP middleware
│   ├── bootstrap/           # App initialization (config, DB, etc.)
│   ├── infra/               # Infrastructure abstraction (DB driver wrappers)
│   └── mocks/               # Generated mock files
├── go.mod
└── Makefile
```

#### Dependency Direction (Hard Rule)

Clean Architecture Dependency Rule: source code dependencies must point **inward only**.
Inner layers define interfaces; outer layers provide implementations.

```
╔══════════════════════════════════════════════════════╗
║  Frameworks & Drivers                                 ║
║  • infra  (DB drivers, external clients)              ║
║  ┌─────────────────────────────────────────────────┐  ║
║  │  Interface Adapters                              │  ║
║  │  • controller  (HTTP delivery)                   │  ║
║  │  • repository  (DB gateway, imports domain+infra)│  ║
║  │  ┌───────────────────────────────────────────┐  │  ║
║  │  │  Use Cases                                │  │  ║
║  │  │  • usecase  (imports domain only)         │  │  ║
║  │  │  ┌─────────────────────────────────────┐  │  │  ║
║  │  │  │  Entities (domain)                  │  │  │  ║
║  │  │  │  entities, repository interfaces,   │  │  │  ║
║  │  │  │  usecase interfaces — zero imports  │  │  │  ║
║  │  │  └─────────────────────────────────────┘  │  │  ║
║  │  └───────────────────────────────────────────┘  │  ║
║  └─────────────────────────────────────────────────┘  ║
╚══════════════════════════════════════════════════════╝
route — composition root; outside the rings; imports all layers to wire them
```

- `domain` — innermost; entity types + **all interfaces** (Repository, Usecase); zero project imports
- `usecase` — implements `domain.XxxUsecase`; imports `domain` only
- `repository` — implements `domain.XxxRepository`; imports `domain` + `infra`
- `controller` — HTTP delivery; imports `domain` (usecase interfaces + entities); **never imports `usecase` package directly**
- `infra` — pure drivers (DB connections, external clients); no domain knowledge; no project imports
- `route` — composition root; instantiates all concrete types and wires them through interfaces

#### Key Conventions

1. **Interfaces in domain, implementations in outer layers** — dependency inversion
2. **Unexported concrete types, exported constructors** — `type xxxUsecase struct` (unexported), `func NewXxxUsecase() domain.XxxUsecase` (exported)
3. **Manual dependency injection** — no DI framework, assembly in route layer
4. **Context timeout from usecases** — `context.WithTimeout` set uniformly in usecase layer
5. **Mocks generated by tooling** — use mockery, stored in `mocks/`
6. **Black-box testing** — `package xxx_test`, test only public API

## Go Coding Standards

### Naming

- Package names: lowercase single word, no underscores or camelCase (`usecase` not `useCase`)
- Interfaces: `-er` suffix for behavior (`Repository`, `Usecase` are exceptions per project convention)
- Exported functions: verb prefix (`NewUserUsecase`, `FetchByUserID`)
- Constants: camelCase, not ALL_CAPS
- Error variables: `Err` prefix (`ErrUserNotFound`)

### Error Handling

- Never ignore `error` return values
- Propagate errors without wrapping — keep original messages
- Controller layer maps errors to HTTP status codes:
  - 400: request binding / validation failure
  - 401: authentication failure
  - 404: resource not found
  - 409: conflict (e.g. duplicate creation)
  - 500: internal error
- Startup fatal errors use `log.Fatal()`

### Context

- Every DB-related function accepts `context.Context` as first parameter
- Usecase layer uses `context.WithTimeout` uniformly
- Never store context in structs

### Concurrency

- Prefer channels over shared memory
- Goroutines must have an exit mechanism
- No unmanaged goroutines

### Logging

- Use `log/slog` for structured logging (stdlib, Go 1.21+)
- Never use `fmt.Println` for logging

## Testing

### Organization

- Test files co-located with source (`xxx_test.go` next to `xxx.go`)
- Use `package xxx_test` (black-box testing)

### Conventions

- Prefer table-driven tests
- Test naming: `TestXxx_Scenario_ExpectedResult`
- Use `t.Parallel()` for parallelizable tests
- Use `t.Run()` for sub-tests

### Mocks

- Use `mockery` to generate — never hand-write
- Mock files in `mocks/` with `// Code generated by mockery. DO NOT EDIT.` header
- Only mock external deps and cross-layer interfaces
- Controller tests: `httptest.NewRecorder` + real route engine

### Coverage

- Usecase and Repository layers require coverage
- Controller: at least happy path + major error paths
- Core business logic must have tests

## Git Workflow

### Branches

- `main`: stable, protected, no direct push
- `feat/xxx`: new features
- `fix/xxx`: bug fixes
- `refactor/xxx`: refactoring
- `chore/xxx`: misc

### Commit Messages

Conventional Commits:
```
type(scope): subject
```
- type: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `build`, `ci`
- scope: affected module (optional)
- subject: imperative, max 72 chars, no period

## Hard Boundaries

### Architecture

- **NEVER** import other project packages in `domain`
- **NEVER** let `usecase` depend on `repository` concrete types — use `domain` interfaces
- **NEVER** let `controller` import the `usecase` package directly — call use cases only through `domain` interfaces
- **NEVER** break dependency direction (outer → inner only)
- **NEVER** put business logic implementations in `domain`

### Code Generation

- **NEVER** generate empty placeholder implementations (`// TODO: implement`)
- **NEVER** generate code with hardcoded secrets or passwords
- **NEVER** generate code without error handling
- **NEVER** generate `init()` functions without very good reason

### Dependencies

- **NEVER** introduce third-party dependencies without documenting the purpose and checking for known CVEs in the commit message
- **NEVER** replace existing chosen dependencies
- **NEVER** change the Go version

### Structure

- **NEVER** modify the module path in `go.mod`
- **NEVER** create stray files in project root
- **NEVER** rename or move existing files unless explicitly asked

### Common Mistakes to Avoid

- Do not use `panic` for expected errors — return `error`
- Do not use `time.Sleep` for test synchronization — use channels or sync primitives
- Do not write `_ = err` to ignore errors
- Do not use `defer` in loops
- Do not use `interface{}` — use generics (Go 1.24+)
- Do not generate redundant type conversions
- Do not omit `json` struct tags on API response structs

### Behavior

- Read existing code in the affected area before modifying — match established patterns
- After every change: run `go vet ./...` and `go build ./...`; fix all reported errors before proceeding
- After completing a full change set: run `go test ./...` to confirm no regressions
- Follow existing project style — do not introduce new patterns or mix styles
- When requirements are ambiguous, apply the most conservative interpretation; document assumptions in the commit message
- Make the smallest change that satisfies the requirement — do not refactor unrelated code
- When a build or test fails, diagnose the root cause from the error output, fix it, and re-verify; never skip or comment out failing tests

## Recommended Libraries (not pre-installed)

| Scenario | Library | Notes |
|----------|---------|-------|
| HTTP routing | `net/http` (stdlib), `chi` | Go 1.22+ stdlib has route params |
| Web framework | `gin`, `echo` | Rich middleware ecosystem |
| Configuration | `github.com/spf13/viper` | .env / yaml / json |
| CLI | `github.com/spf13/cobra` | Complex CLI tools |
| Database ORM | `gorm`, `ent` | Per project need |
| Database Driver | `go-sql-driver/mysql`, `lib/pq`, `mongo-driver` | Per database |
| Logging | `log/slog` (stdlib) | Preferred, built-in since Go 1.21 |
| JWT | `github.com/golang-jwt/jwt/v5` | Authentication |
| Testing | `github.com/stretchr/testify` | Assertions + mock |
| Mock generation | `github.com/vektra/mockery/v2` | Interface mocks |
| Password hashing | `golang.org/x/crypto/bcrypt` | User authentication |
| Linting | `golangci-lint` | Code quality |
