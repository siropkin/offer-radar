# AGENTS.md — offer-radar

Public dashboard of recruiter pitches received and politely declined.
Deployed via GitHub Pages → https://srdkn.com/offer-radar/ (apex custom domain;
siropkin.github.io/offer-radar/ 301-redirects there — always verify with `curl -sL`).

## The one hard rule: anonymization

This repo is PUBLIC. The private source of truth — `~/recruiter-offers.md` —
contains recruiter **names and emails** and must NEVER be copied into this repo.

Data that makes it into `index.html`:

- ✅ company / product, dates, channel ("gmail" | "linkedin"), offer text, salary ranges,
  agency / marketplace names ("via Candidate Labs", "Talor Made Talent") — they identify
  a company, not a person
- ❌ recruiter names, emails, calendar links, anything identifying a person

If a recruiter's identity is the interesting part of the story, it stays out.
"Unnamed client (…)" with a short distinguishing hint is the correct way to log a pitch you can't name — the hint keeps rows distinct in the companies count.

## Updating with new offers

1. Read new rows from `~/recruiter-offers.md` (the private log, both Gmail and
   LinkedIn sections — `f` = first contact date, not the follow-up).
2. Strip recruiter names/emails; keep everything else.
3. Append rows to the `DATA` array in `index.html`. One offer = one object:
   `{d, co, f, ch, loc, off, base, pings} — loc is the office location if stated (SF, Sunnyvale, Foster City…), else null` — see the comment above the array.
4. Map the company to a field in `FIELDS`. **There are exactly 8 slots (c1–c8)
   and their color order is CVD-validated — never reorder or add a 9th slot.**
   A genuinely new field folds into the closest existing one or into
   "Staffing (multi-role)". If that feels wrong, ask before inventing a 9th color.
5. Update the `updated` stamp in the header. All counts (total, per-field,
   stated-range, days-per-offer span) are computed from `DATA` — nothing else
   is hardcoded.
6. Verify locally: open `index.html` in a browser, check the tile counts, hover
   the cadence bars, run the filter box. `cli.mjs shot` via chrome-bridge works
   for a quick look.
7. Commit and push to `master` — GitHub Pages redeploys on push (legacy build,
   normally 30–70s; watch `gh api repos/siropkin/offer-radar/pages/builds/latest`).
   Confirm the live URL with `curl -sL` — plain `curl` gets a 301 stub, not the page.
   If the build shows `duration: 0` with status `building` for >2 min, it already
   died on GitHub's side (status lags ~30 min behind reality) — push an empty
   commit to retrigger instead of waiting.

## Design rules (keep the page coherent)

- Charts are hand-rolled SVG computed from `DATA` — no chart library, keep it that way.
- Colors: 8-slot categorical palette (`--c1…--c8`), blue accent (`--accent`) for
  single-series marks. Both were validated for colorblind safety and contrast
  (light mode has a known sub-3:1 WARN on c3/c4/c5 — mitigated by direct labels
  on every bar; keep those labels when editing).
- Theming: tokens in `:root` (light) / `prefers-color-scheme: dark` /
  `[data-theme="dark"]` — the three-state pattern. Every color must be defined
  in the bare `:root` first; a color that only exists inside a dark block is a bug.
- Type: IBM Plex Sans (text) + IBM Plex Mono (data, figures, labels). Tabular
  numerals on anything columnar.
- Mobile: breakpoint at 520px restacks the field bars and shrinks tiles; the
  table scrolls horizontally in its own container. If you add a section, check
  it at 375px wide before pushing.

## Custom domain

`srdkn.com` (apex) is already configured — siropkin.github.io/offer-radar/
301s to srdkn.com/offer-radar/. The earlier `radar.srdkn.com` subdomain idea
is moot while the apex works.
