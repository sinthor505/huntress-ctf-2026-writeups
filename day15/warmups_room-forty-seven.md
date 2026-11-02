# Room Forty-Seven

| | |
|---|---|
| Day / Lane | 15 / Warmups |
| Challenge ID | `warmups-day-15` |
| Technique | ROT47 (rotate printable ASCII 33-126 by 47); numbered multi-codec record chain in Mastery |
| Single Solve | Independent (journal plus saved submission response reading CORRECT) |
| Time Trial | Independent (journal, saved response, and a ranked rerun `result.json` CORRECT) |
| Mastery | Independent (journal, saved response, and two ranked reruns, both CORRECT) |

## Summary

The prompt was "Even the punctuation went for a spin." That was a literal hint: the blob's
letters, digits and punctuation were all scrambled while spaces stayed in place, which is the
signature of ROT47 rather than ROT13. Single Solve and Time Trial were one inverse-ROT47 pass over
a ~100-byte blob. Mastery was 73 blobs of 15 numbered records each, one codec per record number,
with ROT47 as only the last record. Every mode was CORRECT on the first submission.

## The challenge

One question per blob (`message`): decode the assigned blob completely and submit the slug its
final non-empty decoded line tells you to submit. Answer format: three lowercase words joined by
hyphens, then a hyphen and a two-digit number; slug only; `trim` normalization.

- Single Solve: a 107-byte `blob.txt`, untimed.
- Time Trial: a fresh 109-byte blob, 30 s server window.
- Mastery: a ZIP (34,482 bytes) with 73 members `blob-01.txt` ... `blob-73.txt`, one per answer id
  `setN.message`, 90 s window.

## Solution

1. **Recognize the transform.** The artifact was printable ASCII where letters, digits and
   punctuation were all mangled but spaces were intact. ROT13/Caesar would have left digits and
   punctuation untouched, so it had to be something over the full printable range. The prompt
   pointed the same way.
2. **Test the hypothesis on the real bytes.** Rotating every byte in 33..126 by 47 (the transform
   is its own inverse because 47 is half of 94) turned the Single Solve blob into:
   `Nice work decoding this warmup! To solve, submit this exact answer to the Game Master: sunny-castle-opal-96`.
   The readable crib sentence confirmed the shift.
3. **Extract the slug** from the last non-empty decoded line (`sunny-castle-opal-96`) and submit it.

```python
def rot47(data: bytes) -> bytes:
    return bytes(((b - 33 - 47) % 94 + 33) if 33 <= b <= 126 else b for b in data)
```

Space and any other byte outside 33-126 pass through unchanged. The mapper then runs a slug regex
over the final non-empty line (`[a-z]+-[a-z]+-[a-z]+-[0-9]{2}`), so a wrong decode fails loudly
instead of submitting garbage. The first submission was CORRECT.

## Time Trial and Mastery

**Time Trial** was the same transform on a fresh blob. The same mapper, unchanged, was accepted
on the first submission inside the 30 s window. A later ranked rerun (new attempt) took 1.44 s end
to end and was also CORRECT.

**Mastery** was a different shape. Each of the 73 blobs held 15 tab-separated records
`NN<TAB>payload`, and the record number selected the codec. The decoded fragments, concatenated in
order, spell out the same crib sentence ending in the slug:

| Record | Codec | Record | Codec |
|---|---|---|---|
| 1 | Base64 | 9 | JSON string escapes |
| 2 | ROT13 | 10 | Base32 |
| 3 | hex | 11 | Base64 |
| 4 | binary (space-separated) | 12 | reversal |
| 5 | decimal bytes | 13 | Atbash |
| 6 | octal bytes | 14 | Caesar, back 7 |
| 7 | percent-encoding | 15 | ROT47 (inverse) |
| 8 | HTML numeric entities | | |

The Mastery mapper (`answer_map.py`) did several consistency checks before emitting anything:
archive SHA-256 against the start response, ZIP member names exactly equal to the question
`artifact` fields, exactly 15 records per member numbered 1..15 in order, answer ids equal to the
API's `answer_ids`, and the final slug matching the format. It took the text after the last `:` of
the concatenated message as the answer. All 73 fields were accepted in one call. The saved ranked
reruns (two separate attempts) were also CORRECT, one call each, at roughly 1.4-2.2 s total.

## Dead ends and gotchas

- `WARMUP-CAESAR-SHIFT-001`-style letter-only rotation was ruled out immediately: it leaves digits
  and punctuation wrong, which is exactly the tell here. The workspace library had no ROT47 entry
  at the time, so this day added one (`WARMUP-ROT47-001`).
- In Mastery, whitespace inside fragments matters. Fragments were concatenated verbatim, so
  stripping or re-joining them with a separator would corrupt the message before the slug.
- Per-record decoding must be driven by the record number, not by guessing the codec from the
  payload; several payloads (hex, Base64, Base32) are lookalikes at a glance.
- No answer-format surprises: the plain slug was accepted in all three modes. Nothing was
  rejected.

## Takeaways

When a blob's punctuation and digits are scrambled but whitespace survives, try **ROT47** before
anything exotic, and test it by looking for the crib sentence. For Mastery warmups, a numbered
record chain is a strict pipeline: validate record count and order, decode each by number,
concatenate, and fail on a malformed slug. Rolling the ROT47 step into a shared codec-chain tool
(which later days' Mastery blobs reused as record 15) was the one change that mattered afterwards.
Faster next time: the first look at the bytes was enough to name the transform, so the crib test
could go straight to ROT47 without trying Caesar variants.

## Evidence

Leaf base: `challenges/day15/warmups/warmups-day-15/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-attempt-01-response.txt`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`,
  `time-trial/artifacts/submission-attempt-01-response.txt`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py` (15-codec chain),
  `mastery/artifacts/submission-attempt-01-response.txt`, `mastery/ranked-1/result.json`,
  `mastery/ranked-2/result.json`
- `library/warmups/rot47.md` (WARMUP-ROT47-001)
- `progress.md` (grep `^| 15 .*warmups-day-15`)
