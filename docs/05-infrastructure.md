# 05 — Infrastructure (GCP)

GCP-first. Two GCP projects (one per application) under the nonprofit's
Google Cloud organization, so billing, IAM, and blast radius stay separate.

## Project layout

| Project | Purpose | Key resources |
|---------|---------|---------------|
| `sangam-portal-prod` (+ `-dev`) | Community portal | Cloud Run service, Cloud SQL `sangam_db`, Storage buckets |
| `tamil-schools-prod` (+ `-dev`) | Schools platform | Cloud Run service, Cloud SQL `school_db`, Storage buckets |
| (shared) Firebase project | Identity for both | Firebase Authentication |
| (shared) | Payments | Stripe account (separate products per app) |

Environments: `dev` and `prod` per app to start; add `staging` when the
team needs a pre-prod gate (see [06 — CI/CD](06-cicd.md)).

## Compute — Cloud Run

- One service per app: `sangam-portal` (Next.js + API), `tamil-schools`
  (Next.js + API). Each is a **modular monolith**: one deployable,
  well-separated modules inside.
- Scales to zero; pay per request. Fits bursty nonprofit traffic
  (event launches, enrollment windows).
- Managed TLS, custom domains per app.

## Data — Cloud SQL for PostgreSQL

- One instance per project (start small — shared-core/db-f1-micro class
  is fine for dev; size prod from real load).
- Automated backups + point-in-time recovery enabled from day one.
- Private IP via VPC connector; no public database endpoints.
- `sangam_db` and `school_db` as designed in [03 — Data Model](03-data-model.md).

## Storage — Cloud Storage

- Buckets per app: `sangam-portal-media`, `tamil-schools-content`, etc.
- Signed URLs for private content (curriculum materials, student-visible files).
- Lifecycle rules: move old event media to cheaper storage classes.

## Identity & secrets

- Firebase Authentication in a dedicated Firebase project linked to both
  GCP projects.
- Secret Manager per project: Stripe keys, any third-party API keys.
- Cloud Run services get secrets via mounted volumes/env — never baked
  into images.

## Networking & security basics

- Cloud Run ingress: internal + Cloud Load Balancing for the public apps,
  or direct with custom domains — decide in build; both are fine at this scale.
- Cloud SQL private IP only; Cloud Run connects via VPC connector.
- IAM: developers get project-scoped roles; production deploy rights via
  CI/CD service accounts, not personal accounts.

## Observability

- Cloud Logging + Cloud Monitoring from day one (they're on by default —
  set up alerts, don't just collect logs).
- Alert on: 5xx rate, latency p95, failed Stripe webhooks, Cloud SQL
  storage/CPU, auth failure spikes.
- Uptime checks on both public apps.

## What we are NOT building yet

- No Kubernetes, no premature microservices, no multi-region active/active.
- Single region to start (pick one close to most users, e.g. `us-central1`
  or `us-east4`); revisit if the member base spreads.
