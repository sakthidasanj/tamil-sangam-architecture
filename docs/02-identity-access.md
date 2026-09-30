# 02 — Identity & Access

## Identity provider: Firebase Authentication (shared SSO)

Both applications authenticate against **one Firebase project**. A member who
is also a parent uses one login across the portal and the schools platform.

Supported sign-in methods:

- Email + password (Firebase handles hashing/storage — **we never store
  passwords ourselves**)
- Phone OTP
- Google sign-in

No custom password storage, no home-grown session tokens. The client holds a
Firebase ID token; the API verifies it on every request with the Firebase
Admin SDK.

## Authorization model

**Firebase custom claims** carry role hints (e.g. `roles: ["teacher"]`,
`sangam_admin: true`). Claims are convenient but not trusted on their own:

1. Client sends Firebase ID token with each API call.
2. API verifies the token signature and expiry (Admin SDK).
3. API loads the user's roles/scopes from **its own database**
   (the claims just tell it which rows to check).
4. API enforces row-level scoping (e.g. teacher → only their classes).

> Rule: custom claims for UX routing; database-backed checks for
> authorization. Every endpoint re-verifies.

## Role matrices

### Sangam Portal

| Capability | member | organizer | volunteer | treasurer | admin |
|------------|:------:|:---------:|:---------:|:---------:|:-----:|
| View events / buy tickets | ✅ | ✅ | ✅ | ✅ | ✅ |
| Create & manage own events | — | ✅ | — | — | ✅ |
| Check in attendees | — | ✅ | ✅ | — | ✅ |
| View donation reports | — | — | — | ✅ | ✅ |
| Issue refunds | — | — | — | ✅ | ✅ |
| Manage members & roles | — | — | — | — | ✅ |

### Tamil Schools Platform

| Capability | student | parent | teacher | school_admin | super_admin |
|------------|:-------:|:------:|:-------:|:------------:|:-----------:|
| View own classes / materials | ✅ | — | — | — | — |
| View own child's progress | — | ✅ | — | — | — |
| Take attendance (own classes) | — | — | ✅ | ✅ | ✅ |
| Enter grades (own classes) | — | — | ✅ | ✅ | ✅ |
| Manage curriculum content | — | — | ✅ | ✅ | ✅ |
| Enroll / withdraw students | — | — | — | ✅ | ✅ |
| Manage fees & invoices | — | — | — | ✅ | ✅ |
| Manage staff & roles | — | — | — | — | ✅ |

### School-side scoping rules (hard requirements)

- **Teachers** see only students in their assigned classes/sections.
- **Parents** see only their own children's records.
- **Students** see only their own classes, materials, and grades.
- **School admins** are scoped to their school/branch where applicable.
- Cross-student browsing is impossible by construction: every query is
  filtered by the authenticated user's scope before it runs.

## Key flows

- **Sign-up:** Firebase creates the user → API creates the matching row
  (`members` or `guardians`/`students`) → admin assigns roles → custom
  claims set via Admin SDK.
- **Role change:** update DB first, then refresh custom claims; force
  token refresh on the client.
- **Parent linking:** a parent account is linked to student records only
  after verification (e.g. enrollment record match or admin approval) —
  never by self-assertion alone.

See [diagrams/auth-flow.mmd](../diagrams/auth-flow.mmd).
