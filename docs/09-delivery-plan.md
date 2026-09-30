# 09 — Phased Delivery Plan

Build in slices that each deliver real value. Compliance and identity
foundations come first because everything else stands on them.

## Phase 0 — Foundations (weeks 1–4)

- GCP projects, Cloud Run, Cloud SQL, Storage, Secret Manager, billing alerts.
- Firebase project + Authentication (email, phone OTP, Google sign-in).
- CI/CD skeletons (GitHub Actions → Cloud Run) for both future app repos.
- Shared UI kit: Next.js setup, Tamil/English i18n, Noto Sans Tamil,
  mobile-first base styles.
- Auth library: token verification + claims + server-side scope checks,
  with authorization tests.

**Exit criteria:** a signed-in user hits a "hello" API that enforces roles;
pipelines deploy to dev on merge.

## Phase 1 — Sangam Portal MVP (weeks 5–12)

- Member accounts, household grouping, role management.
- Events: bilingual event pages (SSR), ticket types, checkout via Stripe.
- Donations with receipts.
- Organizer/volunteer flows: event creation, attendee check-in.
- Treasurer reports, refunds.

**Exit criteria:** a real paid event can run end to end.

## Phase 2 — Tamil Schools MVP (weeks 13–22)

- Enrollment with verified parent/guardian linking + consent capture.
- Classes, sections, teacher assignment.
- Attendance marking.
- Parent portal: child's schedule, attendance, announcements.
- Curriculum content management (bilingual).

**Exit criteria:** one term runs on the platform: enrollment → attendance
→ parent visibility.

## Phase 3 — Depth (weeks 23–32)

- Gradebook: assignments, grading, report views (scoped per
  [02 — Identity & Access](02-identity-access.md)).
- Fee schedules, invoicing, Stripe payments, treasurer reconciliation.
- Volunteer coordination and advanced event features in the portal.
- Search, notifications (email/SMS), Tamil transliteration-aware search.

## Phase 4 — Hardening & scale (ongoing)

- Annual access reviews, retention/deletion workflows exercised.
- Load testing before enrollment/event peaks.
- Staging environment if not yet added; multi-region only if proven necessary.
- Ad Grants campaigns for outreach (see [08 — Cost & Grants](08-cost-grants.md)).

## Cross-phase rules

- No phase ships without its authorization tests passing.
- No production data copied to dev, ever.
- Every phase updates these docs when reality diverges from the plan.
