# 08 — Cost & Grants

Planning guidance for the nonprofit's infrastructure spend. **All figures
are estimates — revalidate quotas and pricing before deployment.**

## GCP hosting: the honest picture

- **Google does not offer a general annual nonprofit GCP hosting-credit
  grant** equivalent to Azure's guaranteed $2,000/year. (Research-specific
  credits exist but are not a general nonprofit entitlement.)
- So GCP hosting = ordinary **free-tier allowances** + paid usage for the
  rest. Design to stay inside free tier where practical:
  - Cloud Run: generous free request/instance-hour tier — our bursty,
    low-baseline traffic fits well.
  - Cloud Storage: small free allowance; event media and curriculum
    content are modest in size.
  - Firebase Authentication: free tier covers our user counts.
- **Watch the bill:** Cloud SQL is the main cost to size carefully —
  start on the smallest instance that holds the data, with backups on.
  Set up billing alerts from day one.

## Google Ad Grants (marketing, not hosting)

- Google for Nonprofits / **Ad Grants**: up to **$10,000/month** in
  eligible Google Ads spend for the nonprofit's outreach (event promotion,
  enrollment drives).
- This is advertising credit, not infrastructure credit — it doesn't
  reduce the GCP bill.

## Azure: the safety net

- **Azure's $2,000/year nonprofit grant** is confirmed and guaranteed —
  keep it as the backup/safety net (e.g. disaster-recovery copies,
  or a fallback if GCP costs surprise us).
- Not the primary platform: the team and architecture are GCP-first.

## Stripe

- Nonprofit pricing may be available — confirm directly with Stripe
  before launch; standard fees apply until then.
- Whatever the rate, card data is never stored (see
  [03 — Data Model](03-data-model.md)).

## Cost discipline checklist

- [ ] Billing alerts at $10 / $50 / $100 per project
- [ ] Cloud SQL right-sized quarterly
- [ ] Cloud Run min-instances = 0 (scale to zero) except where
      cold-start latency is proven to hurt
- [ ] Storage lifecycle rules (archive old event media)
- [ ] Review free-tier usage monthly for the first year
