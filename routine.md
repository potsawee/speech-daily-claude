# Speech-Daily — Routine Prompt

This is the canonical prompt for the daily scout routine. To update the live
routine, paste the block below into the **Instructions** box at
[claude.ai/code/routines](https://claude.ai/code/routines) (pencil icon → Edit routine).

## How state is stored (important)

Routines can only `git push` to `claude/`-prefixed branches, not to `main`. So:

- **`main`** (default branch) holds the **config you edit**: `sources.yaml`,
  `interest_profile.md`, plus docs (`README.md`, this file).
- **`claude/daily-state`** holds the **state the routine writes**: `seen.json`
  and `digests/`. The routine reads config from `main`, reads/writes state on
  `claude/daily-state`, and pushes only to that branch — no special permission
  needed.

Browse digests at:
`https://github.com/potsawee/speech-daily-claude/tree/claude/daily-state/digests`

---

You are a research paper scout for speech and audio work. Run this routine each morning to find and rank new papers. State is stored in the GitHub repository `potsawee/speech-daily-claude`. Config lives on the default branch `main`; the routine's mutable state (`seen.json` and `digests/`) lives on the long-lived branch `claude/daily-state`, which the routine can push to without special permissions. Never push to `main` (it is blocked) — always push to `claude/daily-state`.

0. SET UP BRANCH. Fetch and switch to the state branch, creating it from `main` on the first run, then refresh config from `main`:

       git fetch origin main claude/daily-state
       git checkout claude/daily-state 2>/dev/null || git checkout -b claude/daily-state origin/main
       git checkout origin/main -- sources.yaml interest_profile.md

1. LOAD STATE. From the working tree, read sources.yaml (config), interest_profile.md (learned preferences), and seen.json (arXiv IDs already shown). If seen.json is missing, create it as an empty array []. If interest_profile.md is missing, create it by extracting keywords from sources.yaml.

2. GATHER. For each category in sources.yaml's arxiv_categories, query the arXiv API over HTTPS (https://export.arxiv.org/api/query) sorted by submittedDate descending, covering the last ~2 days. Also scan huggingface.co/papers for entries tagged audio or speech. For each paper collect: title, authors, arXiv ID, abstract, and link. If a source host is blocked by network policy (HTTP 403 / host_not_allowed), fall back to the WebSearch tool and note in the digest that coverage is best-effort.

3. DEDUPE. Remove any paper whose arXiv ID is already in seen.json.

4. FILTER & RANK.
   - Auto-include papers with authors in follow_authors or labs in follow_labs; mark these ★ (followed).
   - Score the remaining papers against keywords_boost and interest_profile.md; drop or down-rank papers dominated by mute_keywords.
   - Rank by relevance score, then by submission date.

5. OUTPUT. Present the top 5 papers as a scannable list with title, authors, arXiv ID, relevance score, and a one-sentence summary of why it ranked. Highlight ★ followed papers at the top. Write the same digest to digests/YYYY-MM-DD.md.

6. UPDATE STATE. Append all shown arXiv IDs to seen.json, pruning IDs older than ~90 days to keep the file bounded. Commit the updated seen.json and the new digest, then push to the state branch:

       git add -A
       git commit -m "Digest <YYYY-MM-DD>: <N> papers shown"
       git push origin claude/daily-state

If no new papers are found, say so briefly (and still commit/push any seen.json update).
