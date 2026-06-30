# Speech-Daily

A morning scout routine that finds and ranks new speech & audio research papers.
State lives in this Git repo (no external storage) so it survives scheduled,
non-interactive runs.

## Branch layout (why there are two)

Routines can only `git push` to `claude/`-prefixed branches, not to `main`. So
state is split across two branches:

| Branch | Holds | Written by |
|--------|-------|-----------|
| **`main`** (default) | `sources.yaml`, `interest_profile.md`, docs | **You** |
| **`claude/daily-state`** | `seen.json`, `digests/` | The routine |

Each run reads config from `main`, reads/writes state on `claude/daily-state`,
and pushes only to that branch — so it needs no special push permission.

**📅 Browse digests:** [`claude/daily-state` → `digests/`](https://github.com/potsawee/speech-daily-claude/tree/claude/daily-state/digests)

## Files

| File | Role | Lives on | Edited by |
|------|------|----------|-----------|
| `sources.yaml` | Config: arXiv categories, boost/mute keywords, followed authors & labs | `main` | **You** |
| `interest_profile.md` | Learned preferences used for scoring | `main` | You + routine |
| `seen.json` | Flat array of arXiv IDs already shown (dedupe). Pruned to ~last 90 days. | `claude/daily-state` | Routine |
| `digests/YYYY-MM-DD.md` | Archived ranked digest per run | `claude/daily-state` | Routine |

## Daily flow

1. **Set up branch** — fetch/checkout `claude/daily-state` (created from `main` on first run); refresh config from `main`.
2. **Load state** — read `sources.yaml`, `interest_profile.md`, `seen.json`.
3. **Gather** — query recent papers for each category in `sources.yaml` + scan HuggingFace audio/speech.
4. **Dedupe** — drop IDs already in `seen.json`.
5. **Rank** — auto-include followed authors/labs (★, pinned to top); score the rest against `keywords_boost` + `interest_profile.md`; down-rank `mute_keywords`.
6. **Output** — top 5 as a scannable digest, archived under `digests/`.
7. **Update state** — append shown IDs to `seen.json`, commit, **push to `claude/daily-state`**.

## Network access

The routine's environment uses **Full** network access, so it reaches the live
arXiv API (`export.arxiv.org/api/query`) and `huggingface.co` directly — digests
are date-accurate, category-filtered, and cover a true ~2-day window.

> **Note:** call the arXiv API over **HTTPS** (`https://export.arxiv.org/...`).
> The environment proxy only tunnels HTTPS; a plain `http://` request returns 403.

As a safety net, the routine prompt still falls back to the WebSearch tool if a
host is ever blocked (HTTP 403 / `host_not_allowed`), and notes in the digest when
coverage is best-effort rather than a complete sweep.
