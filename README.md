# pain-finder-agent

A scheduled LLM agent that reads the web for what real people are frustrated about,
drafts a ranked report, has a **separate critic agent** score the draft against a
rubric, revises, and then writes the critique forward so the next night starts
smarter. Everything here is prose that an agent runtime executes: there is no
application code, and that is the point. The specification *is* the system.

Ran nightly from June to September 2026 on Claude Code scheduled tasks. Seventeen
logged runs, sixteen rubric changes proposed by the critic and parked for human
approval, one approved, one rejected.

```
 22:00 PT  ─> SKILL.md (producer)
               1. read rubric.md + operator notes + last 3 log entries   <── memory
               2. pick 3 probes from a rotating bank ("cancelled subscription",
                  "latest update broke", "switching away from", ...)
               3. run the search engine once per probe (Reddit RSS, Hacker News, GitHub)
               4. rank the raw evidence yourself, draft 5 to 8 pains with links + engagement
               5. spawn a critic agent: score 7 dimensions 0 to 5, name 3 fixes, PASS or REVISE
                  <── up to 2 rounds; honest REVISE is the goal, not PASS
               6. write the report; prepend scores, fixes and residual gaps to feedback-log.md
               7. if the critic proposed a rubric change, park it under PENDING APPROVAL
                  (the agent never edits the rubric or its own prompt)
```

## Files

| File | What it is |
|---|---|
| `SKILL.md` | The producer prompt: engine invocation, probe bank, ranking rules, report schema, critic protocol, logging protocol, and a NEVER list of blocking rules |
| `rubric.md` | The seven scoring dimensions the critic uses, with auto-penalties, and the one sub-rule that was approved after the critic proposed it |
| `feedback-log.md` | The loop's memory: standing operator notes, sixteen pending rubric proposals with rationale, and seventeen dated entries of scores, fixes applied, and residual gaps |
| `examples/2026-07-16.md` | A thin night with one buildable pain and four corroborating threads |
| `examples/2026-09-03-honest-empty-night.md` | A night where nothing cleared the bar, reported as such, with a concrete check for each watch item |

## Design decisions worth stealing

- **The model is the planner and the ranker, not the engine.** The free search engine
  (keyless Reddit RSS, Hacker News, GitHub) returns `score: 0` for every item without a
  paid key. The prompt says so and tells the agent to rank the raw evidence itself by
  engagement times relevance.
- **A separate critic, not self-review.** The critic is spawned as its own agent with the
  rubric, the operator's standing notes, and the draft, and is told to be adversarial.
  Self-review is the fallback only when a subagent cannot be spawned, and must be
  labeled as such in the log.
- **The agent cannot change its own rules.** Rubric edits and prompt edits are proposals
  that go to a PENDING section. A human approves or rejects. Sixteen proposals
  accumulated; reading them in order shows the critic finding real holes
  (self-referential statistics, aggregated engagement counts without matching links,
  "the user already solves it for free" demotions).
- **Honest-empty nights are a valid output.** The rubric caps Top Pains at the number of
  items with a real wedge. A night with none says "none tonight" and lists what would
  have to change to promote each watch item. Five of seventeen runs ended this way.
- **Blocking evidence rules.** Every pain needs a real link and an engagement count
  from the evidence block; quotes are verbatim or tagged paraphrased; vendor blogs are
  never primary evidence; invented numbers score zero on evidence integrity.
- **Rate-limit etiquette as a rule.** Three probes back to back got Reddit RSS rate
  limited on the first night; the prompt now sleeps 60 seconds between probes.
- **Plain words.** After one report came back full of abbreviations, the operator added
  a standing note banning them in the report body. It is applied every run.

## Running it

Requirements: an agent runtime that can run scheduled prompts, spawn a subagent, run
shell commands, and read and write files (this was built on Claude Code scheduled tasks),
plus the open-source `last30days` research engine (`mvanhorn/last30days-skill`,
Python 3.12, standard library only).

1. Clone the engine somewhere and note the path; it becomes `<ENGINE_DIR>` in `SKILL.md`.
2. Pick a folder for reports; it becomes `<VAULT>/marketing/reports/market-pains/`. Put
   `rubric.md` and `feedback-log.md` in its `_critic/` subfolder.
3. Replace the two placeholders in `SKILL.md`, register it as a scheduled task (or run it
   on demand), and read the report the next morning.
4. Once a week, open `feedback-log.md`: approve or reject the pending proposals and edit
   the standing notes. The loop only stays aligned with your goal if you do.

## License

MIT
