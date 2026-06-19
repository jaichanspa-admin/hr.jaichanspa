# JAI CHAN Spa

This repository contains the JAI CHAN Spa web app and the Siam Discovery therapist payout dashboard.

The therapist payout dashboard is updated from a ThaiHand XLSX report that the user exports manually each day. The old POS automation flow is no longer the supported workflow because repeated ThaiHand login/browser/network issues made it unreliable.

## What This Project Publishes

- Main spa frontend assets built with Vite/React.
- `TherapistPayoutApp.html`, the therapist payout dashboard built from `skill/raw_data.json`.
- A privacy-safe GitHub Pages build for HR payout review:
  - Live URL: <https://jaichanspa-admin.github.io/hr.jaichanspa/>
  - Public build pseudonymizes/removes customer names.
  - Do not publish the internal full-data build unless explicitly instructed.

## Daily Payout Update Workflow

1. Export the fresh ThaiHand report manually as XLSX.
2. Save it in Downloads with the ThaiHand filename, for example:

```text
C:\Users\JCT-01\Downloads\report_Jai Chan-Siam Discovery_2026-06-01-2026-06-18.xlsx
```

3. Run a dry-run first with explicit expected dates:

```powershell
python scripts\update_therapist_payout.py --xlsx "C:\Users\JCT-01\Downloads\report_Jai Chan-Siam Discovery_YYYY-MM-DD-YYYY-MM-DD.xlsx" --expected-start YYYY-MM-DD --expected-end YYYY-MM-DD --dry-run --json
```

4. Review the dry-run output before applying:

- `parser_diagnostics.header_valid` must be `true`.
- Sheet index 0 must be named `ภาพรวม`.
- Parsed date range must match `--expected-start` and `--expected-end`.
- `dry_run_report` should show the affected date range, before/after totals, added/removed/unchanged rows, payout delta, guarantee top-ups, unknown channels, missing calendar dates, blank customers, zero payouts, per-date deltas, and channel/therapist breakdowns.
- Skipped rows must be reviewed by reason and sample rows. The recurring known source issue is usually one `missing_therapist` row for `shampoo & Conditioner`.

5. If the dry-run passes, apply and publish the privacy-safe build:

```powershell
python scripts\update_therapist_payout.py --xlsx "C:\Users\JCT-01\Downloads\report_Jai Chan-Siam Discovery_YYYY-MM-DD-YYYY-MM-DD.xlsx" --expected-start YYYY-MM-DD --expected-end YYYY-MM-DD --publish --json
```

The publish step clones `jaichanspa-admin/hr.jaichanspa`, copies the scrubbed privacy-safe HTML to `index.html`, commits, and pushes `main` through git HTTPS using the token in `skill/github_token.txt`.

## Validation Rules

Treat these as blockers. Do not apply, rebuild, publish, or push if any occur:

- The XLSX file is missing, stale, empty, or not the intended date range.
- The workbook sheet is wrong or headers fail validation.
- Parsed records are empty.
- The parsed date range does not match the expected start/end dates.
- The parser reports schema/header errors.
- Rebuild validation or publish validation fails.
- Unexpected future dates appear in the parsed data.

Non-fatal source warnings should still be reported:

- Unknown booking channels.
- Blank customer values.
- Zero payout records.
- Skipped rows by reason and sample skipped rows.

## Guarantee Rules

Guarantee/top-up rules are loaded from:

```text
skill/guarantee_config.json
```

The parser and UI must use this same source. Do not hard-code guarantee rules elsewhere.

## Privacy And Metadata

Successful updates inject `APP_META` into the HTML artifact with:

- Generation time and timezone.
- Source type, source filename, and source SHA-256.
- Parsed date range, parsed count, therapist count, and parsed payout total.
- Merged date range and total record count.
- Guarantee top-ups.
- Privacy build/publish status.

Public GitHub Pages publishing must use the default privacy-safe mode. Real customer names must not be published publicly.

## Local Development

Install and build the main web app:

```powershell
npm.cmd install
npm.cmd run build
npm.cmd run preview -- --port 8787
```

Open the preview URL printed by Vite, usually:

```text
http://127.0.0.1:8787/
```

Create a standalone HTML build for the main spa frontend:

```powershell
npm.cmd run build:standalone
```

Output:

```text
dist/jai-chan-spa-standalone.html
```

## Environment

Copy `.env.example` values into the hosting environment as needed.

- `VITE_BOOKING_URL`: external destination for Book Now / Buy Package / Join Membership buttons.

ThaiHand POS credentials are intentionally not required for the current daily payout workflow because the user exports the XLSX manually.

## Deprecated POS Automation

The following flows are deprecated for daily operations:

- `npm run thaihand:export`
- `npm run thaihand:update-daily`
- `scripts/thaihand_pos_export.mjs`
- Browser/XHR capture through `scripts/thaihand_export_capture.js`

They are left in the repository for reference and future debugging only. Do not use them as the production daily update path unless the POS automation is explicitly repaired and re-approved.
