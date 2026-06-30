# Speech-Daily

A morning scout routine that finds and ranks new speech & audio research papers.
State lives in this Git repo (no external storage) so it survives scheduled,
non-interactive runs.

## Files

| File | Role | Edited by |
|------|------|-----------|
| `sources.yaml` | Config: arXiv categories, boost/mute keywords, followed authors & labs | **You** |
| `interest_profile.md` | Learned preferences used for scoring | You + routine |
| `seen.json` | Flat array of arXiv IDs already shown (dedupe). Pruned to ~last 90 days. | Routine |
| `digests/YYYY-MM-DD.md` | Archived ranked digest per run | Routine |

## Daily flow

1. **Load state** — read `sources.yaml`, `interest_profile.md`, `seen.json`.
2. **Gather** — query recent papers for each category in `sources.yaml` + scan HuggingFace audio/speech.
3. **Dedupe** — drop IDs already in `seen.json`.
4. **Rank** — auto-include followed authors/labs (★, pinned to top); score the rest against `keywords_boost` + `interest_profile.md`; down-rank `mute_keywords`.
5. **Output** — top 5 as a scannable digest, archived under `digests/`.
6. **Update state** — append shown IDs to `seen.json`, commit, push.

## Network access

The routine's environment uses **Full** network access, so it reaches the live
arXiv API (`export.arxiv.org/api/query`) and `huggingface.co` directly — digests
are date-accurate, category-filtered, and cover a true ~2-day window.

> **Note:** call the arXiv API over **HTTPS** (`https://export.arxiv.org/...`).
> The environment proxy only tunnels HTTPS; a plain `http://` request returns 403.

As a safety net, the routine prompt still falls back to the WebSearch tool if a
host is ever blocked (HTTP 403 / `host_not_allowed`), and notes in the digest when
coverage is best-effort rather than a complete sweep.
