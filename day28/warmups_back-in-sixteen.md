# Back in Sixteen

| | |
|---|---|
| Day / Lane | 28 / Warmups |
| Challenge ID | `warmups-day-28` |
| Technique | layered text codec chain (reverse, then hex), numbered Mastery records |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Back in Sixteen" is the Day 28 warmup: a blob that has to be peeled back to a one-line template
sentence whose tail is the answer slug. The prompt's riddle ("walking backwards") names the
codec: the payload string is reversed, then hex-decoded. The Single Solve was scored by a teammate
within seconds of unlock, so our submission came back `already_solved` and our offline answer was
never graded. Time Trial and Mastery were ours via `retry` and went CORRECT. The one real trap was
Mastery: record 28 had no entry in our codec table, which made the first 138-answer submission
fail across the board.

## The challenge

Prompt: "Our sixteen-fingered friend is walking backwards." (sixteen fingers = hex, backwards =
reverse.) The single question, `message`, asks for the exact answer slug that the final non-empty
decoded line tells you to submit. Answer format: `pattern`, three lowercase words joined by
hyphens plus a hyphen and a two-digit number (`adjective-noun-noun-NN`), `trim` normalization,
slug only.

Delivery:
- Single Solve / Time Trial: one blob (`blob.txt`), one answer id `message`.
- Mastery: a ZIP of 138 sets (`blob-NN.txt` members), answer ids `setN.message`. Each set's blob
  is a numbered record bundle: record N of a Day-N bundle is decoded with that day's codec, and
  earlier records use earlier days' codecs, so a Day 28 bundle has 28 records.

## Solution

1. **Recon.** The start response gave the blob, a SHA-256 (verified) and the pattern. Every warmup
   so far decodes to the same sentence, `Nice work decoding this warmup! To solve, submit this
   exact answer to the Game Master: <slug>`, so the decode was complete when the output matched that
   template.
2. **Identify the day's codec.** The riddle gave it away: peel `reverse`, then `hex`. The
   consolidated codec-chain tool already had both layers as strict, self-validating decoders, so the
   Single Solve and Time Trial blobs decoded with no new code.
3. **Extract the slug.** The answer is the literal text after the last `:` of the last non-empty
   decoded line. Offline, the Single Solve blob gave `speedy-button-nugget-52`; the Time Trial
   blob (a fresh instance) gave `speedy-chimney-ripple-98`.
4. **Submit.** Time Trial went CORRECT on the first call. Single Solve returned `already_solved`.

## Single Solve: team solve (not ours)

Our submission was `already_solved`: the team's existing attempt had already scored this ID, and
nothing was graded for us. We did decode the blob offline (above), but that answer is unvalidated
here. We did not record this mode as solved. The graded solve was a teammate's. We did not probe whether a fresh Single Solve attempt
could be issued for this ID.

## Time Trial and Mastery

Both start responses first returned the team's CORRECT readout. A `retry` on each issued a fresh
attempt with its own artifact, and we solved those ourselves.

- **Time Trial:** one blob, one answer. Regular (untimed) attempt: CORRECT on call 1. Ranked
  attempt: 30 s window, solved in 4.55 s locally, CORRECT on call 1. (2 calls on the ledger.)
- **Mastery:** 138 answers from a 138-set ZIP. The regular attempt needed fix rounds (below); once
  the codec table was fixed, one 138-answer call went CORRECT. The ranked attempt (155 s window)
  was solved in 9.3 s locally and CORRECT on call 1. (9 calls on the ledger in total, of which 7
  were the failed first attempt and its bisection.)

## Dead ends and gotchas

- **Record 28 was missing from the codec table.** Our numbered-record table mapped records 1-27
  but not 28. Without an entry, record 28 had no codec, so the last
  word of every slug was cut short and ended in a U+FFFD replacement character
  (`dapper-button-haze` + U+FFFD instead of `dapper-button-hazel-93`). That is why all 138 answers
  were rejected and bisection never isolated a good half: it was a systematic error, not 138
  wrong answers. The record-28 payload is the reversed hex of the slug's tail, e.g. `3393d2c6` ->
  reversed `6c2d3933` -> hex `l-93`, which is exactly the "last character plus `-NN`" shape
  Mastery tails take. Adding `reverse-hex` as `DEFAULT_RECORD_MAP[28]` fixed it: 138/138 slugs
  matched the pattern and the next submission was CORRECT.
- **The Single Solve and Time Trial blobs hid this bug.** They are one blob peeled by strict
  layers and never consult the per-record table, so they decoded correctly without it. Only the
  numbered Mastery bundle depends on the table.
- Answer format caused no problem: the bare slug, string-typed, was accepted as is.

## Takeaways

- Technique to search for: **numbered-record codec chain** (a Mastery bundle of N records where
  record i uses the day-i codec). Every new warmup day adds one codec that the Single Solve peels
  by layers but the Mastery needs registered by record number. Add the new record to the table
  *before* the first Mastery submission, and validate every slug against the pattern first.
- A systematic all-rejected Mastery (nothing accepted in any bisection half) points to a decoder
  gap, not to answer ambiguity; look at the common tail of the outputs.
- When a Single Solve is team-scored, do not stop: `retry` on Time Trial and Mastery issues
  independent attempts, and the same decoder covers them.

## Evidence

Leaf base path: `challenges/day28/warmups/warmups-day-28/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-adapter/solve.py` (the solver) and `.../out.json` (offline SS answer)
- `time-trial/ranked-1/result.json`, `time-trial/regular/result.json`
- `mastery/ranked-1/result.json`, `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`
- `challenges/day28/_run/lane-warmups-day-28.log` (rounds, the 138/138 rejection, the fix)
- `library/warmups/codec-chain/` (WARMUP-CODEC-CHAIN-AUTO-001, record 28 = `reverse-hex`)
- `progress.md` (grep `^| 28 .*warmups-day-28`)
