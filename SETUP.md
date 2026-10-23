# CTVA 2026 — Full Site Setup Guide

This extends your **existing, live** registration system — nothing about
how `tv_awards_register.html` submits changes.

## What you have now

**Backend**
- `Code.gs` — your original backend, unchanged, plus approval/Nalo-code/
  vote-logging/judges'-scoring/results-lookup added at the bottom.

**Public site** (give this whole folder to any static host, or open the
files directly — they link to each other by filename)
- `ctva-home.html` — landing page
- `ctva-about.html` — about the awards
- `ctva-vote-results.html` — **"Check My Result"** (student enters their
  reference code, sees only their own status/votes/rank) **+** the public
  live leaderboard by category
- `ctva-nominees.html` — nominee gallery by category, with photo + Nalo code
- `ctva-judges-ratings.html` — judges' panel scores per nominee, ranked
- `ctva-vote-online.html` — how to vote via Nalo, plus a searchable
  name → Nalo code table
- `ctva-contact.html` — contact info

**Admin** (keep this link private — only share with your committee)
- `ctva-admin.html` — password-gated: approve/reject nominees + assign
  Nalo codes, log vote results, enter judges' scores

## 1. Update the backend

1. Open the same Google Sheet your registrations already go to.
2. Extensions > Apps Script.
3. **Select everything and delete it**, then paste in the full contents
   of the new `Code.gs`. (Your original registration logic is unchanged;
   everything new is clearly marked and appended below it.)
4. Save.
5. In the function dropdown, select **setupTvAwardsAdmin**, then **Run**.
   This:
   - Adds `Status`, `Nalo Code`, `Votes`, `Judge Score` columns to your
     `Registrations` sheet if missing (your 15 existing registrations
     are untouched — they'll just show as `pending`, ready to approve).
   - Creates a `VoteLog` sheet for vote-result history.
   - Sets a default admin password: **`ctva2026admin`** — change it
     under **Project Settings > Script Properties**, editing
     `ADMIN_PASSWORD`.
6. Re-deploy: **Deploy > Manage deployments > (pencil icon) > New
   version > Deploy**. Keeps the same URL — `tv_awards_register.html`
   needs no changes.

## 2. Connect every page to the backend

Already done — every data-driven page (`ctva-admin.html`,
`ctva-vote-results.html`, `ctva-nominees.html`,
`ctva-judges-ratings.html`, `ctva-vote-online.html`) is pre-filled with
the same Web App URL your live `tv_awards_register.html` already posts
to (`.../AKfycbxfTk.../exec`), so nothing to paste in. This is exactly
why it works automatically: every student who fills the real
registration form is writing into the same sheet these pages read
from. The only thing that makes the new fields (Status, Nalo Code,
Votes, Judge Score) actually exist is step 1 above — updating the
Apps Script behind that URL and re-deploying it.

## 3. Day-to-day workflow

1. **Registration** — unchanged, students keep using
   `tv_awards_register.html` exactly as now.
2. **`ctva-admin.html` → Pending** — review each registration, type in
   the **Nalo code** you've assigned, then **Approve** (blocked without
   a code, so nothing goes live unmatched).
3. **Set up voting on Nalo** with those same codes.
4. **`ctva-admin.html` → Log Votes** — on your schedule (daily works
   well), type in each approved nominee's current Nalo tally, pick the
   date, **Save Results**. Updates their live standing + appends to
   `VoteLog` history.
5. **`ctva-admin.html` → Judges' Scores** — enter/update each nominee's
   panel score (0–100) whenever judging happens. Shows instantly on the
   public Judges' Ratings page.
6. **Students check `ctva-vote-results.html`** anytime — type in their
   reference code to see their own status, vote count, rank, and judge
   score, without needing a password.
7. **Public can also browse `ctva-vote-results.html`'s live leaderboard**,
   `ctva-nominees.html`'s gallery, and `ctva-judges-ratings.html`.
8. **Your main `admin.html` CMG dashboard** — once winners are final,
   use its existing "Record a Program Result" feature to permanently
   attach the award to the student's NAAD record, same as always.

## Hosting

Any static host works for the public pages + admin page: GitHub Pages,
Google Sites (as embedded HTML), Netlify, or just the school's existing
web space. All pages are self-contained — no build step, no server
needed beyond the Apps Script backend.

## Adding the Nalo API later

When you get API access from Nalo, `logVotes_()` in `Code.gs` is what to
automate: call it on a schedule with
`{ action: 'logVotes', password, date, entries: [{ref, votes}, ...] }`
instead of typing numbers into the admin panel.

## Notes

- `setupTvAwardsAdmin()` is safe to re-run any time — only adds missing
  columns/sheets, never clears data.
- Nominee identity throughout is the same **Reference** code
  (`CTVA-2026-...`) your registration form already generates. This is
  also what students use to check their own result — remind them to
  keep it (it's already in their confirmation email).
- The admin password check is lightweight — fine for internal use, not
  bank-grade auth. Don't reuse a sensitive password for it.
