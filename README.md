# Compliance Tracker

A single-file static webapp for tracking compliance obligations across multiple companies and jurisdictions (Australia, New Zealand, United States).

## Features

- **Portfolio view** — master calendar showing all deadlines across all companies, color-coded per company
- **Per-company dashboard** — overdue/upcoming stats, next 4 deadlines per company
- **Pre-populated compliance library** — 43 jurisdiction-specific requirements (15 AU, 16 NZ, 12 US) auto-applied when you add a company
- **Recurring task auto-spawn** — completing a quarterly/annual task automatically generates the next instance
- **Calendar export (.ics)** — import into Google/Apple/Outlook calendars with built-in 14d/7d/1d alarms
- **Printable client reports** — clean PDF-ready snapshots per client
- **CSV exports** — for spreadsheet analysis or sharing with bookkeepers
- **JSON backup & restore** — portable data, move between devices/browsers
- **Deletion tracking** — deleted requirements stay deleted (don't get re-added by the library sync)
- **Custom requirements per company** — extend the standard library when needed

## Tech

- Single HTML file, no build step, no dependencies
- Vanilla JavaScript (no framework)
- localStorage for persistence (per-browser, per-device)
- Designed to be deployed as a static site

## Run locally

Just open `index.html` in any modern browser. Data persists in that browser's localStorage.

## Deploy

See `HANDOFF.md` for deployment instructions and recommended next steps.
