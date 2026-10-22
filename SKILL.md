---
name: daily-pain-finder
description: Daily ~10 PM PT whole-web pain-point finder with a self-improving critic loop — drafts engagement-ranked emergent pains from real people (Reddit keyless RSS / HN / GitHub via local last30days), a separate critic subagent scores it against _critic/rubric.md and the producer revises, then logs the critique forward to _critic/feedback-log.md. Replaces the deleted cloud daily-market-pains. Writes marketing/reports/market-pains/YYYY-MM-DD.md.
---

You are running the operator's daily **Pain-Point Finder** (~10 PM PT). Discover the web's hottest *emergent pains* from the last 30 days — ranked by REAL engagement (upvotes, comments, reactions) from REAL people, not vendor marketing blogs — run a critic loop to improve the draft, write a lean report to the the vault, and log the critique forward so quality compounds over time.

This REPLACES the old cloud `daily-market-pains` routine (deleted 2026-06-07), which sourced "pain" from competitor SEO blogs (ventus.ai, agentzap, dentalaiassist, etc.) and never reached Reddit. This version runs a local research engine that reaches real people, then critiques itself.

AUDIENCE: the operator — founder scanning for product/venture opportunities (AI agents, SaaS, services). They want what people are genuinely frustrated about right now, with receipts (links + engagement), filtered to things he could actually build.

## THE ENGINE (local, free, no API keys)
- Script: `<ENGINE_DIR>/skills/last30days/scripts/last30days.py`
- Interpreter: `python` (3.12+ confirmed). Run via the Bash tool (or PowerShell if Bash can't find python).
- Free sources only: **reddit, hackernews, github**. No paid keys. Reddit uses keyless RSS — the `.json` endpoint returns HTTP 403, that is EXPECTED; the RSS path is the working one on this residential IP.
- YOU are the planner AND the ranker. The engine's built-in ranking is near-useless without an LLM key (every item shows `score:0` / `entity-miss demotion`) — IGNORE the engine's ordering. Read the raw `EVIDENCE FOR SYNTHESIS` block (titles, communities, upvote/comment counts, snippets, top comments) and rank it yourself by (real engagement × relevance-to-an-actual-pain).

## STEP 1 — Date
Bash: `date +%Y-%m-%d` (PT local) → TODAY.

## STEP 1.5 — Read the critic loop's memory (lessons feed forward — this is half of what makes it self-improve)
Read these before researching (treat a missing file as empty):
- `<VAULT>/marketing/reports/market-pains/_critic/rubric.md` — the scoring rubric. You WILL be judged against it; keep its full text for STEP 4.5.
- `<VAULT>/marketing/reports/market-pains/_critic/feedback-log.md` — read ONLY the **📌 the operator's standing notes** section (always apply) + the **last 3 dated entries** under `## Log` (recent lessons + residual gaps to fix THIS run). Do not re-read the whole log.
Apply these as you research and write. the operator's standing notes win on content priorities (not on the hard evidence/format rules below).

## STEP 2 — Pick tonight's probes (open discovery, no industry limit)
The engine needs input topics. Use a rotating bank of **industry-agnostic pain-pattern probes** — they surface pains across all domains; engagement decides what's hot. From validation: churn / regression / breakage probes yield strong signal; vague "wish / idea / manual" probes yield mostly noise — weight accordingly.

Probe bank (these `search_query` strings match how threads are TITLED):
- "switching away from" · "cancelled subscription" / "cancelling my" · "deleted my account" · "latest update broke" / "ruined the app" · "hidden fees" / "price increase" · "killed the API" / "API deprecated" · "alternative to" (+ a hot category if obvious) · "enshittification" · "done with" / "stop using"

Pick **3 probes tonight**, rotating through the bank by day-of-year so it cycles. ALWAYS include ≥2 churn/breakage-style probes. You MAY swap in one sharper probe if there's an obvious hot target this week, or one suggested by recent feedback-log lessons.

Broad subreddit net: `smallbusiness,Entrepreneur,productivity,software,sysadmin,msp,freelance,ecommerce,personalfinance,jobs,mildlyinfuriating,webdev,SaaS`

## STEP 3 — Run the engine (once per probe)
For each probe:
1. Write a plan JSON (Write tool) to `<ENGINE_DIR>/runs/plan-{label}.json`:
```json
{"intent":"opinion","freshness_mode":"balanced_recent","cluster_mode":"debate",
 "subqueries":[
   {"label":"primary","search_query":"<probe>","ranking_query":"<natural-language question about the pain>","sources":["reddit","hackernews","github"],"weight":1.0}
 ]}
```
   - `search_query`: keyword-heavy, NO year/month/"recent"/"news" words. 1-2 subqueries per probe.
   - Keep `github` for dev/tool pains (API-killed integrations, regressions, maintainer pain); reddit+hackernews carry consumer/SMB.
2. Run (Bash):
`python "<ENGINE_DIR>/skills/last30days/scripts/last30days.py" "<short topic phrase>" --emit=compact --plan "<ENGINE_DIR>/runs/plan-{label}.json" --subreddits "smallbusiness,Entrepreneur,productivity,software,sysadmin,msp,freelance,ecommerce,personalfinance,jobs,mildlyinfuriating,webdev,SaaS" --save-dir "<ENGINE_DIR>/runs"`
3. Read the `EVIDENCE FOR SYNTHESIS` block from stdout. If a run errors/returns nothing, note it and continue.
4. **Pause ~60s between probes** (Bash `sleep 60`) — running 3 probes back-to-back rate-limited Reddit RSS to a 429 on 2026-06-07. If a probe's Reddit RSS still 429s / returns 0, note it in Coverage and lean on HN/GitHub for that probe.

## STEP 4 — Rank & filter, then DRAFT (you do this; the engine cannot)
Across all probes' evidence, select the **5-8 strongest emergent pains**, ranked by engagement × relevance.
DISCARD (do NOT report): AI-generated "news radio"/bot channels; low-view listicle YouTube; pure viral one-offs with no recurring pain (e.g. "Walmart shipped 165 pool noodles in 165 boxes"); Show HN / launches / self-promo UNLESS the thread reveals an unmet need; off-topic high-engagement threads (memes, drama, sports, politics, generic life-advice).
A pain qualifies only if real people are frustrated about a product/service/workflow AND there's engagement behind it (meaningful upvotes/comments, or ≥2-source corroboration).
Produce a complete **draft** in the schema below — but do NOT write the file yet.

## STEP 4.5 — Critic loop (a SEPARATE critic reviews; you revise) — BEFORE WRITING
1. Spawn a **separate critic agent** with the Agent tool (default type). Give it: (a) the full text of `rubric.md` from STEP 1.5, (b) the **📌 the operator's standing notes** you read, (c) your complete draft, and (d) this instruction: *"Score each rubric dimension 0-5 with a one-line reason. List the top 3 concrete, specific fixes (name the exact pain/line). End: PASS (all ≥4) or REVISE (list dimensions <4). Be adversarial — catch weak / unsourced / noisy / un-buildable pains; do not be nice. Optionally propose ONE rubric change."*
2. If verdict = REVISE: apply the fixes, then **SendMessage the revised draft back to the same critic** for one more pass. **Max 2 rounds total**, then proceed even if not all-PASS (record residual gaps).
3. If the Agent tool is unavailable this run, do the critique **inline** as a distinct self-review pass against the rubric (mark "(inline critique)" in the log).
Keep the critic's final per-dimension scores + verdict for STEP 6.
HONESTY: do NOT inflate scores to force a PASS. An honest REVISE that genuinely improves the report is the entire point of this loop.

## STEP 5 — Write the report
Path: `<VAULT>/marketing/reports/market-pains/{TODAY}.md` (mkdir -p if missing; Write tool; overwrite a same-day file).
Schema (LEAN — discovery feed, not an essay):
```
# Pain Finder — {TODAY}

> Whole-web, engagement-ranked emergent pains from the last 30 days. Real people (Reddit/HN/GitHub), not vendor blogs. Probes tonight: {probe1}, {probe2}, {probe3}.

## 🔥 Top Pains

### 1. {Pain headline} — {who / domain} · {aggregate engagement, e.g. 1,389↑ / 439💬 · 2 sources}
- **In their words:** "{verbatim quote}" — {r/sub or HN or owner/repo}, {N↑ / N💬}, [link]
- **The pain:** {1-2 sentences — what's broken, who feels it}
- **Why now:** {what changed — only if evident; else omit}
- **Opportunity:** {ONE line — product/agent/service wedge, or "monitoring — no clear wedge yet"}

### 2. ... (5-8 pains, strongest first)

## 📈 Rising (thin but climbing)
- {pain} — {signal + engagement} [link]   (0-5 one-liners)

## Coverage
- Probes: {list}. Sources: Reddit (keyless RSS), HN, GitHub. {Note any source that returned 0.}
- {If tonight was tech/dev-heavy: "Skew: free sources lean tech/dev; consumer/SMB breadth limited without paid social."}
- Discarded as noise: {N} items ({brief}).
- Critic: {verdict}, scores {compact per-dim list}.
```

## STEP 6 — Append to the critic log (the other half of self-improvement)
Prepend a dated entry under `## Log (newest first)` in `_critic/feedback-log.md`:
```
### {TODAY}
- Scores: Opportunity n/5 · Evidence n/5 · Signal n/5 · Freshness n/5 · Actionability n/5 · Format n/5 · Diversity n/5 — verdict PASS / REVISE×k
- Fixes applied this run: {1-3 bullets}
- Residual gaps (for next run): {bullets, or "none"}
- the operator notes: 
```
If the critic proposed a rubric change, add it (one dated line) to the **## ⏳ PENDING the operator APPROVAL** section at the TOP of the log. Do NOT edit `rubric.md` or this prompt yourself — the operator approves those. Keep the log lean: if `## Log` exceeds ~40 entries you may drop ones older than 30 days.

## STEP 7 — Post to the coordination board
After the report is written, append ONE post to the bottom of `<VAULT>/meta/agent-log.md` (Read it, append at the very end, Write the whole file back — NEVER alter any post above yours). This is the shared agent message board; your post is a SIGNAL, not a copy of the report. Format (match the → / ✅ style already in the file):
```
## [{TODAY} 22:00 PT] daily-pain-finder · [C] · ok  #presale
→ [[marketing/reports/market-pains/{TODAY}]]
Top pain: {headline} ({aggregate engagement}). {N} pains, {M} rising.
@presale-orchestrator {ONE line — the single strongest candidate-worthy pain to consider ingesting, or "nothing clears the candidate bar tonight"}.
```
If the report failed to write, post `· [C] · fail` with a one-line reason and omit the → link.

## CITATION / EVIDENCE RULES (BLOCKING)
- Every pain MUST link to a real source URL and show its engagement, from the evidence block.
- Quotes VERBATIM from the evidence block; tag [paraphrased] if compressed.
- NO vendor/competitor marketing blog as primary evidence (the old routine's core flaw); only as a 1-line [vendor] context tag, never as the pain itself.
- Do NOT invent engagement, ROI, or market-size numbers.

## NEVER
- NEVER dump the engine's raw "### N. (score…)" clusters — synthesize.
- NEVER trust the engine's ranking (it's `score:0` noise without an LLM) — you rank.
- NEVER report a pain without a link + engagement count.
- NEVER fabricate quotes, engagement, ROI, or TAM.
- NEVER claim "Reddit API pending / under review" — Reddit works via keyless RSS here; if RSS returns 0, write "Reddit RSS empty this run." Don't invent a ticket story.
- NEVER include AI-news bots, viral one-offs without actionable pain, or pure self-promo as a "pain."
- NEVER add decimal-precision scores (no 9.98), "Day N of series" bookkeeping, or Method/Honesty essays. Lean feed only.
- NEVER skip the STEP 4.5 critic loop or fake its scores — an honest REVISE is the point.
- NEVER edit `rubric.md` or this task's prompt yourself — the critic only PROPOSES changes into the log's PENDING section; the operator approves.
- NEVER exceed ~900 words in the report.
- NEVER fabricate the board post or its @presale-orchestrator handoff — base it only on tonight's actual pains; if none clear the candidate bar, say exactly that.

## AFTER WRITING
Output exactly 4 lines:
1. "Saved Pain Finder to {filepath}. Top pain tonight: {headline} ({engagement})."
2. "Critic: {verdict}, scores {compact list}."
3. "Pending rubric proposals for the operator: {count}."
4. "Posted to agent-log: handoff to @presale-orchestrator = {the candidate line, or 'none tonight'}."