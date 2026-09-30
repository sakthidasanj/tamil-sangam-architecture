# Tamil Sangam Platform — Architecture

Architecture documents and diagrams for the digital platform of our registered
US nonprofit Tamil Sangam:

- **Sangam Portal** — community portal for ~5,000 members
  (member logins, events, ticket sales, donations, payments)
- **Tamil Schools Platform** — school management for ~2,000 students
  (enrollment, classes, attendance, gradebook, curriculum content,
  parent portal, fees, administration)

Our in-house development team builds and maintains everything. The direction
is **GCP-first**, using free-tier services where practical.

## Documents

| Doc | Contents |
|-----|----------|
| [01 — Overview](docs/01-overview.md) | Vision, the two applications, guiding principles, tech stack |
| [02 — Identity & Access](docs/02-identity-access.md) | Firebase Auth SSO, custom claims, RBAC matrices |
| [03 — Data Model](docs/03-data-model.md) | PostgreSQL schemas and table outlines for both apps |
| [04 — Compliance](docs/04-compliance.md) | COPPA / FERPA guardrails (day-one requirements) |
| [05 — Infrastructure](docs/05-infrastructure.md) | GCP layout: projects, Cloud Run, Cloud SQL, Storage |
| [06 — CI/CD & Environments](docs/06-cicd.md) | Pipelines, environments, branch strategy, migrations |
| [07 — Tamil i18n](docs/07-i18n-tamil.md) | Bilingual Tamil/English requirements |
| [08 — Cost & Grants](docs/08-cost-grants.md) | Cost guidance, Google Ad Grants, Azure backup grant |
| [09 — Delivery Plan](docs/09-delivery-plan.md) | Phased build plan |

## Diagrams (Mermaid)

| Diagram | Contents |
|---------|----------|
| [system.mmd](diagrams/system.mmd) | Full system diagram: users → apps → APIs → data |
| [auth-flow.mmd](diagrams/auth-flow.mmd) | Sign-in and authorization flow |
| [data-boundaries.mmd](diagrams/data-boundaries.mmd) | Database separation and audit boundaries |

Render Mermaid with any Mermaid-capable viewer (GitHub renders `.mmd`
files natively in markdown code fences; copy the contents into
[mermaid.live](https://mermaid.live) for a quick view).

## Key decisions (summary)

- Two separate applications and GCP projects, separate PostgreSQL databases,
  separate audit/billing boundaries.
- Shared identity/SSO via **Firebase Authentication**
  (email/password, phone OTP, Google sign-in). No custom password storage.
- Role-based access via Firebase custom claims **plus** server-side API
  authorization on every request.
- One modular monolith per app on **Cloud Run** (no premature microservices).
- **Next.js** UI, Tamil/English bilingual, mobile-first, SSR for public-event SEO.
- **Stripe** for ticketing/donations/fees. Card data is never stored.
- COPPA/FERPA guardrails are day-one build requirements, not later hardening.

> Pricing, free-tier quotas, and grant figures in these docs are planning
> estimates — revalidate before deployment.
