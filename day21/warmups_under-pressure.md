# Under Pressure

| | |
|---|---|
| Day / Lane | 21 / Warmups |
| Challenge ID | `warmups-day-21` |
| Technique | layered text decoding (base64 wrapper around gzip), codec table per record number |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Under Pressure" is the Day 21 warmup: a text blob that is a base64 wrapper around gzip-compressed
text, whose final decoded line names an answer slug. The Single Solve and Time Trial were one blob
each and went CORRECT on the first submission. The Mastery was 103 blobs of 21 layers each; its
only difficulty was that the decoder table stopped at record 20 and then a per-set harness paired
the wrong blob with sets 11-103. That cost about 108 submission calls before the regular Mastery
attempt went CORRECT in one call.

## The challenge

Prompt: "A familiar wrapper. Something compressed inside." One question per blob (`message`, or
`setN.message` in Mastery): decode the assigned blob completely and submit the slug named by its
final non-empty decoded line. Answer format is `pattern` with `trim` normalization: three lowercase
words separated by hyphens, then a hyphen and a two-digit number, slug only (no surrounding
sentence).

- Single Solve: one file `blob.txt`, one answer.
- Time Trial: one blob, one answer. Ranked windows were 30 s.
- Mastery: a ZIP of `blob-01.txt` ... `blob-103.txt`, 103 answers (`set1.message` ...
  `set103.message`). Ranked window was 120 s.

## Solution

1. **Recon.** The blob decoded with the library's chain-peeling tool in one pass: base64, then gzip,
   then a short English sentence ending in a colon and the slug. The tool scores candidate decoders
   against a crib/template and stops when the output matches the slug pattern.
2. **Answer rule.** From earlier warmup days the answer is the text after the last colon on the last
   non-empty line, not the whole sentence. The solver checked the result against the format regexp
   before it was allowed to answer.
3. **Submission.** The Single Solve answer was `eager-compass-mosaic-50`. It was graded CORRECT on
   the first call. The Time Trial used the same decoder on a fresh blob and was also CORRECT on call
   1. No alternate readings were needed, so no variant groups were built.

The solver itself is a thin wrapper: load the codec tool, run it over the delivered blob(s), keep
results whose slug matches the pattern, and key them to answer ids by the question's `artifact`
field:

```python
by_member = {}
for r in good:
    by_member[Path(r["source"].rsplit("!", 1)[-1]).name] = r["slug"]
qmap = {q["id"]: q.get("artifact") for q in questions}
for aid in ids:
    art = qmap.get(aid) or qmap.get(strip_set(aid))   # strip "setN." prefix
    slug = by_member.get(Path(art).name) if art else None
```

## Time Trial and Mastery

**Time Trial.** Same shape as the Single Solve, one blob. Regular and ranked attempts were all
CORRECT on one call (five ranked attempts exist in the leaf, 1.7 s to 47 s total wall time; the long
ones were slow submits, not solve time, which was 0.3-1.2 s).

**Mastery.** Each of the 103 blobs carried 21 layers, matching the lane's pattern that day N has N
records and the record number fixes the codec. Records 1-20 were already in the decoder table.
Record 21 turned out to be the same base64-then-gzip wrapper as the Single Solve, which the table
did not yet list, so the first run printed "no slug decoded" for every set. Adding
`21 -> base64-gzip` to the table fixed the decode (103 distinct slugs, ids matching `answer_ids`,
0.2 s). The final regular Mastery went CORRECT with a single 103-answer call, and ranked Mastery
attempts (10.7 s and, on a later re-run, 5.8 s total) did the same.

## Dead ends and gotchas

- **A missing record in the table.** The Mastery first failed because record 21 had no codec entry.
  The Single Solve passed anyway because it peels the bare blob without needing the per-record table,
  the same split seen on later warmup days. Fix the table before submitting.
- **Sorted order breaks at 100.** Mastery members are named `blob-01` ... `blob-99`, `blob-100` ...
  `blob-103`. A plain lexical sort puts `blob-100..103` ahead of `blob-11`. The per-set run handed set
  N the N-th sorted blob, so sets 1-10 were right and sets 11-103 received the wrong slug. The
  validator only said "incorrect" and "accepted" for small subsets, and bisecting that map burned
  about 100 submission calls (the ledger was raised from 100 to 111 to finish). The decoder was
  correct throughout; the pairing of blob to set was not. The lesson is to map answers by the
  question's `artifact` field, never by position, and to treat a map built from a path with a known
  decode problem as unsuitable for bisection. This part was a mistake in our own run tooling; see the
  Day 21 overview.
- **No answer-format surprise.** The slug (not the sentence) was accepted as-is, with no alternate
  spellings.

## Takeaways

- Technique: **layered codec peeling with a per-record codec table** (record number selects the codec;
  a new day adds one record). A blind peel-and-score tool finds the new layer; adding it to the table
  makes Mastery deterministic.
- Fail closed: if any blob fails to decode or a name does not match its question, refuse to answer
  instead of falling back to position.
- Natural-sort archive members, or better, look them up by name.
- Faster: add the next day's expected record to the table before release, and submit the full Mastery
  map in one call once the decode and id mapping are verified offline (here that was the one call that
  worked).

## Evidence

Leaf base path: `challenges/day21/warmups/warmups-day-21/`.

- `single-solve/journal.md`, `single-solve/accepted_plan.json`, `single-solve/start_response.json`
- `single-solve/artifacts/cand-rubric/solve.py` (the accepted decoder wrapper),
  `single-solve/artifacts/feedback-round1.json` to `feedback-round3.json`
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (also `ranked-2` to `ranked-5`)
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `challenges/day21/_run/lane-warmups-day-21.log` (submission rounds)
- `library/warmups/gotchas.md` (WARMUP-GOTCHAS-001), `library/warmups/lane-evolution-and-playbook.md`
  (WARMUP-PLAYBOOK-001), `library/warmups/codec-chain/codec_chain.py`
- `progress.md` (grep `^| 21 .*warmups`)
