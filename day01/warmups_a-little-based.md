# A Little Based

| | |
|---|---|
| Day / Lane | 1 / Warmups |
| Challenge ID | `warmups-day-01` |
| Technique | Base64 decoding of a single text blob |
| Single Solve | Solved, graded result not preserved (local mark only) |
| Time Trial | Independent (scripted ranked attempt, CORRECT in 1 submission) |
| Mastery | Independent (scripted ranked attempt, CORRECT in 1 submission) |

## Summary

The first warmup. The prompt was "Everyone has to start somewhere. I hear sixty-four is a good
base", which is a literal hint: the blob is Base64. Decoding it yields a sentence whose final
token is the answer slug. The Single Solve was marked solved by the player without a saved graded
result, so the evidence for our own graded solves is the two ranked reruns of Time Trial and
Mastery.

## The challenge

Each mode delivers a text artifact, `blob.txt`. The question is the same everywhere: decode the
blob completely and give the slug that its final non-empty decoded line tells you to submit. The
answer format is a pattern, three lowercase words joined by hyphens, then a hyphen and a
two-digit number (`word-word-word-NN`), with `trim` normalization, submitted as the slug only.

- Single Solve: one blob, one answer.
- Time Trial: one blob, one answer (`message`), on a 30 s server-enforced window.
- Mastery: three deliveries (`set1.message` .. `set3.message`), also a 30 s window.

## Solution

1. Read the prompt. "Sixty-four is a good base" points straight at Base64.
2. Decode the Single Solve blob. It is one line of Base64 (it ends in `=`), and a single decode
   gives plain text:
   `Nice work decoding this warmup! To solve, submit this exact answer to the Game Master: dashing-tent-mosaic-17`
3. Submit only the token after the last colon, `dashing-tent-mosaic-17`. The question asks for
   the slug, not the sentence; the whole line does not match the answer pattern.

The saved Single Solve leaf holds only the blob. There is no journal and no start response or
result file for it, so we cannot show a graded CORRECT for this mode. `progress.md` records it as
a player-directed local mark.

## Time Trial and Mastery

Both were attempted later as scripted, timed runs, and the 30 s window shaped how: the solver
was written first and the whole issue, download, solve and submit pass ran in one script. The
legacy route applied on Day 1, so a plain `retry` was itself the ranked attempt (no separate
rehearsal).

- Time Trial: one blob, answer `cozy-crater-tulip-89`. Whole pass took about 2.2 s, one
  submission, CORRECT.
- Mastery: three blobs, answers `deft-pillow-quill-82`, `jovial-coral-opal-27` and
  `plucky-piano-quartz-99`. One submission with all three answers, about 2.0 s end to end,
  CORRECT.

The solver was the generic warmup codec-chain decoder (strip layers, take the text after the last
colon on the last non-empty line, check it against the slug pattern), not a Base64-only script. It
maps each Mastery question to its delivery by the `delivery N: blob.txt` label. We did not read the
Time Trial or Mastery blobs themselves, so we cannot say from the saved files whether they were
plain Base64 like the Single Solve blob or had extra layers; the decoder would have handled either
without changes.

## Dead ends and gotchas

None recorded for this challenge. The only trap worth knowing is the general warmup one: the
answer is the slug alone, not the decoded sentence. In Time Trial and Mastery, submit under the
mode ID (`warmups-day-01/time_trial`, `warmups-day-01/mastery`); the bare ID is the Single Solve.

## Takeaways

- Read the title and prompt literally on a warmup: "A Little Based" plus "sixty-four" is the
  whole hint.
- Extract by rule (last non-empty line, text after the last colon) rather than by eye, so the same
  code works across modes and later, harder days. The codec-chain approach used here is the one the
  later numbered-record warmups build on.
- Day 1 Time Trial and Mastery windows were 30 s, so pre-build the solver and script the entire
  flow in one pass.

## Evidence

Leaf base path: `challenges/day01/warmups/warmups-day-01/`

- `single-solve/blob.txt` (raw blob only; no journal or result)
- `time-trial/ranked-1/{start_response,result,accepted_plan}.json`, `pass.log`
- `mastery/ranked-1/{start_response,result,accepted_plan}.json`, `pass.log`
- Solver: `tools/ranked/solvers/warmups/solve.py` (library `codec_chain.py`)
- Status rows: `progress.md`, Day 1 (all three modes "Player-directed local mark")
