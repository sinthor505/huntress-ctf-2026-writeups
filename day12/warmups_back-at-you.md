# Back at You

| | |
|---|---|
| Day / Lane | 12 / Warmups |
| Challenge ID | `warmups-day-12` |
| Technique | character-order reversal (Single Solve, Time Trial); 12-codec numbered fragment chain (Mastery) |
| Single Solve | Independent (journal and `submission-attempt-01.json`, CORRECT on call 1) |
| Time Trial | Independent (journal and `submission-attempt-01.json`, CORRECT on call 1); a later ranked rerun was not possible |
| Mastery | Independent (journal and `submission-attempt-01.json`, CORRECT on call 1); a later ranked rerun was not possible |

## Summary

"Back at You" is a Warmups challenge whose prompt ("Maybe we started at the wrong end.") says what
the transform is. The Single Solve and Time Trial blobs were one ASCII line written backward;
reversing it produced an instruction that ended in the answer slug. The Mastery bundle scaled that
up to 58 blobs of 12 numbered fragments, each fragment using a different encoding. All three modes
went CORRECT on the first submission.

## The challenge

Every mode asks the same thing, once per blob: "Decode the assigned blob completely. What exact
answer slug does its final non-empty decoded line instruct you to submit?" The answer format is a
pattern: three lowercase words joined by hyphens, then a hyphen and a two-digit number, slug only,
with `trim` normalization.

- **Single Solve:** one question (`message`), one text blob (`blob.txt`), untimed.
- **Time Trial:** same shape, fresh blob, 30-second window (started 18:58:54, deadline 18:59:24).
- **Mastery:** a ZIP of 58 entries (`blob-01.txt` to `blob-58.txt`), 58 questions
  (`set1.message` to `set58.message`), 75-second window.

## Solution

### Single Solve

1. **Look at the bytes.** `xxd` and `strings` showed one ASCII line with no trailing newline and
   text that read backward. The prompt's "wrong end" wording matched.
2. **Reverse the whole string.** Reversing the entire byte string gave an English instruction
   ending in the slug. Because there is no newline, that single line is also the "final non-empty
   decoded line".
3. **Take the slug.** The instruction ends in `...: <slug>`, so the mapper splits on the last
   colon and trims:

   ```python
   decoded = text[::-1]
   lines = [l for l in decoded.splitlines() if l.strip()]
   slug = lines[-1].rsplit(":", 1)[-1].strip()
   print(json.dumps({"message": slug}))
   ```

   The Single Solve answer was `zesty-sunset-quartz-65`.

The solver (`answer_map.py`) is deliberately tiny: read, reverse, take the last non-empty line,
split on the last colon.

### Time Trial

The fresh blob was the same reversed ASCII text. The same `answer_map.py` was reused unchanged and
the map was submitted inside the window; CORRECT on call 1.

## Time Trial and Mastery

Mastery is where the challenge actually grew. Each `blob-NN.txt` is a set of 12 tab-separated
lines, `N<TAB>payload`, and the record number selects the codec. Decoding every fragment and
concatenating the results rebuilds a full instruction whose last colon-separated field is the
slug. The repeated layout across all 58 blobs was:

| Record | Codec |
|---|---|
| 1 | Base64 |
| 2 | ROT13 |
| 3 | hex bytes |
| 4 | binary bytes (space-separated) |
| 5 | decimal bytes |
| 6 | octal bytes |
| 7 | percent-encoding |
| 8 | HTML character references |
| 9 | JSON string escapes |
| 10 | Base32 |
| 11 | Base64 |
| 12 | character reversal (the Single Solve transform) |

The mapper dispatches on the record number rather than guessing a codec per fragment, walks the ZIP
entries in sorted order, and derives the answer id from the file name (`blob-07.txt` becomes
`set7.message`). It also asserts every slug matches
`[a-z]+-[a-z]+-[a-z]+-\d{2}`, so a wrong codec on any fragment fails loudly instead of submitting
garbage. All 58 blobs passed that check, and the full 58-field map was accepted on the first call.

Two details worth keeping: the numbering is a property of the day (Day 12 has 12 records), and the
last record being plain reversal is what ties the Mastery chain back to the Single Solve prompt.

## Dead ends and gotchas

- No rejected answers: all three modes were CORRECT on submission 1, so there was no validator
  guesswork to record.
- The library had no entry for character reversal when the Single Solve began, so the transform was
  established directly from the bytes and the prompt, then promoted afterward as a new entry.
- Short windows were the real constraint. The 30 s and 75 s deadlines meant the solve had to be
  a script run in one pass, which is why the Mastery mapper checks formats itself.
- Ranked reruns were not possible later. A ranked Time Trial pass found `unique_attempts_remaining:
  0` and returned a practice payload instead of issuing an attempt, and a ranked Mastery start
  likewise read `practice_only` with zero attempts remaining. Those folders hold no solve; the
  CORRECT results above come from the original attempts.

## Takeaways

- **Reading the prompt literally pays off in Warmups.** "Started at the wrong end" meant reversal.
  Check the cheapest whole-text transforms (reversal, ROT13, Base64) first.
- **Numbered-fragment codec chains** (Warmups Mastery): the record number fixes the codec, so build a
  dispatch table keyed by number and add a slug-format assertion per blob.
- Slug extraction rule that held: take the last non-empty decoded line, split on the last `:`, trim.

## Evidence

Leaf base path: `challenges/day12/warmups/warmups-day-12/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-attempt-01.json`
- `time-trial/journal.md`, `time-trial/start_response.json`,
  `time-trial/artifacts/submission-attempt-01.json`, `time-trial/ranked-1/result.json`
  (`practice_only`)
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/submission-attempt-01.json`, `mastery/ranked-1/result.json` (`practice_only`)
- Library: `WARMUP-TEXT-REVERSAL-001`, `WARMUP-TEXT-CODECS-001`
- `progress.md` (grep `^| 12 .*warmups-day-12`)
