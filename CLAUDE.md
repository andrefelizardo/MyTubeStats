# CLAUDE.md — Project Intelligence

## Project Overview

This is a SaaS/Web platform built with a Go REST API backend and a Vite + React frontend, backed by PostgreSQL.

## Tech Stack

### Backend (Go)

- **Router**: Chi (`github.com/go-chi/chi/v5`) with CORS middleware
- **Database**: PostgreSQL with `pgx` driver and `sqlc` for type-safe queries
- **Migrations**: `golang-migrate/migrate`
- **Auth**: JWT-based authentication with refresh tokens
- **Validation**: Struct tags with `go-playground/validator`
- **Logging**: Structured logging with `slog`
- **Config**: Environment variables via `.env` files

### Frontend (React)

- **Build Tool**: Vite
- **Framework**: React 18+ with TypeScript
- **Routing**: React Router v6
- **State Management**: Zustand or React Context (keep it simple)
- **HTTP Client**: Axios with interceptors for auth
- **Styling**: Tailwind CSS
- **Forms**: React Hook Form + Zod validation

### Infrastructure

- **Containerization**: Docker + Docker Compose for local dev
- **CI/CD**: GitHub Actions
- **Database**: PostgreSQL 16

## Project Structure

```
.
├── backend/
│   ├── cmd/
│   │   └── server/          # Application entrypoint
│   │       └── main.go
│   ├── internal/
│   │   ├── config/          # App configuration
│   │   ├── handler/         # HTTP handlers (grouped by domain)
│   │   ├── middleware/       # Custom middleware (auth, logging, etc.)
│   │   ├── model/           # Domain models / DTOs
│   │   ├── repository/      # Database access layer
│   │   ├── service/         # Business logic layer
│   │   └── router/          # Route definitions
│   ├── migrations/          # SQL migration files
│   ├── sqlc/                # sqlc configuration and generated code
│   ├── go.mod
│   └── go.sum
├── frontend/
│   ├── src/
│   │   ├── api/             # API client and endpoints
│   │   ├── components/      # Reusable UI components
│   │   ├── features/        # Feature-based modules
│   │   ├── hooks/           # Custom React hooks
│   │   ├── layouts/         # Page layouts
│   │   ├── pages/           # Route pages
│   │   ├── stores/          # State management
│   │   ├── types/           # TypeScript type definitions
│   │   └── utils/           # Utility functions
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   └── vite.config.ts
├── docker-compose.yml
├── Dockerfile.backend
├── Dockerfile.frontend
├── .github/workflows/
├── .env.example
└── CLAUDE.md                # (this file)
```

## Code Conventions

### Go Backend

- Follow standard Go project layout (`cmd/`, `internal/`)
- Use `internal/` to prevent external imports of private packages
- Handlers receive dependencies via struct injection, not globals
- All errors must be wrapped with context: `fmt.Errorf("failed to create user: %w", err)`
- Use table-driven tests: `func TestXxx(t *testing.T) { tests := []struct{...}{} }`
- HTTP responses use a consistent JSON envelope: `{ "data": ..., "error": ..., "meta": ... }`
- Database queries go through the repository layer, never called directly from handlers
- Use `context.Context` propagation for all database and service calls

### React Frontend

- Components use functional style with hooks (no class components)
- File naming: `PascalCase.tsx` for components, `camelCase.ts` for utilities
- Co-locate tests next to source files: `Button.tsx` → `Button.test.tsx`
- API calls are centralized in `src/api/` — never call `axios` directly from components
- Use TypeScript strict mode — avoid `any` type
- Custom hooks prefixed with `use`: `useAuth.ts`, `useDebounce.ts`

### Database

- Migration files follow the pattern: `YYYYMMDDHHMMSS_description.up.sql` / `.down.sql`
- Every migration must have both `up` and `down` files
- Use snake_case for table and column names
- All tables must have `id`, `created_at`, and `updated_at` columns
- Foreign keys must have explicit `ON DELETE` behavior

### Git

- Branch naming: `feature/short-description`, `fix/short-description`, `chore/short-description`
- Commit messages follow Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`
- PRs require passing CI checks before merge
- Squash merge to main

## Common Commands

```bash
# Backend
cd backend && go run ./cmd/server          # Run API server
cd backend && go test ./...                 # Run all tests
cd backend && go test -race ./...           # Run tests with race detector
cd backend && go vet ./...                  # Lint

# Frontend
cd frontend && npm run dev                  # Dev server
cd frontend && npm run build                # Production build
cd frontend && npm run test                 # Run tests
cd frontend && npm run lint                 # Lint

# Docker
docker compose up -d                        # Start all services
docker compose down                         # Stop all services
docker compose logs -f backend              # Tail backend logs

# Database
migrate -path backend/migrations -database "$DATABASE_URL" up    # Run migrations
migrate -path backend/migrations -database "$DATABASE_URL" down 1 # Rollback one
```

## Environment Variables

Backend reads from `.env` at project root. Required variables:

| Variable       | Description                  | Example                                                    |
| -------------- | ---------------------------- | ---------------------------------------------------------- |
| `DATABASE_URL` | PostgreSQL connection string | `postgres://user:pass@localhost:5432/saas?sslmode=disable` |
| `PORT`         | API server port              | `8000`                                                     |
| `JWT_SECRET`   | Secret for signing JWTs      | `super-secret-key`                                         |
| `JWT_EXPIRY`   | Token expiration duration    | `24h`                                                      |
| `CORS_ORIGINS` | Allowed frontend origins     | `http://localhost:5173`                                    |
| `LOG_LEVEL`    | Logging level                | `debug`                                                    |

## Important Patterns

### Error Handling (Backend)

Handlers return errors to a central error middleware that maps them to HTTP status codes. Use domain-specific error types defined in `internal/model/errors.go`.

### Authentication Flow

1. User registers/logs in → backend returns JWT access + refresh tokens
2. Frontend stores tokens and attaches access token via Axios interceptor
3. Expired access token → interceptor uses refresh token to get a new one
4. Refresh token expired → user is redirected to login

### API Versioning

All API routes are prefixed with `/api/v1/`. When breaking changes are needed, create `/api/v2/` alongside the existing version.

## What NOT to Do

- Do NOT use global state or package-level variables for dependencies
- Do NOT write raw SQL in handlers — always use the repository layer
- Do NOT skip error handling — every error must be checked
- Do NOT commit `.env` files — use `.env.example` as a template
- Do NOT use `any` type in TypeScript — define proper types
- Do NOT install new dependencies without checking if an existing one covers the use case
