# Pain-Finder Rubric

The in-run critic scores each nightly report against this. **Evolvable** — the critic only PROPOSES changes (into the feedback-log's PENDING section); **the operator approves** before they land here.

Score each dimension **0–5**. Target: every dimension ≥4. If any dimension <4 → the report must be REVISED before it's written.

| # | Dimension | What earns a high score | Auto-penalty |
|---|---|---|---|
| 1 | **Opportunity value** *(most important)* | Each Top Pain maps to a plausibly **buildable** product / AI-agent / service the operator could pursue — clear-ish ICP + a one-line wedge that isn't generic. | A pain with no conceivable wedge → demote to "Rising" or cut. |
| 2 | **Evidence integrity** | Every pain has a real link + engagement count from the engine's evidence block. Quotes verbatim (or tagged `[paraphrased]`). | **Score 0** if any quote/number can't be traced to the evidence, or if a vendor/competitor blog is used as PRIMARY pain evidence, or any number is invented. |
| 3 | **Signal quality (noise control)** | Every pain has real engagement (meaningful upvotes/comments OR ≥2-source corroboration). | Any AI-news bot, viral one-off with no actionable pain, pure self-promo, or off-topic item in Top Pains. |
| 4 | **Freshness / relevance** | Pains genuinely from the last ~30 days and genuinely hot (engagement-backed), not evergreen filler. | Recycled anchors with no new signal. |
| 5 | **Actionability for the operator** | the operator could do something tomorrow (run presale direct-validate, build a probe, write content), not just "interesting". | "Interesting but inert" framing. |
| 6 | **Format discipline** | Lean schema, ≤900 words, no decimal scores, no "Day N of series" bookkeeping, no Method/Honesty essays. | Any reappearance of the old format's bloat. |
| 7 | **Diversity / coverage** | Not all pains from one niche/source unless the night genuinely only surfaced that. Tech-skew flagged explicitly when present. | Silent mono-source/mono-niche dump. |

> **Dim 2 sub-rule (approved 2026-06-07):** dynamically-rendered counts (live GitHub/Reddit numbers) and paraphrases must carry an as-of date or `[paraphrased]` tag. Untagged false-precision = **−1**, not auto-zero; auto-zero stays reserved for invented/untraceable numbers.

## Critic output format (structured)
- **Per-dimension:** `{dim}: {0–5} — {one-line reason}`
- **Top 3 fixes:** specific and addressed (name the exact pain/line, e.g. *"Pain #4 has no link — cut or source it"*).
- **Verdict:** `PASS` (all ≥4) or `REVISE` (list which dimensions <4 and why).
- **Optional:** at most ONE proposed rubric change → goes to the feedback-log PENDING section, never edited here directly.

## Anchor (read before scoring)
The ultimate test is **the operator's real goal**, not this rubric's letter: *would the operator look at this report and find a pain worth chasing?* Read **the operator's standing notes** in `feedback-log.md` and weight toward them. The critic's job is to be adversarial — catch weak / unsourced / noisy / un-buildable pains — not to be nice.
