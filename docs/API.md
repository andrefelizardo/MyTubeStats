# API Documentation

Base URL: `http://localhost:8000/api/v1`

## Response Format

All endpoints return a consistent JSON envelope:

```json
{
  "data": {},
  "error": null,
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100
  }
}
```

On error:

```json
{
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": {
      "field": "email",
      "rule": "email"
    }
  },
  "meta": null
}
```

## Authentication

### Register

```
POST /auth/register

Body:
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "securePassword123"
}

Response (201):
{
  "data": {
    "user": { "id": "uuid", "name": "John Doe", "email": "john@example.com" },
    "access_token": "eyJ...",
    "refresh_token": "eyJ..."
  }
}
```

### Login

```
POST /auth/login

Body:
{
  "email": "john@example.com",
  "password": "securePassword123"
}

Response (200):
{
  "data": {
    "user": { "id": "uuid", "name": "John Doe", "email": "john@example.com" },
    "access_token": "eyJ...",
    "refresh_token": "eyJ..."
  }
}
```

### Refresh Token

```
POST /auth/refresh

Body:
{
  "refresh_token": "eyJ..."
}

Response (200):
{
  "data": {
    "access_token": "eyJ...",
    "refresh_token": "eyJ..."
  }
}
```

### Logout

```
POST /auth/logout
Authorization: Bearer <access_token>

Response (204): No content
```

## Protected Endpoints

All protected endpoints require the `Authorization` header:

```
Authorization: Bearer <access_token>
```

### User Profile

```
GET /users/me
Authorization: Bearer <access_token>

Response (200):
{
  "data": {
    "id": "uuid",
    "name": "John Doe",
    "email": "john@example.com",
    "role": "user",
    "created_at": "2025-01-01T00:00:00Z"
  }
}
```

```
PATCH /users/me
Authorization: Bearer <access_token>

Body:
{
  "name": "Jane Doe"
}

Response (200):
{
  "data": { "id": "uuid", "name": "Jane Doe", "email": "john@example.com" }
}
```

## Pagination

List endpoints support pagination:

```
GET /resource?page=1&per_page=20&sort=created_at&order=desc
```

Parameters:

- `page` — Page number (default: 1)
- `per_page` — Items per page (default: 20, max: 100)
- `sort` — Sort field (default: `created_at`)
- `order` — Sort order: `asc` or `desc` (default: `desc`)

## Error Codes

| HTTP Status | Code               | Description                           |
| ----------- | ------------------ | ------------------------------------- |
| 400         | `VALIDATION_ERROR` | Invalid request body or parameters    |
| 401         | `UNAUTHORIZED`     | Missing or invalid token              |
| 403         | `FORBIDDEN`        | Insufficient permissions              |
| 404         | `NOT_FOUND`        | Resource not found                    |
| 409         | `CONFLICT`         | Resource already exists               |
| 422         | `UNPROCESSABLE`    | Valid syntax but semantically invalid |
| 429         | `RATE_LIMITED`     | Too many requests                     |
| 500         | `INTERNAL_ERROR`   | Unexpected server error               |

## Rate Limiting

API endpoints are rate-limited per IP and per authenticated user. Limits are returned in response headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704067200
```
