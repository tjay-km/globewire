# 🌐 GlobeWire - International Payment Portal

A secure international payments portal built with **Next.js, TypeScript, Prisma and MongoDB**.
Customers create cross-border payments, and bank employees verify them and submit them to SWIFT.
Every action is recorded in a tamper-evident audit log.

> Group project for **APDS7311 (Application Development Security)** at IIE MSA, 2025, built with
> Keanu Muller, Kabelo Diutluileng and Kyle James. I was one of the two main developers
> (146 of the 208 commits). This is the portfolio copy: environment files and build artefacts are
> excluded.

**Demo videos:** [Final demo](https://youtu.be/_J5ldhbyFiU) · [Task 2 demo](https://youtu.be/QsTyMCxH0iQ) · [Database walkthrough](https://youtu.be/iGB8hhKpEwk)

## Features

- **Customer portal**: register, log in, create international payments (amount, currency, SWIFT
  code, recipient) and track their status.
- **Employee portal**: review pending payments, verify them and submit them to SWIFT. Each
  privileged action needs a single-use action token.
- **Audit logs**: every login, payment, verification and submission is logged with IP and user
  agent, chained with SHA-256 hashes so tampering is detectable (`scripts/verify-audit-chain.mjs`).
- **No public staff sign-up**: employee accounts can only be created by seeding the database.

## Security

The project was graded on application security, so most of the work went here:

| Area | Implementation |
|---|---|
| Passwords | bcrypt with 12 rounds (configurable), strong password policy |
| Input validation | Whitelist RegEx validation on every field, client and server side (`lib/validation.ts`) |
| Sessions | JWT in `HttpOnly`, `SameSite`, `Secure` cookies; 30-minute sessions; 15-minute single-use action tokens |
| CSRF | Token issued by `/api/csrf` and checked on every state-changing request |
| Brute force | Token-bucket rate limiting per IP (3 requests/minute by default) and account lockout after failed logins |
| Transport | HTTPS server with HTTP→HTTPS redirect and HSTS in production |
| Headers | Strict Content-Security-Policy, `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy: no-referrer` |
| Data access | Prisma ORM only, with guardrails against raw queries |
| Code quality | Jest test suite, ESLint, and SonarCloud analysis in CircleCI |

## Tech stack

Next.js (App Router pages + API routes) · React · TypeScript · Prisma ORM · MongoDB Atlas ·
JSON Web Tokens · bcrypt · Tailwind CSS · Jest · CircleCI + SonarCloud

## Getting started

Prerequisites: Node.js 18+ and a MongoDB Atlas cluster (the free tier works).

```bash
git clone https://github.com/tjay-km/globewire.git
cd globewire
npm install
cp .env.example .env        # Windows: copy .env.example .env
```

Fill in `.env`. Never commit real values.

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | Your MongoDB Atlas connection string |
| `JWT_SECRET` | A long random string, e.g. `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"` |
| `DISABLE_RATE_LIMIT`, `ALLOW_REGISTRATION` | Optional development toggles |

Then create the database client and the local test accounts, and start the app:

```bash
npm run prisma:generate
npm run prisma:seed          # creates a test customer and employee locally

npm run dev                  # http://localhost:3000
# or, with HTTPS:
npm run generate-certs       # self-signed certificates for localhost (Windows PowerShell)
npm run build
NODE_ENV=production node server.mjs   # https://localhost:3443
```

The seeded login details and a full manual test walkthrough (payments, verification, audit logs and
security checks) are in **[docs/TESTING.md](docs/TESTING.md)**.

## Scripts

| Command | What it does |
|---|---|
| `npm run dev` | Development server |
| `npm run build` / `npm start` | Production build / server |
| `npm run start:https` | HTTPS server (`server.mjs`) |
| `npm test` / `npm run test:ci` | Jest tests / tests with coverage |
| `npm run lint` | ESLint |
| `npm run prisma:studio` | Browse the database |

API collections for Postman are in [`tools/postman/`](tools/postman/).

## Project structure

```
app/            UI pages: customer (login, register, payments), employee (dashboard, audit logs), components
pages/api/      API routes: auth, customer, employee, payments, audit-logs, csrf, logout
lib/            auth, sessions, validation, rate limiting, audit chain, CSRF fetch, DB client
prisma/         MongoDB schema and seed script
middleware.ts   security headers, CSP, HTTPS enforcement
test/           Jest unit and component tests
scripts/        audit-chain verification and SonarCloud helpers
```

## Academic integrity

This was submitted as coursework for APDS7311 at IIE MSA. It's shared as a portfolio piece. Please
don't copy it for your own coursework.
