# Big Entrance

| | |
|---|---|
| Day / Lane | 20 / Warmups |
| Challenge ID | `warmups-day-20` |
| Technique | layered text decoding: hex of UTF-16 with a big-endian BOM; numbered codec chain (Mastery) |
| Single Solve | Solved (journal-recorded; the attempt log also reads CORRECT, no result file saved) |
| Time Trial | Independent (`ranked-1` and `ranked-2` `result.json` CORRECT; the original untimed attempt is journal-recorded) |
| Mastery | Independent (`ranked-1/result.json` CORRECT; the original untimed attempt is journal-recorded) |

## Summary

"Big Entrance" is the Day 20 warmup. The blob is ASCII hex that decodes to UTF-16 text, and the
byte-order mark says big-endian, which fits the prompt ("the big one goes first"). Single Solve and
Time Trial are one decode. Mastery ships 98 blobs, each wrapped in 20 numbered transform records.
Record 20 is the new big-endian hex step, and record 18 turned up with a block length we had not
seen before. All three modes went CORRECT on the first submission.

## The challenge

Prompt: "Same bar. This time the big one goes first." Each question asked for the exact answer slug
that the final non-empty line of the fully decoded blob tells you to submit. The answer format was a
pattern: three lowercase words joined by hyphens, then a hyphen and a two-digit number, slug only,
with `trim` normalization.

- **Single Solve and Time Trial:** one question (`message`) and one `blob.txt`, SHA-256 given in the
  start response.
- **Mastery:** a ZIP of 98 blobs (`blob-01.txt` to `blob-98.txt`) with 98 questions `setN.message`.
  Each blob holds 20 numbered records.

## Solution

1. **Recognise the container.** The Single Solve blob is hex text starting `feff`, and every second
   byte is `00`. That is a UTF-16 byte-order mark with ASCII-range text, so the blob is hex wrapping
   UTF-16 with the BOM selecting big-endian.
2. **Decode.** `bytes.fromhex(text).decode("utf-16")` is enough, because Python's `utf-16` codec
   reads the BOM to choose the endianness. That is the same one-liner as the previous day's
   little-endian (`fffe`) variant. The result is the usual warmup sentence ("Nice work decoding
   this warmup! To solve, submit this exact answer to the Game Master: ...").
3. **Take the slug.** The slug is the last token of the last non-empty line. The extractor checked
   it against the strict pattern before printing JSON:

   ```python
   text = bytes.fromhex(blob.decode("ascii").strip()).decode("utf-16")
   slug = [l for l in text.splitlines() if l.strip()][-1].split()[-1]
   assert re.fullmatch(r"[a-z]+-[a-z]+-[a-z]+-\d{2}", slug), slug
   ```

   The Single Solve answer was `deft-blanket-clover-43`. It was submitted in a separate, authorized
   step and was CORRECT on attempt 1.

## Time Trial and Mastery

**Time Trial** was the same decode on a fresh blob (SHA-256 checked against the start response). The
answer was `crisp-compass-coral-68`, CORRECT on attempt 1. The first attempt was untimed. We also
ran two ranked attempts later. Each had a 30 s window, solved in about 2 s total, and was CORRECT in
one call. Ranked runs were bragging rights only and not the prize scoring.

**Mastery** is where the day's work was. The previous day's numbered-chain decoder handled records
1-19 unchanged, and the record-to-codec table was fixed per record number:

- 1/11 base64, 2 ROT13, 3 hex, 4/5/6 binary/decimal/octal byte lists, 7 percent-encoding,
  8 HTML entities, 9 JSON string, 10 base32, 12 reverse, 13 Atbash, 14 Caesar -7, 15 ROT47-style
  shift, 16 Morse digits/hex letters to hex, 17 Baconian a/b groups read as hex nibbles,
  18 block permutation, 19 hex of UTF-16LE.
- **New record 20:** hex of UTF-16 with the `feff` BOM (big-endian). The same `fromhex(...).decode("utf-16")`
  call handles it.

The only open question was record 18. In 8 of the 98 blobs its blocks were 5 characters long instead
of 6. The journal records that the 6-character permutation `out[i] = in[[0,2,5,3,1,4][i]]` was
already known and that the 5-character one, `[0,2,4,3,1]`, was inferred from context. The decoded
words ("rain" + "nbow-", "cal" + "m-roc" + "ket" once reordered) had to read as the usual
adjective-noun-noun slugs. All 98 slugs matched the strict pattern, and the one submission with 98
fields was CORRECT. The ranked Mastery rerun (115 s window, about 2.7 s total) was also CORRECT in
one call.

## Dead ends and gotchas

- **Record 18 was a guess when we submitted.** We described the 5-character block permutation as
  inferred from word plausibility, not derived. It happened to be right, but nothing in the data
  said so. Treat an unseen block size as a hypothesis until the output reads as words.
- **Record 18 is a 3-rail rail fence, not an ad hoc permutation.** We worked this out afterwards. A
  3-rail fence over 5 characters unscrambles with `[0,2,4,3,1]`, and over 6 characters with
  `[0,2,5,3,1,4]`; we re-derived both index lists to check. The two "permutations" were the same fence on
  shorter strings, and payload lengths of 5-7 occurred on Days 18-20. We later generalized the decoder
  to a rail fence for that reason.
- **Mastery has `5N-2` blobs on Day N**, which is 98 on Day 20, and Day N has N records, so the record
  table is a growing prefix. Check the blob count against the question list before decoding.

## Takeaways

- Technique names to search for: **hex of UTF-16 with BOM** (the `fffe`/`feff` pair), the
  **numbered codec chain** and **3-rail rail fence** as the "block permutation".
- In a layered warmup, let the record number choose the codec instead of auto-detecting per line. When
  one record misbehaves, decode its neighbours first, then use the known message template
  ("Nice work decoding this warmup! ...") to test candidate transforms.
- If a transform "permutes blocks" and the block size varies, check whether it is a classical
  transposition (rail fence, columnar) before inferring one permutation per size.
- Single Solve and Time Trial were one-liners. A faster path is a generic decoder that peels `hex`
  then `bom` automatically.

## Evidence

Leaf base: `challenges/day20/warmups/warmups-day-20/`.

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/candidate.json`, `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py` (the Day 20 numbered-chain decoder),
  `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- Library entries: WARMUP-DAY19-HEX-UTF16-001, WARMUP-NUMBERED-CODEC-CHAIN-001 (record 18 correction)
- `progress.md` (grep `^| 20 .*warmups-day-20`)
- No librarian note or lane log exists for this challenge in `challenges/day20/_run/`; that folder
  holds only the release list and watcher log.
