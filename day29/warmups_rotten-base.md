# Rotten Base

| | |
|---|---|
| Day / Lane | 29 / Warmups |
| Challenge ID | `warmups-day-29` |
| Technique | Layered text decoding (ROT13 over base64), per-record codec table for the Mastery bundle |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent (regular and ranked attempts all CORRECT on the first submission) |
| Mastery | Independent (regular CORRECT on call 2, ranked CORRECT on call 1) |

## Summary

A warmup whose prompt names the two layers: "Sixty-four" is base64 and "thirteen" is ROT13. The blob is base64 text that has been ROT13'd, so one ROT13 and one base64 decode give a template sentence whose last line carries the answer slug. Single Solve and Time Trial were trivial. Mastery looked the same but failed 143 of 143 on the first submission, because each of its blobs holds 29 numbered records and record 29 used a codec our table did not know yet.

## The challenge

The prompt was "Sixty-four invited thirteen over. Things got scrambled." Every mode asks the same question: decode the assigned blob completely and submit the slug named by its final non-empty decoded line. The answer format is a pattern: three lowercase words joined by hyphens, a hyphen, and a two-digit number (for example `hearty-stream-laurel-58`, the Time Trial answer of the first ranked attempt).

- Single Solve: one `blob.txt`, one answer (`message`), no deadline.
- Time Trial: one `blob.txt`, one answer, ranked window 30 s.
- Mastery: one ZIP of 143 blobs (`blob-NN.txt`), 143 answers (`setN.message`), ranked window 160 s.

## Solution

1. The prompt is a literal hint. "Sixty-four" maps to base64 and "thirteen" to ROT13. "Things got scrambled" suggests the layers are stacked rather than one encoding.
2. Our warmup decoder (a layer-peeling tool that tries known encodings and scores each result against the expected template and slug pattern) already knew both layers. It peeled ROT13 first and then base64, with no changes needed. A synthetic example of the layering is `rP0jAj==` -> ROT13 -> `eC0wNw==` -> base64 -> `x-07`.
3. The decoded text is a short template message. The answer is the literal text after the last `:` of its last non-empty line. We took the literal tail as the submitted answer. If the tail ever failed the pattern, the pattern-shaped alternative was only a lower-priority variant. That fallback was never needed here.
4. The solver was the Day 28 warmup solver (a thin wrapper around the decoder's `process_all`), plus a group of digit-rotation (ROT18-style) alternates as variants 1 and up. Variant 0 was accepted in every graded call.

## Time Trial and Mastery

Time Trial was identical to Single Solve with a fresh blob. The first ranked attempt took 4.19 s end to end (30 s window), and later ranked reruns took 4.41 s and 2.57 s, all CORRECT on the first submission.

Mastery is where the day's only real problem sat. The bundle holds 143 blobs, and each blob contains 29 numbered records; the record number fixes the codec (Day N has N records). Records 1 to 28 reused older codecs from earlier days. Record 29 was a new table entry: ROT13 over base64, on a short payload (a base64 string whose ROT13 is a base64 group that decodes to the slug's last character plus `-NN`).

The first Mastery submission used the decoder's table as it was, so record 29 was left undecoded and every one of the 143 slugs ended in a replacement character. The Game Master returned `incorrect` for all of them (143/143). We added `29: rot13-base64` to the codec table, re-ran offline (143/143 matched the slug pattern, no repairs, no warnings, about 0.15 s), and the next submission was CORRECT. The ranked Mastery attempt then went CORRECT on call 1 (13.2 s total against a 160 s window).

## Dead ends and gotchas

- Single Solve and Time Trial pass without the table fix, because the bare blob is peeled by the strict layer detector. A solver that special-cases the Single Solve blob will pass those two modes and fail Mastery. The solver has to call the whole-bundle path (`process_all`), not the single-blob peel.
- `incorrect` on all 143 answers at once is a systematic cause, not 143 bad guesses. Here it was one missing codec for one record number, and the outputs visibly ended in a replacement character.
- The digit-rotation alternate group and the "non-template tail" group were prepared but never used.

## Takeaways

- Read warmup prompts literally: the number words name the codecs (base64, ROT13).
- Before submitting a Mastery bundle, check that every record number in the bundle is in the codec table. A new day usually adds exactly one new record number.
- Layer-peeling with a template/pattern score plus a per-record table handles these challenges quickly. The Mastery set is only the same thing repeated 143 times at 29 records each.

## Evidence

Provenance, checked per mode from local files:

- Single Solve: `challenges/day29/warmups/warmups-day-29/single-solve/` holds `team_solved.json` (reason `start_already_correct`). In the release run our offline answer was submitted once and returned `already_solved`, so nothing was graded for us. A fresh attempt was not possible (retry returned 405 `operation_not_allowed`, practice returned `retry_variants_unavailable`). What we did do is derive the answer offline with the decoder. The method is validated by the Time Trial and Mastery results, but the Single Solve answer value itself was never graded for us, so none is given here, and the graded solve was a teammate's.
- Time Trial: `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (and `ranked-2`, `ranked-vm-1`), plus `accepted_plan.json`. All CORRECT, 1 call each.
- Mastery: `mastery/regular/result.json` (143 answers, CORRECT after the table fix), `mastery/ranked-1/result.json`, plus `accepted_plan.json`.
- Solver: `single-solve/artifacts/cand-adapter/solve.py`. Journal: `single-solve/journal.md` (rubric fix round 1).
- Day-level: `challenges/day29/_run/librarian/warmups-day-29.md` and `challenges/day29/_run/lane-warmups-day-29.log`.
