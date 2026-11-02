# Percent Percent

| | |
|---|---|
| Day / Lane | 30 / Warmups |
| Challenge ID | `warmups-day-30` |
| Technique | double percent-decoding (`%25%HH`), layered text codec |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Percent Percent" is a Warmups text-decoding challenge: the blob is URL percent-encoded twice, and
the final decoded line is the standard warmup sentence that names the answer slug. Our existing
codec-chain tool already peeled both layers unchanged. The Single Solve was scored by a teammate
before we submitted, so our submission returned `already_solved` and nothing was graded for us.
Time Trial and Mastery (148 blobs) both went CORRECT on the first submission.

## The challenge

Prompt: "That address bar problem? It happened twice." Every Warmups mode asks the same question:
decode the assigned blob completely and submit the slug named by its last non-empty decoded line.
The answer format is a `pattern` (three lowercase words joined by hyphens, then a hyphen and a
two-digit number), normalization `trim`, one answer id (`message`).

- Single Solve and Time Trial: one text blob (`blob.txt`), one answer.
- Mastery: a ZIP of 148 blobs (`blob-01.txt` ... `blob-148.txt`), 148 answers with ids
  `setN.message`, each pointing at its blob through the question's `artifact` field. Each blob is a
  list of numbered records, and the record number selects the codec (Day N has records up to N, so
  Day 30 reaches record 30).

## Solution

1. **Recognize the layer.** The prompt is the hint: an address bar encodes text with `%HH`
   escapes, and "twice" means the escapes were themselves escaped. Every byte of the plaintext
   appears as `%25%HH` (`%25` is an escaped `%`), so a first decode yields ordinary `%HH` text and
   a second yields the sentence. The first bytes tell the layers apart: `%25%` is a second layer,
   a bare `%HH` is one.
2. **Decode with the existing tool.** The library's repeated-percent-decoding technique
   (`decode_percent_layers.py`) covers this by decoding until the text stops changing. For the
   release we used the consolidated warmups tool `codec_chain.py`, which peels percent-decoding
   twice on its own. The solver was a thin wrapper around its `process_all`:

   ```python
   results = cc.process_all(list(cc.iter_inputs([Path(a.artifact)])), crib,
                            cc.DEFAULT_SLUG, dict(cc.DEFAULT_RECORD_MAP), 16, strong, [])
   ```

3. **Take the slug.** The decoded last line is the standard warmup template; the slug is the literal
   text after the last `:`. The wrapper accepted it only if it matched
   `[a-z]+-[a-z]+-[a-z]+-\d{2}` or at least the template shape, and otherwise failed loudly.
4. **Submit.** The solver's Single Solve answer map was one id and no alternate groups, but the
   submission returned `already_solved`, so it was never validated by us. The accepted answers
   came from the Time Trial (`cozy-summit-fern-54` in the saved plan) and Mastery.

## Time Trial and Mastery

Both went through `retry`, which issued a fresh attempt regardless of the team's earlier solve.

- **Time Trial:** one blob, one answer, CORRECT on call 1.
- **Mastery:** 148 sets in one ZIP, each decoded independently and mapped to its id through the
  question's `artifact` field. The whole bundle went CORRECT in a single call (148 answers).
- **New record codec.** In Mastery, record 30 is the same double percent layer. Before it was
  added to the tool's record map, `codec_chain.py` auto-picked it as a two-step percent peel in
  148/148 sets, which gave the same slugs. Afterwards it was registered as `percent-x2` in
  `DEFAULT_RECORD_MAP`; the Librarian confirmed Time Trial 1/1 and Mastery 148/148 against the
  accepted plans.
- Later ranked (timed) reruns of both modes (a 30 s Time Trial window, a 165 s Mastery window)
  also went CORRECT on one call each. They were bragging-rights only and did not count for the
  prize ranking.

## Dead ends and gotchas

- **Record 30 was unmapped but harmless.** This is the day's only tool-relevant wrinkle: a record
  number missing from the map is auto-detected, and the tool printed an `AUTO-PICKED` notice. The
  auto-pick was right here; the rule of thumb remains to add the record to the map after a CORRECT.
- **`unquote` is not form decoding.** The percent-decoding library entry notes that `+` stays a
  plus sign and that invalid escapes stay literal. Neither mattered here, since the plaintext was
  plain ASCII.
- **Team-scored Single Solve.** `already_solved` from a submission is not a verdict, and `start`
  already reading `CORRECT` is the team's status, not ours. We moved on to Time Trial and Mastery
  instead of treating it as a failure.

## Takeaways

- Technique name: **repeated percent-decoding / double URL encoding**; tell the layer count from the
  first bytes (`%25%`).
- Do not decode a whole URL blindly; here the blob was text, so decoding until stable was safe.
- Extend the shared record map when a new record number appears; do not rely on auto-pick alone.
- Because the Single Solve was lost to a teammate within seconds, the independent proof that the
  workflow works came from Time Trial and Mastery.

## Evidence

Leaf base path: `challenges/day30/warmups/warmups-day-30/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-solo/solve.py` (the wrapper used for all three modes)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`,
  `mastery/regular/start_response.json`, `mastery/ranked-1/result.json` (also `ranked-2`, `ranked-3`)
- `challenges/day30/_run/librarian/warmups-day-30.md`, `challenges/day30/_run/lane-warmups-day-30.log`
- Library: `WARMUP-PERCENT-DECODING-001`, `WARMUP-CODEC-CHAIN-AUTO-001` (`library/warmups/`)
- `progress.md` (grep `^| 30 .*warmups-day-30`)
