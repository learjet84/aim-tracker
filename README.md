# AIM Partner Employer Job Tracker

A standing, auto-updating view of **open job requisitions at the employers that hire AIM graduates**, organized by campus. It exists so the Industry Partnerships team has one place to see what every partner employer is currently hiring for — and what changed since the last check — instead of visiting dozens of careers sites by hand.

**Current scope:** Pilot on the **AMA campus (AIM – Duluth, GA / Atlanta metro)** — 54 partner employers drawn from the 2026 Hires YTD spreadsheet. The same machinery extends to the other 13 campuses without rework.

---

## What it produces

| File | What it is |
| --- | --- |
| `dashboard_AMA.html` | The dashboard. Open in any browser. Employers grouped and sorted by grad-hire volume, each with click-through links to live openings and change badges. |
| `site/index.html` | A copy of the dashboard named for web hosting (GitHub Pages). This is the file that becomes the shareable team link. |
| `openings_AMA.csv` | Flat export of every link with its change status — for pivoting, filtering, or pasting into a deck. |
| `snapshots/AMA_<date>.json` | One file per run. This is the tracker's memory; comparing the newest two is what powers new/closed detection. |
| `ama_employers.json` | The master list of the 54 AMA employers and their YTD hire counts. |
| `pipeline.py` | The engine: classifies links, writes the snapshot, diffs against the prior run, rebuilds the dashboard + CSV. |
| `RUNBOOK.md` | Operator steps for a run (used by the automated job). |
| `GITHUB_SETUP.md` | One-time steps to publish the dashboard to a shareable link. |

---

## How it works

Each run does five things:

1. **Pull** — For every employer in `ama_employers.json`, a live web search (via Nimble) finds current openings across the employer's own careers board, its applicant-tracking system (Workday, Greenhouse, iCIMS, etc.), and major aggregators.
2. **Classify** — Each result is tagged: **req** (an individual posting), **careers board** (the employer's full openings page), or **aggregator** (Indeed/LinkedIn/ZipRecruiter/etc.).
3. **Snapshot** — Results are saved as a timestamped file in `snapshots/`.
4. **Diff** — The new snapshot is compared to the previous one. Anything not seen last time is flagged **NEW**; anything present last time but gone now is flagged **CLOSED** (likely filled or pulled).
5. **Render** — The dashboard and CSV are rebuilt with the latest data and change badges.

### Reading the dashboard
- **Green NEW badge** — a posting/link that appeared since the last run.
- **Red CLOSED badge** (struck through) — a posting that disappeared since the last run.
- **req / careers board / aggregator** — the kind of link, so you know whether you're clicking a single job or a full board.
- The header shows totals: employers tracked, live links, and new/closed counts for the run.

---

## Update cadence

- **Automatic:** every **Monday, Wednesday, and Friday at ~6:00 AM** local time.
- Change tracking (new/closed) is meaningful from the **second** run onward — the first run only sets the baseline.
- Automated runs happen while the Claude desktop app is open. If the app is closed when a run is due, it runs the next time the app is opened.
- After each run, **publish the update** so the team link reflects it: drag `site/index.html` into the GitHub repo on the web, or in GitHub Desktop click **Commit → Push** (about two clicks). See `GITHUB_SETUP.md`.

---

## Running an on-demand report

The data pull requires the Nimble connection, so on-demand runs go through Claude (not by double-clicking a file). Two ways:

1. **From the app:** open the **Scheduled** section in the sidebar, find **aim-ama-job-tracker**, and click **Run now**.
2. **Just ask:** tell Claude *"run the AMA job tracker now."*

Either one re-pulls all 54 employers, updates the snapshot, recomputes new/closed against the last run, and rebuilds the dashboard + CSV. Then publish as above to push it live.

> `pipeline.py` on its own only re-processes the most recent pull (re-renders the dashboard from existing data). It does **not** fetch new jobs — fetching is the Nimble step that Claude runs.

---

## Sharing the dashboard

The dashboard is hosted as a web page so anyone can open it at a link. Setup is in `GITHUB_SETUP.md`; the team link looks like `https://learjet84.github.io/aim-tracker/`. On free GitHub the page is public-but-unlisted — keep the link internal. Hard access control would require a different host.

---

## Data integrity

- **No fabricated data.** Every link comes from a live search result. If the Nimble connection is expired, a run stops and reports it rather than inventing jobs.
- **Coverage:** 48 of 54 AMA employers resolve to a real careers board or postings. Six small shops (Airenelle, Ellison Manufacturing, Fullard Flight, AirStar Flight Support, Bynum & Sons, Tactical Workforce Solutions) don't surface a dedicated jobs page and are flagged in the dashboard for manual confirmation.
- **Fidelity:** employer, title, location, and link are reliable. Exact requisition IDs and posting dates are captured only where the source exposes them. A later high-fidelity pass (dedicated Workday/Greenhouse agents for top employers) can tighten this.

---

## Campus codes (for rollout)

AMA = Duluth GA (Atlanta) · AMC = Charlotte NC · AMD = Irving TX · AMH = Houston TX · AMI = Indianapolis IN · AMK = Kansas City MO · AML = Las Vegas NV · AMM = Manassas VA · AMN = Norfolk VA · AMO = Casselberry FL (Orlando) · AMP = Philadelphia PA · AMS = Fremont CA · AMT = Teterboro NJ · AMW = Chicago IL. *(AMX appears in the hires spreadsheet but not the affiliate-schools doc — confirm its location before adding.)*

To add a campus: build its `<CODE>_employers.json`, run the same pull, then `python3 pipeline.py <CODE>`. Each campus keeps its own snapshot history and dashboard.
