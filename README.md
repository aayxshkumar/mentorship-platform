# Mentorship Platform

![CI](https://github.com/aayxshkumar/mentorship-platform/actions/workflows/ci.yml/badge.svg)

A mentor and mentee matching app. Users register, build a profile (as a mentor or mentee) with skills and interests, browse other profiles, and send mentorship requests that the other side can accept or decline. The backend is a Node/Express REST API on PostgreSQL; the frontend is plain HTML, CSS and JavaScript modules with no build step. No live demo.

## Features

- Registration and login with bcrypt-hashed passwords, a JWT access token, and a refresh token in an HTTP-only cookie
- Accounts default to the `mentee` role; profiles carry a `mentor` or `mentee` role
- Profile create, read, update and delete
- Profile browsing with `role`, `skills` and `interests` filters and pagination
- Mentorship requests with statuses `pending`, `accepted`, `declined`, `cancelled`, `completed`
- Request validation with express-validator
- Rate limiting on login and register: 10 attempts per 15 minutes per IP

## Tech stack

- Backend: Node.js, Express 5, `pg`, jsonwebtoken, bcryptjs, express-validator, express-rate-limit, cookie-parser
- Database: PostgreSQL (`database/schema.sql`)
- Frontend: static HTML pages and ES module scripts

## Getting started

Requires Node.js 20 or newer (CI runs on Node 22) and a PostgreSQL database.

```bash
git clone https://github.com/aayxshkumar/mentorship-platform.git
cd mentorship-platform
```

Database:

```bash
createdb mentorship_platform
psql mentorship_platform -f database/schema.sql
```

`database/seeds.sql` was written against an earlier version of the schema (unquoted table names, `Mentorship_Requests`) and does not load against `schema.sql` as is.

Backend:

```bash
cd backend
npm ci
cp .env.example .env   # then fill in the values
node server.js
```

The server checks the database connection on startup and exits if it cannot connect.

Frontend, in a second terminal (any static file server works; the API's CORS settings allow `http://localhost:8080` and `http://127.0.0.1:8080`):

```bash
python3 -m http.server 8080 --directory frontend
```

Then open http://localhost:8080. The frontend calls the API at `http://localhost:5001/api` (set in `frontend/src/js/services/api.config.js`), so run the backend with `PORT=5001`.

### Environment variables (backend)

| Name | Purpose |
| --- | --- |
| `PORT` | API port (defaults to 3000; the frontend expects 5001) |
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Access token signing key |
| `JWT_REFRESH_SECRET` | Refresh token signing key |
| `NODE_ENV` | `production` turns on SSL for the database connection |

## API

| Method | Path | Auth |
| --- | --- | --- |
| POST | `/api/auth/register` | rate limited |
| POST | `/api/auth/login` | rate limited |
| POST | `/api/auth/logout` | |
| POST | `/api/auth/refresh-token` | refresh cookie |
| GET | `/api/profiles` | bearer token |
| GET, PUT, DELETE | `/api/profiles/:userId` | bearer token |
| POST | `/api/profiles` | bearer token |
| GET, POST | `/api/requests` | bearer token |
| PATCH | `/api/requests/:requestId` | bearer token |

## Project structure

```
backend/
  server.js          Express app and startup
  config/            env config and pg pool
  routes/            auth, profiles, requests
  controllers/       request handlers
  models/            SQL queries per table
  middleware/        auth, validation, rate limiting, error handler
database/            schema.sql, seeds.sql (outdated)
frontend/            HTML pages and src/ (css, page scripts, API services)
```
