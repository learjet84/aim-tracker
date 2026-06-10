# AIM Partner Employer Job Tracker

A standing, auto-updating view of **open job requisitions at the employers that hire AIM graduates**, organized by campus. It gives the Industry Partnerships team one link to see what every partner employer is currently hiring for — and what changed since the last check — instead of visiting dozens of careers sites by hand.

**Live link:** https://learjet84.github.io/aim-tracker/ (landing page → pick a campus)

**Scope:** Rolling out to all AIM campuses. Live now: **AMA (Duluth, GA / Atlanta)** and **AMM (Manassas, VA)**, with the remaining campuses being added. Employer lists are drawn from the 2026 Hires YTD spreadsheet.

---

## Catchment prioritization (within ~100 miles)

Each campus has a ~100-mile catchment. Postings whose location falls inside that radius are:

- badged **📍 ~100 mi**,
- floated to the **top of each employer's list**, and
- collected in a **"Within ~100 miles of campus"** section at the very top of the dashboard, plus a header count.

Everything relevant is still listed — local roles are simply surfaced first to show priority. Proximity is inferred from the city/state in each posting's title or snippet against a curated city list per campus, so it applies to individual postings; full careers-board links (no single city) aren't geo-pinned.

---

## What it produces

| File | What it is |
| --- | --- |
| `site/index.html` | The multi-campus **landing page** (the shared link). Auto-generated. |
| `site/<code>.html` | Per-campus dashboard (e.g. `ama.html`, `amm.html`). Auto-generated copies of the dashboards. |
| `dashboard_<CODE>.html` | The working dashboard for a campus. |
| `openings_<CODE>.csv` | Flat export of every link + change status + a `within_100mi` column. |
| `snapshots/<CODE>_<date>.json` | One file per run — the tracker's memory; comparing the newest two powers new/closed detection. |
| `<code>_employers.json` | The master employer list for that campus (name + YTD hires). |
| `employer_cache.json` | Global cache of already-pulled employer results, so shared employers are searched once and reused across campuses. |
| `pipeline.py` | Engine: classifies links, tags catchment, snapshots, diffs, renders dashboard + CSV. |
| `assemble.py` | Builds `raw_pull_<CODE>.json` from the cache and reports which employers still need searching. |
| `build_index.py` | Rebuilds the landing page + per-campus `site/*.html`. |
| `RUNBOOK.md` / `GITHUB_SETUP.md` | Operator steps and one-time publishing setup. |

---

## How a run works

For each campus:

1. **Pull** — for every employer, a live web search (Nimble) finds current openings across the employer's careers board, ATS (Workday, Greenhouse, iCIMS, Breezy, etc.), and aggregators. Shared employers are reused from `employer_cache.json` so they're searched once.
2. **Classify & geo-tag** — each link is tagged **req / careers board / aggregator**, and flagged **📍 within ~100 mi** if its location is in the campus catchment.
3. **Snapshot** — results saved as a timestamped file in `snapshots/`.
4. **Diff** — compared to the previous snapshot: links not seen last time are **NEW**; links gone since last time are **CLOSED** (likely filled/pulled).
5. **Render** — dashboards + CSV rebuilt; then `build_index.py` rebuilds the landing page.

### Reading a dashboard
- **📍 ~100 mi** (teal) — posting within the campus catchment; these sort to the top.
- **Green NEW** / **Red CLOSED** badges — change since the last run (from the 2nd run on).
- **req / careers board / aggregator** — what kind of link it is.
- Header stats: employers, live links, within ~100 mi, new, closed.

---

## Update cadence

- **Automatic:** every **Monday, Wednesday, and Friday ~6:00 AM** local. The job loops **every campus** that has an employer list, rebuilds all dashboards + the landing page.
- Change tracking is meaningful from the **second** run onward (the first sets a baseline).
- Runs happen while the Claude desktop app is open; if it's closed when due, it runs on next launch.

## Running an on-demand report

The pull needs the Nimble connection, so on-demand runs go through Claude:

1. **From the app:** sidebar → **Scheduled** → **aim-ama-job-tracker** → **Run now**.
2. **Just ask:** "run the job tracker now" (optionally name a campus).

> `pipeline.py` alone only re-renders from the latest pull; it does **not** fetch new jobs — fetching is the Nimble step Claude runs.

## Publishing updates

After a run, the changed files in `site/` need to be pushed to the `aim-tracker` GitHub repo so the live link refreshes:
- **Web:** repo → Add file → Upload files → drag the `site/` files → Commit.
- **GitHub Desktop:** Commit → Push.
- Or ask Claude to drive the upload in the browser.

Free GitHub Pages serves a public-but-unlisted URL — keep the link internal. Hard access control would need a different host.

---

## Data integrity

- **No fabricated data.** Every link comes from a live search result. If Nimble is disconnected/expired, a run stops and reports it rather than inventing jobs.
- **Coverage:** most employers resolve to a real careers board or postings; tiny shops without an applicant-tracking system are annotated for manual follow-up.
- **Fidelity:** employer, title, location, and link are reliable. Exact req IDs and posting dates are captured only where the source exposes them.

---

## Campus codes (authoritative, from Affiliate Schools doc rev 08/2022)

AMA = Duluth GA (Atlanta) · AMC = Charlotte NC · AMD = Irving TX · AMH = Houston TX · AMI = Indianapolis IN ·
AMK = Kansas City MO · AML = Las Vegas NV · AMM = Manassas VA · AMN = Norfolk VA · AMO = Casselberry FL (Orlando) ·
AMP = Philadelphia PA · AMS = Fremont CA · AMT = Teterboro NJ · AMW = Chicago IL. *(AMX appears in the hires spreadsheet but not the affiliate doc — confirm its location before adding.)*

To add a ca