# Bacon Bits

| | |
|---|---|
| Day / Lane | 17 / Warmups |
| Challenge ID | `warmups-day-17` |
| Technique | Baconian-looking a/b groups that are really 5-bit hex nibbles; numbered multi-codec record chain |
| Single Solve | Solved (journal-recorded; CORRECT on 1 call, no result file saved) |
| Time Trial | Independent (journal-recorded CORRECT, 1 call; later ranked reruns saved as `ranked-1/` and `ranked-2/` `result.json`) |
| Mastery | Independent (journal-recorded CORRECT, 1 call; later ranked reruns saved as `ranked-1/` and `ranked-2/` `result.json`) |

## Summary

The Single Solve and Time Trial blobs are plain text made of five-letter `a`/`b` groups, which
looks like a Baconian cipher. It is not: read as 5-bit integers, every group is 0-15, so each is a
hex nibble. Pairing nibbles into bytes gives a UTF-8 message whose last line holds the answer
slug. Mastery ships 83 blobs, each a 17-record chain of different encodings with the same nibble
codec as the last record. Everything was CORRECT on the first submission in all three modes.

## The challenge

Prompt: "Sixteen guests. Five little a's and b's apiece." The title and prompt point at Baconian
(Bacon's) cipher: 5-letter a/b groups. The "sixteen guests" line is the real hint: only sixteen
distinct values occur, i.e. a nibble alphabet. We read that as a hint only after the fact; the
journal does not say whether it helped.

Each mode delivered a static blob, verified against the `sha256` in the start response:

- Single Solve: `warmups-day-17.bin`, 1,295 bytes, one answer field (`message`).
- Time Trial: a fresh 1,283-byte blob, one answer field.
- Mastery: a 49,038-byte ZIP of 83 blobs (`blob-01.txt` ... `blob-83.txt`), 83 answer fields
  (`set1.message` ... `set83.message`).

Every question asked: decode the blob completely and submit the slug the final non-empty decoded
line instructs, format "three lowercase words separated by hyphens, followed by a hyphen and a
two-digit number", normalization `trim`.

## Solution

1. **Look at the blob.** 216 space-separated groups of five letters, only `a` and `b`.
2. **Test the Baconian reading.** A 26-letter Baconian alphabet decoded to garbage. Mapping
   `a=0, b=1` and reading each group as a 5-bit integer showed a maximum of 15 across the whole
   blob, so the codec is "hex nibble in five bits", not a letter code.
3. **Pair the nibbles.** An even count (216) gives 108 bytes: `byte = hi*16 + lo`. Decoded as
   UTF-8 this is a "Nice work decoding this warmup!" message; the slug is the text after the
   last colon on the last non-empty line.
4. **Submit.** The extractor asserts the nibble bound, the even count and the slug pattern, so a
   wrong reading fails loudly instead of producing a plausible string:

   ```python
   v = [int(g.replace("a", "0").replace("b", "1"), 2) for g in data.decode("ascii").split()]
   assert max(v) < 16 and len(v) % 2 == 0
   text = bytes(v[i] * 16 + v[i + 1] for i in range(0, len(v), 2)).decode("utf-8")
   slug = [l for l in text.splitlines() if l.strip()][-1].rsplit(":", 1)[-1].strip()
   assert re.fullmatch(r"[a-z]+-[a-z]+-[a-z]+-\d{2}", slug), slug
   ```

   The Single Solve slug was `tidy-rocket-harbor-20` and the submission was CORRECT on call 1
   (flag returned by the Game Master was `e6a384e7341c389fe2bc60ff0386325a`).

## Time Trial and Mastery

**Time Trial** was the same blob type with a different payload; the same mapper went CORRECT on
call 1. The attempt we solved for the record was untimed (`deadline: null`).

**Mastery** changed the container, not the nibble codec. Each of the 83 blobs is 17
tab-separated numbered records (`N<TAB>payload`), and the record number fixes the codec. From
the journal:

| Record | Codec |
|---|---|
| 1, 11 | base64 |
| 2 | ROT13 |
| 3 | hex |
| 4, 5, 6 | binary, decimal, octal byte lists |
| 7 | percent-encoding |
| 8 | HTML character references |
| 9 | JSON string |
| 10 | base32 |
| 12 | string reversal |
| 13 | Atbash |
| 14 | Caesar, shift -7 |
| 15 | ROT47 |
| 16 | Morse for hex digits |
| 17 | a/b nibbles (the Single Solve codec) |

Each record decodes to a fragment; the fragments are concatenated in record order and the slug is
the text after the last colon. We reused our existing numbered-codec decoder, changing its
expected record count to 17 (it had been written for a longer chain). Every one of the 83 slugs
matched the strict slug regex, and the whole 83-answer batch was CORRECT in one submission.

Afterwards, two ranked (timed) reruns of Time Trial and of Mastery were also CORRECT on call 1
(ranked-1: 30 s window, 2.4 s total for Time Trial; 100 s window, 7.2 s for Mastery; ranked-2:
1.3 s and 1.66 s). Those were speed-scoreboard attempts, not the prize measure.

## Dead ends and gotchas

- **The title is a decoy.** "Bacon" suggests the classic 26-letter Baconian alphabet; the right
  reading is 5 bits per group interpreted as a nibble. The tell is the maximum value: a Baconian
  letter code would reach 25 (and 24 in the 24-letter variant), but this blob never exceeds 15.
- **Mastery chain length differs from neighbouring days.** The 17-record chain stops at the
  nibble codec; there are no records 18 or 19 (those appear in later days' chains). A decoder
  that hard-codes the record count rejects the bundle. Check the real count from the file.
- No format surprises were recorded: the bare slug was accepted with `trim` normalization.

## Takeaways

- Technique name: **a/b groups as hex nibbles** (Baconian-looking binary alphabet). Before
  decoding a "Baconian" blob, compute the maximum 5-bit value; if it is under 16, pair nibbles
  into bytes rather than mapping to letters.
- Assert invariants in the extractor (value bound, even count, slug regex) so a wrong codec guess
  fails immediately.
- For numbered-record Mastery bundles, derive the record count and per-record codec from the
  file, not from a previous day's decoder.

## Evidence

Leaf base: `challenges/day17/warmups/warmups-day-17/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/candidate-answers.json`,
  `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`
  (its docstring still names an older day; it enforces 17 records),
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `library/warmups/bacon-nibbles-d17.md` (WARMUP-D17-BACON-NIBBLES-001)
- `progress.md` (grep `^| 17 .*warmups-day-17`)
