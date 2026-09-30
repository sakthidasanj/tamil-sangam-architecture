# 07 — Tamil Language Requirements (i18n)

The platform is bilingual: **Tamil and English**, with Tamil treated as a
first-class language — not a translation afterthought.

## Text & encoding

- Everything is UTF-8 end to end: database (`UTF8` encoding), API
  (JSON), HTML (`<meta charset="utf-8">`).
- Tamil typeface: **Noto Sans Tamil** (or equivalent) for UI; test
  rendering on low-end Android devices, where most parents will read it.
- Never break Tamil grapheme clusters when truncating (use a proper
  grapheme-aware truncator, not byte/char counts).

## Content model

- User-facing content tables carry `_ta` columns alongside English
  (see [03 — Data Model](03-data-model.md)): event titles/descriptions,
  class names, curriculum titles and bodies, announcements.
- UI strings via a standard i18n framework (e.g. `next-intl`); Tamil is
  the default locale for school-facing surfaces, with an obvious
  Tamil/English toggle persisted per user.
- Curriculum authors write Tamil directly — no romanized-Tamil-to-Unicode
  pipeline needed in v1.

## Formatting

- Dates/times shown in the viewer's locale; Tamil locale (`ta`) formats
  where the user chose Tamil.
- Currency: USD with Tamil labels where appropriate
  (e.g. "₹" never appears — amounts are USD; label as `$` / `டாலர்`).
- Numbers: Western digits by default; Tamil digits optional later —
  don't mix within one surface.

## Input & search

- Accept Tamil input everywhere users type names and content
  (no ASCII-only validation on name fields).
- Search across both languages: searching "மாலா" and "maala" should
  find the same event/student where practical (transliteration-aware
  search is a later enhancement — start with exact-match both scripts).

## SEO & sharing

- Public event pages are server-side rendered (Next.js SSR) with Tamil
  titles/descriptions in meta tags, so Tamil event names show up
  correctly in search results and link previews.
- `lang="ta"` attributes on Tamil-dominant pages.

## Mobile-first

- Most members and parents are on phones. Touch targets, readable Tamil
  at small sizes, and low-bandwidth friendliness (optimized images,
  minimal JS on content pages) are requirements, not polish.
