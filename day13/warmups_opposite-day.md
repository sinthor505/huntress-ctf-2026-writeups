# Opposite Day

| | |
|---|---|
| Day / Lane | 13 / Warmups |
| Challenge ID | `warmups-day-13` |
| Technique | Atbash (alphabet-opposite) substitution; Mastery chains 13 text codecs by record number |
| Single Solve | Independent (journal-recorded CORRECT plus a saved `submission-attempt-01.json` reading CORRECT) |
| Time Trial | Independent (retry attempt, saved `retry-submission-response.json` reads CORRECT; ranked rerun `result.json` CORRECT) |
| Mastery | Independent (saved `submission-attempt-01.json` reads CORRECT; ranked rerun `result.json` CORRECT) |

## Summary

The whole prompt is a one-line hint: "A met Z. They really hit it off." A maps to Z, so the blob is
Atbash-encoded. Applying the substitution to the assigned text gives a short congratulatory
instruction whose last line ends with the answer slug. Single Solve and Time Trial are one Atbash
call over a one-line file. Mastery scales this to 63 files of 13 tab-numbered records each, where the
record number selects one of 13 different simple codecs and Atbash is only the last of them. All three
modes were CORRECT on the first submission to the right mode ID.

## The challenge

Prompt: "A met Z. They really hit it off." Question (same in every mode): decode the assigned blob
completely and submit the exact answer slug its final non-empty decoded line tells you to submit.
Answer format is a `pattern`: three lowercase words joined by hyphens, a hyphen, and a two-digit
number, slug only, `trim` normalization. One answer id, `message`, for Single Solve and Time Trial.

Deliveries:
- Single Solve and Time Trial: `blob.txt`, a single line of ASCII with no line terminator (the Time
  Trial retry blob was 109 bytes). Single Solve was untimed. The Time Trial window was 30 seconds.
- Mastery: a 31,004-byte ZIP of `blob-01.txt` ... `blob-63.txt`, one per answer id `set1.message` ...
  `set63.message`, under an 80-second timer.

## Solution

1. **Read the clue literally.** "A met Z" is the alphabet-opposite pairing A<->Z, B<->Y ... M<->N.
   That is Atbash, a self-inverse substitution on A-Z/a-z that leaves digits, spaces and punctuation
   alone. The existing library entry for it (WARMUP-ATBASH-001) matched.
2. **Verify the bytes.** The saved blob's SHA-256 matched the start response before analysis.
3. **Decode.** One `str.translate` over the whole blob:

   ```python
   ATBASH = str.maketrans(
       "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz",
       "ZYXWVUTSRQPONMLKJIHGFEDCBAzyxwvutsrqponmlkjihgfedcba",
   )
   decoded = raw.decode("ascii").translate(ATBASH)
   ```

   The encoded text begins `Mrxv dlip wvxlwrmt gsrh dzinfk!` and decodes to a sentence of the form
   "Nice work decoding this warmup! To solve, submit this exact answer to the Game Master: <slug>".
4. **Extract the slug.** Take the last non-empty line, split on the final `:`, strip, and check it
   against the pattern `[a-z]+-[a-z]+-[a-z]+-\d{2}`. Single Solve answered `handy-candle-quartz-31`.
5. **Map to the answer id.** The mapper read `answer_ids` from the start response and asserted it
   equalled `["message"]` before printing `{"message": "<slug>"}`.

## Time Trial and Mastery

**Time Trial** used the same transform on a different blob, so it yields a different slug
(`silly-basket-maple-95` for the first timed blob). The first timed attempt was only downloaded and
hashed during recon and expired unanswered at its 30-second deadline; a timed `retry` issued a fresh
109-byte blob (new attempt, new hash), which the saved offline mapper decoded and submitted in time. A
later ranked rerun (2026-10-26, on a scripted pass) went CORRECT in one call, 1.63 s end to end.

**Mastery** is the real change. Each of the 63 blobs holds 13 lines, `N<TAB>payload`, and the
record number fixes the codec. The decoded fragments concatenate into the same kind of instruction
sentence, and the slug is again after the final `:`. The codec table the mapper used:

| Record | Codec |
|---|---|
| 1, 11 | Base64 |
| 2 | ROT13 |
| 3 | hex (UTF-8) |
| 4, 5, 6 | space-separated byte values in base 2, 10, 8 |
| 7 | percent-decoding |
| 8 | HTML character references |
| 9 | JSON string |
| 10 | Base32 |
| 12 | text reversal (character order, not alphabet) |
| 13 | Atbash |

The mapper checked the archive hash, that ZIP member names equalled the questions' `artifact` fields,
that every file had exactly 13 records numbered 1-13 in order, that each fragment decoded cleanly, that
every resulting slug matched the pattern, and that the mapped ids equalled the API's `answer_ids`.
All 63 answers were submitted in one call to `warmups-day-13/mastery` and were accepted. The ranked
rerun (2026-10-26) also went CORRECT in one call, 2.15 s end to end.

## Dead ends and gotchas

- **Mode suffix on the submission id.** Two early Time Trial submissions went to the bare
  `warmups-day-13` id and came back `already_solved`, scoring nothing; Time Trial and Mastery must be
  submitted under `warmups-day-13/time_trial` or `/mastery`. That was a process slip on our side (see
  the day overview), not a property of the puzzle.
- **Short timer versus recon.** The first Time Trial attempt expired during recon because the clock
  starts at `start`. With a 30-second window, hash and decode have to be scripted into the same pass.
- **Atbash is not reversal.** Record 12 in Mastery reverses character order; record 13 applies the
  alphabet-opposite map. They are different codecs and both are needed in the right order.
- **The Mastery artifact has a `.bin` name but is a ZIP.** Members are plain ASCII text, so
  `zipfile` works directly.
- No answer-format surprises: the bare slug passed the pattern check in every mode, and nothing was
  rejected.

## Takeaways

Warmup prompts are literal. When the wording describes a transform ("A met Z"), try that transform
first on the whole blob and read the full output, then take the requested line. For Mastery warmups,
look for per-record numbering and treat the number as the codec selector rather than auto-detecting.
Assert member names, record counts and answer-id sets before submitting; here that made the 63-field
submission a single call. Related techniques: Atbash substitution, multi-codec record chains.

## Evidence

Leaf base path: `challenges/day13/warmups/warmups-day-13/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-01.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/retry_response.json`,
  `time-trial/artifacts/retry_answer_map.py`, `time-trial/artifacts/retry-submission-response.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/submission-attempt-01.json`, `mastery/ranked-1/result.json`
- Library: WARMUP-ATBASH-001 (`library/warmups/atbash.md`) and the per-codec warmup entries named in the
  Mastery journal
- `progress.md` (grep `^| 13 .*warmups-day-13`)
