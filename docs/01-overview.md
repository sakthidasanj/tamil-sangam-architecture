# 01 — Architecture Overview

## Vision

One shared platform serving two missions of our registered US nonprofit
Tamil Sangam:

1. **Sangam Portal** (~5,000 members) — community hub: member accounts,
   events, ticket sales, donations, payments, volunteer coordination.
2. **Tamil Schools Platform** (~2,000 students) — school operations:
   enrollment, classes, attendance, gradebook, curriculum content,
   parent portal, fees, administration.

## Guiding principles

- **GCP-first.** Build on Google Cloud; use free-tier allowances where
  practical. Azure's $2,000/year nonprofit grant is the backup/safety net,
  not the primary plan.
- **Built by our own developers.** No low-code builders, no WordPress —
  the team writes and owns the full stack.
- **Two apps, hard boundaries.** Separate codebases, separate GCP projects,
  separate PostgreSQL databases, separate audit and billing boundaries.
  The only thing they share is identity.
- **Compliance is day one.** With ~2,000 minors in the system, COPPA/FERPA
  guardrails (parental consent, data minimization, strict authorization,
  audit logs, retention/deletion) are build requirements, not later hardening.
- **Tamil-first UX.** Bilingual Tamil/English, mobile-first, readable Tamil
  typography.

## System shape

```
                    ┌─────────────────────────┐
                    │   Firebase Authentication│  (shared SSO)
                    │ email · phone OTP · Google│
                    └────────────┬────────────┘
                                 │ ID tokens + custom claims
            ┌────────────────────┼────────────────────┐
            ▼                    ▼                    ▼
   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
   │  Sangam Portal  │  │ Tamil Schools   │  │     Stripe      │
   │  Next.js + API  │  │ Next.js + API   │  │ tickets/donations│
   │  (Cloud Run)    │  │ (Cloud Run)     │  │ /fees           │
   └────────┬────────┘  └────────┬────────┘  └─────────────────┘
            ▼                    ▼
   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
   │ PostgreSQL      │  │ PostgreSQL      │  │ Cloud Storage   │
   │ sangam_db       │  │ school_db       │  │ media + content │
   │ (Cloud SQL)     │  │ (Cloud SQL)     │  │ (shared buckets)│
   └─────────────────┘  └─────────────────┘  └─────────────────┘
```

See [diagrams/system.mmd](../diagrams/system.mmd) for the full diagram.

## Tech stack

| Layer | Choice | Notes |
|-------|--------|-------|
| UI | Next.js (React) | SSR for public-event SEO; Tamil/English bilingual |
| API | Modular monolith per app | One deployable per app, modules inside — no premature microservices |
| Runtime | Cloud Run | Serverless containers, scales to zero |
| Database | PostgreSQL on Cloud SQL | One database per app, no cross-DB joins |
| Identity | Firebase Authentication | Email/password, phone OTP, Google sign-in; shared SSO |
| Authorization | Firebase custom claims + server-side checks | Claims are hints; the API re-verifies every request |
| File/media | Cloud Storage | Event media, curriculum content, uploads |
| Payments | Stripe | Ticketing, donations, school fees; card data never stored |
| Secrets | Secret Manager | API keys, Stripe keys, service credentials |
| CI/CD | GitHub Actions | Build → test → deploy to Cloud Run |

## What lives where

| Concern | Sangam Portal | Tamil Schools |
|---------|---------------|---------------|
| GCP project | `sangam-portal-prod` | `tamil-schools-prod` |
| Database | `sangam_db` | `school_db` |
| Users | members, organizers, volunteers, treasurers | students, parents, teachers, school admins |
| Money | tickets, donations | fees |
| Compliance | standard nonprofit | COPPA/FERPA (minors) |

Shared across both: Firebase Auth project (SSO), Cloud Storage buckets
(with per-app prefixes and access rules), Stripe account (separate
products/prices per app), CI/CD patterns.
