# 03 — Data Model

Two PostgreSQL databases (Cloud SQL), one per application. **No cross-database
joins** — the only shared concept is the Firebase `uid`, stored as an opaque
reference in each DB.

## sangam_db (Sangam Portal)

| Table | Purpose | Key columns |
|-------|---------|-------------|
| `members` | Member accounts | `id`, `firebase_uid` (unique), `name`, `email`, `phone`, `tamil_name`, `membership_status`, `joined_at` |
| `households` | Family groupings | `id`, `primary_member_id` |
| `household_members` | Links members to households | `household_id`, `member_id`, `relationship` |
| `roles` | Role assignments | `member_id`, `role` (organizer, volunteer, treasurer, admin) |
| `events` | Events | `id`, `title`, `title_ta`, `description`, `description_ta`, `starts_at`, `ends_at`, `venue`, `status` |
| `ticket_types` | Ticket tiers per event | `id`, `event_id`, `name`, `price_cents`, `capacity` |
| `orders` | Ticket orders | `id`, `member_id`, `event_id`, `status`, `total_cents`, `stripe_payment_intent_id` |
| `order_items` | Line items | `order_id`, `ticket_type_id`, `quantity`, `price_cents` |
| `donations` | Donations | `id`, `member_id` (nullable for guests), `amount_cents`, `campaign`, `stripe_payment_intent_id`, `receipt_sent_at` |
| `volunteer_signups` | Volunteer coordination | `id`, `member_id`, `event_id`, `role`, `status` |
| `audit_log` | Immutable audit trail | `id`, `actor_member_id`, `action`, `entity`, `entity_id`, `at`, `metadata` |

## school_db (Tamil Schools Platform)

| Table | Purpose | Key columns |
|-------|---------|-------------|
| `students` | Student records | `id`, `name`, `name_ta`, `dob`, `grade_level`, `status`, `enrolled_at` |
| `guardians` | Parent/guardian accounts | `id`, `firebase_uid` (unique), `name`, `email`, `phone`, `consent_status`, `consent_at` |
| `student_guardians` | Verified links | `student_id`, `guardian_id`, `relationship`, `verified_at`, `verified_by` |
| `staff` | Teachers & admins | `id`, `firebase_uid` (unique), `name`, `role` (teacher, school_admin), `status` |
| `academic_years` | School years/terms | `id`, `name`, `starts_at`, `ends_at` |
| `classes` | Class offerings | `id`, `academic_year_id`, `name`, `name_ta`, `level`, `schedule` |
| `sections` | Class sections | `id`, `class_id`, `teacher_id`, `capacity`, `room` |
| `enrollments` | Student ↔ section | `id`, `student_id`, `section_id`, `status`, `enrolled_at` |
| `attendance` | Daily attendance | `id`, `section_id`, `student_id`, `date`, `status` (present/absent/late), `marked_by` |
| `assignments` | Assignments | `id`, `section_id`, `title`, `title_ta`, `due_at`, `max_points` |
| `grades` | Gradebook entries | `id`, `assignment_id`, `student_id`, `points`, `graded_by`, `graded_at` |
| `curriculum_content` | Lessons & materials | `id`, `class_id`, `title`, `title_ta`, `body_md`, `body_md_ta`, `attachment_path`, `visibility` |
| `fee_schedules` | Fee structures | `id`, `academic_year_id`, `name`, `amount_cents` |
| `invoices` | Fee invoices | `id`, `student_id`, `fee_schedule_id`, `amount_cents`, `due_at`, `status`, `stripe_invoice_id` |
| `audit_log` | Immutable audit trail | `id`, `actor_uid`, `actor_role`, `action`, `entity`, `entity_id`, `at`, `metadata` |

## Cross-cutting rules

- **Money:** only Stripe references (`stripe_payment_intent_id`,
  `stripe_invoice_id`) are stored. Card numbers, CVVs — never.
- **PII minimization:** collect only what each feature needs (see
  [04 — Compliance](04-compliance.md)).
- **Tamil content:** user-facing text columns have `_ta` counterparts;
  see [07 — Tamil i18n](07-i18n-tamil.md).
- **Audit logs** are append-only; application roles cannot update or delete
  them.
- **Soft deletes** for member/student records (`status` flags) to preserve
  financial and attendance history; hard delete only via the documented
  retention/deletion process.
- Media files (event photos, curriculum attachments) live in Cloud Storage;
  the DB stores only paths.

See [diagrams/data-boundaries.mmd](../diagrams/data-boundaries.mmd).
