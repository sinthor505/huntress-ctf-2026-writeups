# Seven Steps

| | |
|---|---|
| Day / Lane | 14 / Warmups |
| Challenge ID | `warmups-day-14` |
| Technique | known-offset Caesar shift (7); Mastery chains 14 numbered codecs, one per record |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (original attempt journal-recorded; two later ranked reruns also CORRECT) |
| Mastery | Independent (original attempt journal-recorded; two later ranked reruns also CORRECT) |

## Summary

The prompt was one line, "Caesar took a week off.", and it meant what it said: shift every letter
back seven places. Single Solve and Time Trial each delivered one Caesar-shifted sentence ending in
the answer slug. Mastery delivered 68 small files, each a 14-record codec chain whose last record
is the same Caesar-7. All three modes went CORRECT on the first scored submission, with one
exception: the first Mastery attempt expired before we submitted.

## The challenge

Prompt: "Caesar took a week off." Every mode asked the same question per field: decode the blob
completely and submit the slug that its final non-empty decoded line tells you to submit. The
answer format was a pattern (three lowercase words joined by hyphens, a hyphen, then two digits),
with `trim` normalization.

- **Single Solve:** one `blob.txt` (a single line), one answer field `message`, untimed.
- **Time Trial:** same shape, one answer field, a 30-second window.
- **Mastery:** a ZIP of 68 numbered blobs (`blob-01.txt` ... `blob-68.txt`), answer fields
  `set1.message` ... `set68.message`, an 85-second window.

## Solution

1. **Read the hint literally.** "A week" is seven, and Caesar is a rotation. The Single Solve blob
   was English with its capitalization and punctuation intact (`Upjl dvyr kljvkpun aopz dhytbw! ...`),
   which is what a letter-only substitution leaves behind. The scout did not need to run a cipher
   detector.
2. **Shift back by seven.** Every alphabetic character was moved seven places backward, wrapping
   mod 26 and leaving non-letters alone. The output was coherent:
   `Nice work decoding this warmup! To solve, submit this exact answer to the Game Master: dapper-sprout-boulder-71`.
3. **Submit the slug only.** The format description says "submit the slug only", so the value sent
   was `dapper-sprout-boulder-71`, not the sentence. CORRECT on the first call.

Time Trial was the same transform on a fresh blob and gave `clever-daisy-laurel-27`, also CORRECT
on the first call (accepted about 4.4 s after the attempt started, inside the 30 s window).

## Time Trial and Mastery

Time Trial changed nothing technically; the window just meant the decode had to be one script run
against the downloaded bytes rather than done by hand.

Mastery is where the title earns its name. Each of the 68 files holds 14 tab-separated records
numbered `01` to `14`, and the record number fixes the codec:

| Record | Codec | Record | Codec |
|---|---|---|---|
| 01 | Base64 | 08 | HTML character references |
| 02 | ROT13 | 09 | JSON string (`\uXXXX`) |
| 03 | hex | 10 | Base32 |
| 04 | binary (8-bit groups) | 11 | Base64 (unpadded) |
| 05 | decimal bytes | 12 | text reversal |
| 06 | octal bytes | 13 | Atbash |
| 07 | percent-encoding | 14 | Caesar, back 7 |

The decoded fragments are slices of one sentence, so the order matters: concatenating the 14 pieces
gives the same "Nice work decoding this warmup! ... submit this exact answer to the Game Master: <slug>"
text as the earlier modes, with the slug straddling the last three records (reversed, Atbash and Caesar
pieces). The offline mapper took everything after the final `:` as the slug and checked it against
the answer pattern.

The Caesar branch of the mapper was the only piece that had no library entry, so it was written
out directly:

```python
if number == 14:
    return "".join(
        chr((ord(c) - ord("a") - 7) % 26 + ord("a")) if "a" <= c <= "z"
        else chr((ord(c) - ord("A") - 7) % 26 + ord("A")) if "A" <= c <= "Z"
        else c
        for c in payload
    )
```

The mapper also verified the archive SHA-256 against the start response, that the ZIP members
exactly matched the question `artifact` names, that record numbers ran 1 to 14 in order, and that
the final key set equalled the API's `answer_ids`.

The first Mastery attempt (85 s window) was submitted after its deadline and returned
`expired`; nothing was graded. A `retry` issued a new variant with a fresh 85 s window and 68 new
blobs, and the same mapper produced all 68 answers, accepted in one call about 11 s after the
retry started.

## Dead ends and gotchas

- **Expired Mastery window.** The only loss in this challenge was timing, not decoding: the first
  Mastery submission arrived after its 85 s deadline (the journal does not say why). The retry
  variant was different data, so nothing from the first variant carried over.
- **Do not decode with a Caesar tool and stop.** In Mastery the Caesar shift is only the last
  record. Applying it to a whole file would have turned the other 13 records into garbage; the
  record number, not the content, selects the codec.
- **Slug extraction.** The final line is a sentence, and the slug is whatever follows the last colon.
  The `trim` normalization is the only forgiveness, so strip whitespace and nothing else.

## Takeaways

- For warmups, take the hint literally first ("a week" = shift 7); a one-line prompt usually names
  the transform.
- Preserved capitalization and punctuation with unreadable letters points at a monoalphabetic
  substitution or rotation, not an encoding such as Base64.
- Mastery warmups on this day: "record number selects the codec, and Day N has N records" held
  here (Day 14 has 14). Build one numbered-codec pipeline and validate structure (record count and
  order, member-to-question mapping, answer pattern) before submitting. A signed-shift Caesar
  transform was promoted to the library as `WARMUP-CAESAR-SHIFT-001`.

## Evidence

Leaf: `challenges/day14/warmups/warmups-day-14/`

- Single Solve: `single-solve/journal.md` records our own submission as CORRECT; no result file was
  saved. `single-solve/start_response.json`, `single-solve/blob.txt`.
- Time Trial: `time-trial/journal.md` (original attempt, CORRECT, journal-recorded);
  `time-trial/ranked-1/result.json` and `ranked-2/result.json` (CORRECT, one call each, 30 s window,
  about 2 to 2.6 s total).
- Mastery: `mastery/journal.md`; `mastery/artifacts/answer_map.py` (the mapper),
  `candidate-answers-retry.json`, `submission-attempt-log.json` (expired, then CORRECT);
  `mastery/ranked-1/result.json` and `ranked-2/result.json` (CORRECT, 68 answers, one call each).
- `progress.md` rows 14 (`warmups-day-14` and its `/time_trial`, `/mastery`): all three modes marked Solved.
- Ranked reruns were bragging-rights attempts, not the prize ranking.
