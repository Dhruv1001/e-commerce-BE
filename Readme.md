# BE — E-commerce API

NestJS + TypeScript + Prisma (PostgreSQL) backend.

## Stack
- NestJS (Express adapter)
- Prisma ORM → PostgreSQL
- Redis (planned, for cart/session cache) — env var scaffolded, not wired yet
- JWT auth: short-lived access token (Bearer header) + rotating refresh token (httpOnly cookie)
- Resend for transactional email (password reset links)

## Setup

1. Copy the env template and fill in real values:
   ```
   cp .env.example .env
   ```
2. Point `DATABASE_URL` at a local or hosted Postgres instance (e.g. [Neon](https://neon.tech) or [Supabase](https://supabase.com) free tier, or run Postgres locally).
3. Get a [Resend](https://resend.com) API key and set `RESEND_API_KEY` — without it, password-reset emails aren't sent; the link is logged to the server console instead (fine for local dev).
4. Install dependencies and apply the schema:
   ```
   npm install
   npx prisma migrate dev --name init
   ```
5. Run the dev server:
   ```
   npm run start:dev
   ```
   The API listens on `http://localhost:4000` by default. `GET /health` should return `{ "status": "ok" }`.

## Auth endpoints (`/auth`)
- `POST /register`, `POST /login` — return `{ accessToken, user }`, set a `refresh_token` httpOnly cookie
- `POST /refresh` — rotates the refresh token using the cookie, returns a new `{ accessToken, user }`
- `POST /logout` — revokes the refresh token, clears the cookie
- `POST /forgot-password` — always responds generically; emails a reset link when the address matches an account
- `POST /reset-password` — `{ token, newPassword }`, invalidates all of that user's refresh tokens
- `GET /me` — protected by `AccessTokenGuard`, returns the caller's JWT payload

`/register`, `/login` and `/forgot-password`/`/reset-password` are rate-limited (`@nestjs/throttler`) against brute-force and email-bombing.

## Prisma
- Schema: `prisma/schema.prisma`
- Config: `prisma7.config.ts`
- Generated client: `src/generated/prisma` (gitignored, regenerated via `npx prisma generate`)

## Scripts
- `npm run start:dev` — dev server with watch
- `npm run build` / `npm run start:prod` — production build/run
- `npm run test` / `npm run test:e2e` — Vitest unit/e2e tests
- `npm run lint` — oxlint
