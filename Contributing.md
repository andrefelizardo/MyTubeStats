# Contributing Guide

## Getting Started

1. Fork and clone the repository
2. Follow the [README.md](README.md) setup instructions
3. Create a feature branch from `main`

## Branch Naming

```
feature/add-user-dashboard
fix/login-redirect-loop
chore/update-dependencies
refactor/auth-middleware
docs/api-endpoints
test/user-service
```

## Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add user registration endpoint
fix: resolve JWT refresh token race condition
docs: update API documentation for v1
chore: upgrade Go to 1.22
refactor: extract email service from user handler
test: add integration tests for billing service
```

### Scope (optional)

```
feat(api): add rate limiting middleware
fix(ui): correct responsive layout on mobile
```

## Pull Request Process

1. Create a branch following the naming convention above
2. Write code and tests
3. Ensure all checks pass locally:

   ```bash
   # Backend
   cd backend && go test ./... && go vet ./...

   # Frontend
   cd frontend && npm run lint && npm run test
   ```

4. Push your branch and open a PR against `main`
5. Fill in the PR template with a clear description
6. Request review from at least one team member
7. Address review comments
8. Squash and merge once approved

## Code Style

### Go

- Run `gofmt` and `go vet` before committing
- Follow [Effective Go](https://go.dev/doc/effective_go) guidelines
- Keep functions short and focused (< 40 lines as a guideline)
- Export only what needs to be public
- Write doc comments for all exported types and functions
- Use meaningful variable names — avoid single-letter names outside of loops

### TypeScript / React

- Follow the ESLint configuration in the project
- Use TypeScript strict mode — no `any`
- Prefer named exports over default exports for non-page components
- Keep components under 150 lines; extract sub-components when needed
- Co-locate tests: `Component.tsx` → `Component.test.tsx`

### SQL

- Use snake_case for all identifiers
- Always include `created_at` and `updated_at` on new tables
- Write both `up` and `down` migrations
- Test `down` migrations before pushing

## Testing Guidelines

### Backend

- Unit tests for service layer logic
- Integration tests for repository layer (use test database)
- Handler tests using `httptest` for HTTP layer
- Use table-driven tests where appropriate
- Aim for 80%+ coverage on business logic

### Frontend

- Unit tests for utility functions and hooks
- Component tests for interactive UI elements
- Use React Testing Library (not Enzyme)
- Test user behavior, not implementation details

## Environment

- Never commit `.env` files
- Update `.env.example` when adding new environment variables
- Document new variables in the README environment section

## Questions?

Open an issue or reach out to the team lead.
