# HANDOFF — Instructions for Claude Code

## Context

Hi Claude Code. This is a compliance tracker built for **Candice Hesseling** (Founded To Be Counted, AI CFO Candice). Candice is a non-developer fractional CFO managing compliance for ~5 client companies across AU/NZ/US. We built this as a single-file static webapp during a chat conversation. It currently works but lives inside a Claude.ai artifact, which means she can't bookmark it as a real website.

**Your job: get this deployed as a real website she can bookmark, and set up the GitHub repo so we can iterate on it.**

## What's in this folder

- `index.html` — the entire app, single-file. Already converted from `window.storage` (Claude artifacts only) to browser `localStorage` (works as a normal website). Everything works as-is when opened in a browser.
- `README.md` — user-facing overview
- `HANDOFF.md` — this file
- `compliance_library.csv` — source data for the compliance library (the LIBRARY constant in index.html was generated from this; kept here for reference / future tooling)

## Primary task: deploy it

Recommended deployment: **GitHub Pages** (free, simple, matches Candice's request to use your GitHub CLI).

### Step 1: Create the repo

```bash
gh repo create compliance-tracker --public --source=. --remote=origin --push
```

Or if she prefers private:

```bash
gh repo create compliance-tracker --private --source=. --remote=origin --push
```

### Step 2: Enable GitHub Pages

```bash
gh api -X POST repos/{owner}/{repo}/pages -f source[branch]=main -f source[path]=/
```

(Replace `{owner}` and `{repo}` with the actual values, or use `gh repo view` to grab them.)

Alternatively, in the GitHub web UI: Settings → Pages → Source: "Deploy from a branch" → Branch: `main` / `/ (root)` → Save.

### Step 3: Confirm the URL

Within 1–2 minutes the site will be live at `https://{owner}.github.io/compliance-tracker/`. Tell Candice the URL and ask her to bookmark it on every device she'll use.

### Alternative: Netlify Drop (if Pages is being slow)

```bash
# from this directory
npx netlify-cli deploy --prod --dir .
```

Or just drag `index.html` onto https://app.netlify.com/drop — gives a URL in seconds.

## Important things to know

### Storage model

Data lives in `localStorage` on whichever browser is being used. This means:

- **Per-device, per-browser.** Phone Safari and laptop Chrome have separate copies.
- **Persists across refreshes and browser restarts.**
- **Cleared if she clears site data.**
- **5–10MB capacity.** Way more than this app needs.

The Share tab has "Download backup" (JSON) and "Import backup" buttons so she can move data between devices manually. That's the current cross-device story.

### Existing data in her current artifact version

Candice has been using the artifact version for a while and has accumulated data: 5 companies (Dispute Buddy AU, Dispute Buddy NZ, Clutch Glue Pty, Clutch Glue LLC, Founded To Be Counted), with various due-date adjustments and deletions.

**That data does NOT migrate automatically.** When she opens the new deployment, it will seed with default companies. To bring her data over:

1. Tell her to open her Claude.ai conversation, find the artifact, go to Share tab, click "Download full backup" — she gets a JSON file
2. On the new site, go to Share tab → "Import backup" → select that JSON file
3. Confirm the import

She'll likely forget to do this; remind her early.

### Known compliance library coverage

Pre-populated requirements in `LIBRARY` constant inside `index.html`:

- **AU (15 items)** — ASIC obligations, ATO tax (income tax, BAS, FBT, STP, super, PAYG), state taxes (payroll, workers comp), AML, climate disclosures, beneficial ownership
- **NZ (16 items)** — Companies Office (annual return, director changes), IRD (IR4, provisional, GST, FBT, RWT, NRWT, transfer pricing), payroll (PAYE, KiwiSaver, ACC), audit, AML, SPFR
- **US (12 items)** — IRS (1120, 941, 940, W2, 1099-NEC, 5472), Delaware franchise tax, state-level (annual report, sales tax, SUI, workers comp)

Source CSV is in `compliance_library.csv`. If you want to extend the library, edit the `LIBRARY` constant in `index.html` directly — it's a JS object literal, easy to grok.

### Auto-spawn behaviour for recurring tasks

When user marks a task done, if the requirement frequency is recurring (quarterly/monthly/annual/etc.), a new task is auto-created with a calculated due date. The calculation is in `suggestDueDate()` and is approximate — for statutory dates that fall on specific calendar days (e.g., NZ provisional tax due 28 Aug / 15 Jan / 7 May), the auto-spawned date will need manual adjustment. This is a known limitation.

### Specific FTBC migration

There's a `runMigrations()` function with a one-time migration that adds Founded To Be Counted (her own company) with a specific 6-monthly GST cadence and a hardcoded 28 July 2026 due date. This was set up because she gave us that exact info during the chat. Don't remove this — it's useful; it just needs to not run twice (and it doesn't, due to the migrations_done flag).

## Optional enhancements (in priority order)

If she asks for any of these, here's how to approach them:

### 1. Cloud sync across devices (highest value)

Currently data is per-browser. To sync across her phone and laptop:

- Easiest: **Supabase** free tier. Add a `lib/supabase.js` module, create a `compliance_data` table with a single row keyed to her user, store the entire state JSON in it. Add login via magic link. ~100 lines of code.
- Alternative: **Firebase Realtime Database** with anonymous auth.
- Alternative: **GitHub Gist** as backend (auth via personal access token, store JSON in a private gist). Hacky but zero ongoing cost.

Recommendation: Supabase. It scales, free tier is generous, and she's a single user so usage will be trivial.

### 2. Email/SMS reminders that actually fire from a server

Currently reminders rely on importing the .ics into her phone calendar. If she wants reminders without the calendar step:

- Add a Cloudflare Worker (free) on a daily cron schedule
- Worker reads her data (requires cloud sync from #1) and sends emails via Resend (free 3000/mo) or SMS via Twilio (paid)
- Reminder windows: 14 days, 7 days, 1 day before, plus daily for overdue

### 3. Per-task frequency override + statutory date awareness

Right now requirement.frequency is shared across all instances. To support per-task overrides (e.g., "this BAS instance is for Q2 ending 30 June, due 28 July"):

- Add a `recurrenceOverride` field on tasks
- Update `completeTask()` auto-spawn to use it
- Add a "next due date" picker in the task edit modal

For statutory date awareness (knowing that BAS is due specifically 28 Apr / 28 Jul / 28 Oct / 28 Feb):

- Add a `statutoryDates` array to LIBRARY items where applicable
- `suggestDueDate()` picks the next statutory date after `Date.now()`

### 4. Multi-user / client-facing read-only views

If she wants to share a client-specific dashboard without giving them edit access:

- Add a per-company sharing token (UUID)
- A `/view/{token}` route serves a read-only dashboard for that company only
- Requires cloud sync from #1

### 5. Better mobile UI

The current layout is responsive but tight on small screens. Worth a pass at some point — bigger touch targets, simplified portfolio view on phone.

## What NOT to do

- Don't switch frameworks (React/Vue/etc.) without good reason. The single-file vanilla JS approach is intentional — it makes the app trivially deployable, debuggable, and modifiable by a non-developer. Adding a build step adds friction.
- Don't introduce npm dependencies unless absolutely needed. Currently zero deps.
- Don't change storage keys casually — `compliance_tracker_*` prefix is set in `STORAGE_PREFIX`. If you change it, existing user data is invisible to the new code.
- Don't refactor for the sake of it. The code is small (~1100 lines, lots of HTML strings) and Candice can read it with you.

## Questions for Candice (ask before deploying)

1. Public or private GitHub repo? (Public is fine — there's no sensitive data in the code, just compliance metadata.)
2. Custom domain, or is `{owner}.github.io/compliance-tracker` fine for now?
3. Does she want help importing her existing artifact data right after deploy?

## Contact

This handoff was generated from a conversation between Candice and Claude (the chat assistant). The full conversation context is in her Claude.ai history if you need historical decisions. Ping her if anything is unclear.

Good luck.
