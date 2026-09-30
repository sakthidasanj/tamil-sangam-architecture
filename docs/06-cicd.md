# 06 — CI/CD & Environments

## Pipelines (GitHub Actions)

One workflow per app repo (repos to be created: `sangam-portal`,
`tamil-schools`; this repo holds architecture only).

```
pull request → lint + typecheck + unit tests
merge to main → build container → run migrations (dev) → deploy to dev
tag release  → integration tests → run migrations (prod) → deploy to prod
```

- Container images: Artifact Registry per GCP project.
- Deploys target Cloud Run via `gcloud run deploy` (or Terraform — pick
  one IaC approach when the team starts; don't hand-roll both).
- Database migrations run as a separate job **before** the new revision
  goes live; migrations are backwards-compatible (expand → migrate →
  contract).

## Environments

| Env | Purpose | Data |
|-----|---------|------|
| `dev` | Daily development | Seed/fake data only — never production copies |
| `prod` | Live | Real data, backups on |

Add `staging` when releases need a pre-prod gate. Never copy prod data
down — especially student data (see [04 — Compliance](04-compliance.md)).

## Branch strategy

- `main` is always deployable.
- Short-lived feature branches → PR → required review → squash merge.
- Release tags (`v1.2.0`) mark what went to prod.

## Quality gates

- Required PR checks: lint, typecheck, unit tests, and (for the schools
  app) authorization tests proving teachers can't see other classes and
  parents can't see other children.
- Dependency scanning (Dependabot) on.
- Stripe webhook handlers tested with Stripe CLI fixtures.

## Secrets in CI

GitHub Actions uses Workload Identity Federation to GCP — no long-lived
service-account keys. Production deploy credentials live only in the
CI identity, never on developer laptops.
