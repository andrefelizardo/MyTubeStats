# MyTubeStats

> Your YouTube history, finally visualized. A "YouTube Wrapped" SaaS built with Go and React.

YouTube doesn't expose watch history through its API — by design. **MyTubeStats** works around this by letting users export their own data via Google Takeout and upload it for analysis. The result: a personal, interactive dashboard showing total watch time, top channels, viewing habits over time, and more.

---

## How It Works

```
Google Takeout Export  →  Upload JSON/ZIP  →  Processing  →  Interactive Dashboard
```

1. User follows a guided tutorial to export their YouTube history via [Google Takeout](https://takeout.google.com)
2. Upload the `.json` or `.zip` file to the platform
3. The backend streams and parses the file (no full load into memory), extracts video IDs, and fetches metadata from the YouTube Data API v3
4. A personalized stats dashboard is generated with total watch time, most-watched channels, and viewing patterns

---

## Key Technical Challenges

- **Large file processing** — watch history files can reach gigabytes. The backend processes them as streams to avoid exhausting server memory.
- **YouTube API rate limits** — video metadata is fetched in batches with graceful backoff and retry logic.
- **Privacy by default** — raw user data is never permanently stored without explicit consent. Files are processed in memory and discarded.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, TypeScript, Tailwind CSS, Zustand, Vite |
| Backend | Go, Chi router, JWT authentication |
| Database | PostgreSQL 16, sqlc (type-safe queries), pgx |
| Infra | Docker Compose (local), GitHub Actions (CI/CD) |

---

## Architecture

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Go REST API    │────▶│  PostgreSQL  │
│  React/Vite  │◀────│   (Chi Router)   │◀────│     16       │
│  :5173       │     │   :8000          │     │   :5432      │
└──────────────┘     └──────────────────┘     └──────────────┘
```

The backend follows a strict layered architecture:

```
Router → Middleware → Handler → Service → Repository → PostgreSQL
```

Each layer has a single responsibility: routing, auth/logging, HTTP parsing, business logic, and data access — kept separate and independently testable.

---

## Getting Started

### Prerequisites

- [Go 1.22+](https://go.dev/dl/)
- [Node.js 20+](https://nodejs.org/)
- [Docker](https://www.docker.com/) (for PostgreSQL)
- A [YouTube Data API v3](https://console.cloud.google.com/) key

### Local Setup

```bash
# 1. Clone the repo
git clone https://github.com/andrefelizardo/MyTubeStats.git
cd MyTubeStats

# 2. Start the database
docker compose up -d

# 3. Configure environment variables
cp api/.env.example api/.env
# Fill in your YouTube API key and database credentials

# 4. Run the backend
cd api && go run ./cmd/server

# 5. Run the frontend (new terminal)
cd frontend && npm install && npm run dev
```

Frontend: `http://localhost:5173` | API: `http://localhost:8000`

### Environment Variables

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Secret key for signing JWT tokens |
| `YOUTUBE_API_KEY` | YouTube Data API v3 key |
| `PORT` | API server port (default: `8000`) |

---

## API Overview

All endpoints live under `/api/v1/` and return a consistent envelope:

```json
{
  "data": {},
  "error": null,
  "meta": { "page": 1, "per_page": 20, "total": 100 }
}
```

Authentication uses short-lived JWT access tokens + long-lived refresh tokens stored in the database for revocation. See [`docs/API.md`](docs/API.md) for the full reference.

---

## Project Structure

```
MyTubeStats/
├── api/
│   ├── cmd/server/        # Entry point
│   ├── internal/
│   │   ├── handler/       # HTTP layer
│   │   ├── service/       # Business logic
│   │   ├── repository/    # Data access (sqlc)
│   │   └── middleware/    # Auth, logging, rate limiting
│   └── migrations/        # SQL migrations (up + down)
├── frontend/
│   └── src/
│       ├── api/           # Axios client & endpoints
│       ├── features/      # Feature modules (auth, dashboard…)
│       ├── components/    # Shared UI components
│       └── stores/        # Zustand global state
└── docs/
    ├── PRD.md             # Product requirements
    ├── ARCHITECTURE.md    # System design decisions
    └── API.md             # Full API reference
```

---

## Contributing

Contributions are welcome. Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a PR — it covers branch naming, commit conventions, code style for Go/TypeScript/SQL, and testing expectations.

**Quick summary:**
- Branch: `feature/`, `fix/`, `chore/`, `refactor/`, `docs/`, `test/`
- Commits: [Conventional Commits](https://www.conventionalcommits.org/)
- Before pushing: `go test ./... && go vet ./...` (backend) | `npm run lint && npm run test` (frontend)
- PRs go against `main`, squash and merge once approved

---

## Docs

- [`docs/PRD.md`](docs/PRD.md) — Product requirements and user journey
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — System design, auth flow, error handling
- [`docs/API.md`](docs/API.md) — Full REST API reference

---

## License

MIT
