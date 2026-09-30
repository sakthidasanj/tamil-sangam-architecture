# 04 — Compliance (COPPA / FERPA)

With ~2,000 students — many of them minors — these are **day-one build
requirements**, not later hardening. The Sangam Portal (adults) follows the
same data-hygiene baseline; the schools platform adds the minor-specific
controls below.

## Parental consent

- A student record becomes active only after a verified guardian link
  (`student_guardians.verified_at`) **and** recorded consent
  (`guardians.consent_status`, `consent_at`).
- Consent covers: enrollment, attendance tracking, gradebook, and any
  photo/media featuring the child. Each is consentable separately where
  practical.
- Guardians can review and withdraw consent; withdrawal stops new
  collection and triggers the retention/deletion workflow for
  non-essential data.

## Data minimization

- Collect only fields the feature needs (see [03 — Data Model](03-data-model.md)).
- No student data is used for marketing or analytics beyond aggregate,
  de-identified school operations reporting.
- Free-text fields that could collect unnecessary PII are avoided in
  student-facing forms.

## Access control

- Strict scoping per [02 — Identity & Access](02-identity-access.md):
  teachers see only their classes, parents only their children.
- Every student-data endpoint enforces scope server-side; bulk export
  endpoints require `school_admin` or higher and are audit-logged.
- Staff access is role-based and least-privilege; role grants are
  themselves audit-logged.

## Encryption & secrets

- TLS everywhere (Cloud Run managed certificates, Cloud SQL SSL).
- Encryption at rest: Cloud SQL and Cloud Storage default encryption;
  consider CMEK if the board requires it.
- Secrets (Stripe keys, service credentials) in Secret Manager —
  never in code, env files, or chat.

## Audit logging

- Append-only `audit_log` tables in both databases: who did what to whose
  record, when. Application roles cannot modify or delete entries.
- Log authentication events (sign-in, role changes, failed attempts) and
  all student-record reads/writes by staff.

## Retention & deletion

- Documented retention schedule per data category (e.g. attendance kept
  for N years for school records; marketing consents honored promptly).
- Guardian-requested deletion: remove non-essential student data;
  retain only what law or financial record-keeping requires, and say
  what was retained and why.
- Soft-delete in the app; hard-delete/purge via a reviewed,
  logged administrative process.

## Operational practices

- Annual access review: prune staff roles, verify guardian links.
- Incident response: suspected breach of minor data → contain, assess,
  notify board and affected families per legal counsel's guidance.
- Vendor review: Firebase, Stripe, Google Cloud — keep track of their
  compliance postures (they are processors; we are the controller).
