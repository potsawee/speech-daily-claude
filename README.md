# Speech-Daily

A morning scout routine that finds and ranks new speech & audio research papers.
It runs as a scheduled Claude Code routine; all state lives in this Git repo (no
external storage) so it survives scheduled, non-interactive runs.

**📱 Read the digests:** <https://potsawee.github.io/speech-daily-claude/> —
always opens the latest digest, with a month-grouped archive of past runs.
(Raw files: [`claude/daily-state` → `digests/`](https://github.com/potsawee/speech-daily-claude/tree/claude/daily-state/digests).)

## Branch layout (why there are two)

Routines can only `git push` to `claude/`-prefixed branches, not to `main`. So
state is split across two branches:

| Branch | Holds | Written by |
|--------|-------|-----------|
| **`main`** (default) | Config (`sources.yaml`, `interest_profile.md`), docs, web reader (`docs/`) | **You** |
| **`claude/daily-state`** | `seen.json`, `digests/` | The routine |

Each run reads config from `main`, reads/writes state on `claude/daily-state`,
and pushes only to that branch — so it needs no special push permission.

## Files

| File | Role | Lives on | Edited by |
|------|------|----------|-----------|
| `routine.md` | **Canonical routine prompt** (see [Updating the live routine](#updating-the-live-routine)) | `main` | **You** |
| `sources.yaml` | Config: arXiv categories, boost/mute keywords, followed authors & labs | `main` | **You** |
| `interest_profile.md` | Learned preferences used for scoring | `main` | You + routine |
| `docs/index.html` | Web reader served by GitHub Pages (see [Web reader](#web-reader-github-pages)) | `main` | You |
| `seen.json` | Flat array of arXiv IDs already shown (dedupe). Pruned to ~last 90 days. | `claude/daily-state` | Routine |
| `digests/YYYY-MM-DD-HHMM.md` | Archived ranked digest per run (UTC timestamp; same-day re-runs don't clash) | `claude/daily-state` | Routine |

## Daily flow

1. **Set up branch** — fetch/checkout `claude/daily-state` (created from `main` on first run); refresh config from `main`.
2. **Load state** — read `sources.yaml`, `interest_profile.md`, `seen.json`.
3. **Gather** — collect papers from a ~3–4-day window (spans weekends/holidays/skipped runs; dedupe absorbs the overlap) through a reliability ladder:
   1. **arXiv API** (primary, HTTPS) — single-threaded, spaced requests; on 429/503, back off and keep retrying politely for up to ~40 minutes.
   2. **arXiv listing pages via WebFetch** (`arxiv.org/list/<category>/recent`) if the API stays down or is hard-blocked.
   3. **WebSearch** as a last resort.

   Plus a scan of `huggingface.co/papers` for audio/speech entries. Any fallback is flagged in the digest as best-effort coverage.
4. **Dedupe** — drop IDs already in `seen.json`.
5. **Score & rank** — two 0–10 scores per paper: **Relevance** (fit to `keywords_boost` + `interest_profile.md`; `mute_keywords` down-rank) and **Quality** (novelty, rigor, results, track record). Followed authors (matched on full first + last name) and labs are auto-included, marked ★, pinned to top. Asymmetry: low-quality-but-relevant is dropped; standout-quality-but-moderately-relevant may still make the list.
6. **Output** — top 5 (fewer on a thin day) as a scannable digest. Per paper: both scores, full author names + primary lab affiliation, categories, date, a clickable **[Link]** to the arXiv PDF, *Problem / Method / Result* lines (~70 words total), and a one-sentence **Why it ranked**. Printed in full in the session **and** archived under `digests/`; a footer reports the run's active reasoning-effort level.
7. **Update state** — append shown IDs to `seen.json` (pruned to ~90 days), commit, **push to `claude/daily-state`**.

**Completion guarantee:** a run only counts as done once a digest is committed
and pushed to `claude/daily-state`. If gathering degrades, the routine still
pushes a clearly-labelled degraded digest; if it truly cannot produce one, it
pushes whatever state it can and sends a push notification instead of failing
silently.

## Web reader (GitHub Pages)

<https://potsawee.github.io/speech-daily-claude/> is a single static page
([`docs/index.html`](docs/index.html)) served by GitHub Pages from `main` /
`docs`. It fetches the digest list and content from `claude/daily-state`
client-side via the GitHub API, so:

- new digests appear on a plain refresh — no rebuild, fully decoupled from the routine (Pages only rebuilds when `docs/` itself changes);
- the latest digest opens by default; the archive dropdown is grouped by month;
- ◀ / ▶ buttons and arrow keys step between digests; `#YYYY-MM-DD-HHMM` deep-links a specific run;
- dark/light theme follows the system, with a manual toggle.

> Relies on the repo staying **public** — both the anonymous GitHub API reads
> and free GitHub Pages require it.

## Network access

The routine's environment uses **Full** network access, so it reaches the live
arXiv API (`export.arxiv.org/api/query`) and `huggingface.co` directly — digests
are date-accurate, category-filtered, and cover the full window.

> **Note:** call the arXiv API over **HTTPS** (`https://export.arxiv.org/...`).
> The environment proxy only tunnels HTTPS; a plain `http://` request returns 403.

## Updating the live routine

The scheduled routine runs the prompt stored in its **Instructions** box at
[claude.ai/code/routines](https://claude.ai/code/routines) — it does **not**
read `routine.md` at runtime; the file is the canonical, version-controlled
copy. After editing `routine.md`, paste the prompt block into the Instructions
box (pencil icon → Edit routine) so the change takes effect on the next run.
