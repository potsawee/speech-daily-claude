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

2. GATHER. For each category in sources.yaml's arxiv_categories, query the arXiv API over HTTPS (https://export.arxiv.org/api/query) sorted by submittedDate descending, covering the last ~3–4 days (anchored to the most recent papers, to span weekends/holidays/a skipped run; dedup removes anything already shown). Fetch enough results per category (e.g., up to ~100) to cover the full window. Also scan huggingface.co/papers for entries tagged audio or speech. For each paper collect: title, authors, arXiv ID, abstract, and link. If a source host is blocked by network policy (HTTP 403 / host_not_allowed), fall back to the WebSearch tool and note in the digest that coverage is best-effort.

3. DEDUPE. Remove any paper whose arXiv ID is already in seen.json.

4. SCORE & RANK. Give each surviving paper two 0–10 scores:
   - **Relevance** — fit to the user: score against keywords_boost and interest_profile.md; drop or down-rank papers dominated by mute_keywords. Papers with an author in follow_authors (match on the **full first + last name**, never surname alone — many researchers share common surnames like Li, Chen, Wang, Zhang, so a surname-only match is unreliable) or a lab in follow_labs are auto-included and marked ★ (followed).
   - **Quality** — judge as an experienced AI/ML/Speech researcher deciding whether to share the paper in a peer/group chat: weigh the significance and ambition of the problem, genuine novelty (vs. an incremental tweak), rigor and credibility (strong baselines, ablations, honest or SOTA results, released code), the authors'/lab's track record, and clarity of the contribution; penalize overclaiming, weak or cherry-picked evaluation, and "yet another X" papers. These are fresh, unreviewed preprints, so treat Quality as an informed prior from the abstract — for the shortlist, skim the linked paper/code when it would change the call.
   Rank by an overall "would I recommend this to the user" judgment that combines both scores, with this asymmetry: do NOT surface a low-quality paper even if it is highly relevant, but DO allow a standout-quality paper onto the list even if only moderately relevant. When two papers are similar, prefer the more relevant one; break remaining ties by submission date.

5. OUTPUT. Produce the digest and deliver it **two ways**: (a) print it in full as your final session message, so the entire digest is readable directly in the Claude session without opening GitHub; and (b) write the identical content to a timestamped file digests/YYYY-MM-DD-HHMM.md on the state branch (UTC date-and-time, e.g. `date -u +%Y-%m-%d-%H%M`) so multiple runs on the same day don't overwrite each other. The digest content (the same in both places): the top 5 papers as a scannable list. For each paper: a header line with title, arXiv ID, and both scores shown as **Relevance R/10 · Quality Q/10**; an authors line listing authors by their **full name (first + last — never surname-only**, since many researchers share common surnames) followed by the paper's primary lab/company affiliation(s), the arXiv categories, the date, and a clickable `[Link](https://arxiv.org/pdf/<arXiv ID>)` pointing to the paper's arXiv PDF so the user can open it directly; then three italic-labeled lines, each on its own line — *Problem:* (what gap/limitation it tackles), *Method:* (the core approach), *Result:* (the headline finding or contribution) — together running ~70 words total, enough to grasp the paper in 1–2 minutes without opening it. End each entry with a bold **Why it ranked** line that justifies both the relevance and the quality call in one sentence. Aim for 5 papers, but don't pad below a "worth your time" quality bar — on a thin day show fewer (3–4) and say so. Highlight ★ followed papers at the top.

6. UPDATE STATE. Append all shown arXiv IDs to seen.json, pruning IDs older than ~90 days to keep the file bounded. Commit the updated seen.json and the new timestamped digest file, then push to the state branch:

       git add -A
       git commit -m "Digest <YYYY-MM-DD-HHMM>: <N> papers shown"
       git push origin claude/daily-state

If no new papers are found, say so briefly (and still commit/push any seen.json update).
