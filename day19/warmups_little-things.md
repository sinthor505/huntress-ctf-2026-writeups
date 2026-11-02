# Little Things

| | |
|---|---|
| Day / Lane | 19 / Warmups |
| Challenge ID | `warmups-day-19` |
| Technique | layered text decoding: hex of UTF-16LE with BOM; numbered per-record codec chain (Mastery) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Little Things" is a decode-the-blob warmup. The hint is the little-endian trap: the Single Solve and
Time Trial blobs are ASCII hex of UTF-16LE text with a `ff fe` BOM, and the slug is the last token of
the decoded message. Mastery scaled this to 93 blobs of 19 numbered records each, four of which were
new codecs, and went CORRECT on the first submission.

## The challenge

Prompt: "Two bytes walk into a bar. The little one goes first." Each question asked for the exact answer
slug the blob's final non-empty decoded line tells you to submit. `answer_format` was a `pattern`: three
lowercase words separated by hyphens, then a hyphen and a two-digit number (`word-word-word-NN`), slug
only.

- **Single Solve:** one 436-byte text file `blob.txt`, one answer `message`.
- **Time Trial:** one 440-byte blob of the same shape, one answer `message`.
- **Mastery:** a ZIP of 93 files `blob-01.txt` ... `blob-93.txt` (54,118 bytes), 93 answers
  `set1.message` ... `set93.message`.

## Solution

1. **Check the artifact.** SHA-256 matched the start response before any decoding.
2. **Recognize the encoding.** The blob was a hex string starting `fffe` with every second byte `00`:
   UTF-16LE with a byte order mark. The prompt's "little one goes first" points at the byte order.
3. **Decode.** `bytes.fromhex(...)` followed by `.decode("utf-16")` lets the BOM choose the endianness:

   ```python
   raw = bytes.fromhex(t)
   txt = raw.decode("utf-16")          # BOM fffe -> UTF-16LE
   lines = [l for l in txt.splitlines() if l.strip()]
   print(json.dumps({"answer": lines[-1].split()[-1]}))
   ```

   The text read "Nice work decoding this warmup! ... Game Master: <slug>"; the slug is the last token of
   the last line. The Single Solve answer was `jaunty-raven-glade-73`.
4. **Submit.** One call, CORRECT.

## Time Trial and Mastery

**Time Trial** used the same codec on a fresh blob, so the same extractor was reused unchanged. The
regular (untimed) attempt went CORRECT on call 1. A later ranked attempt (30 s window) also went CORRECT
on call 1; the script spent about 2.1 s in total, nearly all of it in the issue and submit round trips.

**Mastery** is where the work was. Each of the 93 blobs held 19 tab-numbered records (`NN<TAB>payload`),
and the record number fixes the codec. Decoding every record and concatenating the fragments gave "Nice
work decoding this warmup! To solve, submit this exact answer to the Game Master: <slug>". Fragment
boundaries do not line up with codec boundaries, so only the concatenation is meaningful; the slug is
what follows the last colon.

Records 1-15 were codecs already seen on Day 15 (base64, ROT13, hex, binary/decimal/octal byte lists,
percent-encoding, HTML references, JSON string, base32, base64, reversal, Atbash, Caesar by 7, ROT47-style
shift by 47). Four were new on Day 19:

- **16:** Morse code for the hex digits; decode Morse to hex, hex to bytes, bytes to text.
- **17:** five-letter `a`/`b` groups in the style of the Baconian cipher, but each group is a 5-bit value
  used as one hex nibble, not a letter.
- **18:** a fixed permutation inside 6-character blocks, `out[i] = in[[0,2,5,3,1,4][i]]`.
- **19:** the Single Solve codec, hex of UTF-16LE with BOM.

The extractor validated the SHA-256, required exactly 19 records numbered 1-19 per file, required the ZIP
members to match the question artifacts, and required every slug to match the strict `word-word-word-NN`
regex before it would print anything. All 93 answers went in as one submission: CORRECT, 93 of 93. A later
ranked attempt (110 s window, fresh bundle) was also CORRECT on call 1, with about 2.5 s of total script
time.

## Dead ends and gotchas

- **Record 18 was inferred, not read.** The block permutation was reconstructed from two blobs by putting
  known slug words back together; the check was that all 93 slugs then matched the strict regex with
  dictionary-like words. A later review found it is a 3-rail rail fence (checked on all 93 payloads),
  which is the same transform expressed as a standard cipher.
- **Fragment boundaries do not align with codecs.** Decoding a record in isolation can yield half a
  word; always concatenate all 19 decoded fragments before looking for the slug.
- **Bytes versus nibbles in record 17.** Reading the `a`/`b` groups as Baconian letters gives
  nonsense; each 5-bit value is a single hex digit.
- **Slug only.** The answer is the bare slug, not the whole sentence or the `Game Master:` prefix.

## Takeaways

- A hex blob that begins `fffe` (or `feff`) is UTF-16 with a BOM: `bytes.fromhex(t).decode("utf-16")`
  and let the BOM pick the endianness.
- For numbered-record warmups, build a table-driven chain keyed on the record number and fail loudly on
  any slug that does not match the answer pattern. That regex gate is what made the inferred record 18
  safe to submit on one call.
- Later days' warmups reuse and extend this chain (the next day uses the big-endian `feff` variant), so
  add each new codec to a shared table instead of to a one-off script.

## Evidence

Leaf: `challenges/day19/warmups/warmups-day-19/`

- `single-solve/journal.md`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-attempt-log.json` (CORRECT, attempt 1)
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json` (CORRECT)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json` (CORRECT, 93 answers)
- `library/warmups/hex-utf16-bom.md` (WARMUP-DAY19-HEX-UTF16-001), `library/warmups/numbered-codec-chain/` (WARMUP-NUMBERED-CODEC-CHAIN-001)
- `progress.md` (grep `^| 19 .*warmups-day-19`)
