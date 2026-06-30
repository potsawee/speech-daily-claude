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
5. **Output** — top 10–15 as a scannable digest, archived under `digests/`.
6. **Update state** — append shown IDs to `seen.json`, commit, push.

## ⚠️ Known limitation: network policy blocks arXiv

This environment's egress policy **denies outbound to `arxiv.org`, `export.arxiv.org`,
`huggingface.co`, and `api.semanticscholar.org`** (403 policy denial). The routine's
intended primary source — the arXiv API — is therefore unreachable, and gathering
falls back to the server-side WebSearch tool. Consequences:

- Coverage is **best-effort**, not a complete 48-hour sweep.
- Submission dates are **approximate** (inferred from arXiv IDs).

**To restore full coverage:** allowlist `arxiv.org` and `export.arxiv.org` in the
environment's network settings (see https://code.claude.com/docs/en/claude-code-on-the-web).
Once allowed, the routine can hit `export.arxiv.org/api/query` directly with proper
date-sorted, category-filtered results.
