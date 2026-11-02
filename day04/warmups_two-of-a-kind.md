# Two of a Kind

| | |
|---|---|
| Day / Lane | 4 / Warmups |
| Challenge ID | `warmups-day-04` |
| Technique | Plain 8-bit binary text decode; Mastery adds numbered per-record codecs (base64, ROT13, hex, binary) |
| Single Solve | Solved (journal-recorded): the journal records our own `CORRECT` response; the leaf has no `result.json` |
| Time Trial | Independent (`submission-response.json` CORRECT, plus a `ranked-1/result.json` CORRECT) |
| Mastery | Independent (`submission-response.json` CORRECT, plus a `ranked-1/result.json` CORRECT) |

## Summary

A warmup whose prompt ("There are only two sides to this story") hints at binary. The Single Solve and
Time Trial artifacts are a single text blob of space-separated eight-bit groups; decoding them gives a
sentence ending in the answer slug. Mastery is the same idea scaled up: 18 blobs, each split into four
numbered records that use four different codecs, one of which is binary. One small solver covered all
three modes.

## The challenge

- One question per blob: decode it completely and submit the slug named by the final non-empty decoded
  line. Answer ID `message` (Single Solve, Time Trial) or `setN.message` for N = 1..18 (Mastery).
- Format: three lowercase words joined by hyphens, a hyphen, and a two-digit number; `trim`
  normalization. Every answer is a string.
- Single Solve and Time Trial deliver `blob.txt` (a 998-byte ASCII file with no line terminator).
  Mastery delivers `assigned_bundle.zip` (8,638 bytes as served) holding `blob-01.txt` .. `blob-18.txt`.
- Timing on the first (October 18) attempts: Single Solve untimed; Time Trial window 20 s; Mastery
  window 35 s. A plain `retry` was already the ranked attempt on this day, so the Time Trial and
  Mastery solves were scripted.

## Solution

1. **Recon.** `file` reports ASCII text with one 998-character line. The content is only `0`, `1` and
   spaces, in groups of eight, so it is a byte stream written out in binary. (998 = 111 groups x 8
   digits + 110 separators.)
2. **Decode.** Strip whitespace, cut into 8-bit chunks, `int(chunk, 2)` per byte, decode as UTF-8:

   ```python
   compact = re.sub(rb"\s+", b"", raw)
   decoded = bytes(int(compact[i:i+8], 2) for i in range(0, len(compact), 8))
   text = decoded.decode("utf-8")
   lines = [l.strip() for l in text.splitlines() if l.strip()]
   slug = re.search(r"\b[a-z]+-[a-z]+-[a-z]+-\d{2}\b$", lines[-1])
   ```

   The message is a fixed template, "Nice work decoding this warmup! To solve, submit this exact answer to
   the Game Master: " followed by the slug. The slug is the last non-empty line, as the question asked.
3. **Submit.** Single Solve answer `witty-starfish-turtle-96` came back `CORRECT`.

## Time Trial and Mastery

**Time Trial** used the identical blob format (answer `jaunty-daisy-pebble-47`), so the Single Solve
script ran unchanged. From the journal timestamps, the attempt started at 10:07:36.9 and was accepted
at 10:07:43.1, about 6 s inside a 20 s window. The 2026-10-26 ranked rerun took 1.95 s end to end, one
submit call.

**Mastery** changed the data, not the question. Each `blob-NN.txt` is a set of tab-separated lines
`NN<TAB>payload`, one codec per record number. Day 4 has four records:

| Record | Codec |
|---|---|
| 01 | base64 (padding stripped, re-added before decoding) |
| 02 | ROT13 |
| 03 | hex |
| 04 | 8-bit binary groups |

The decoded fragments are concatenated in order (the splits fall mid-word) and the slug is taken from the
end of the result. The solver dispatches on the record tag:

```python
if tag == "01":
    token = payload.encode("ascii"); token += b"=" * ((-len(token)) % 4)
    chunks.append(base64.b64decode(token, validate=True))
elif tag == "02":
    chunks.append(codecs.decode(payload, "rot_13").encode("ascii"))
elif tag == "03":
    chunks.append(bytes.fromhex(payload))
elif tag == "04":
    bits = re.sub(r"\s+", "", payload)
    chunks.append(bytes(int(bits[i:i+8], 2) for i in range(0, len(bits), 8)))
```

The same script also walks a ZIP (`zipfile` over each member, recursing), so one command produced all 18
slugs (e.g. `set1` `quick-daisy-trail-34`, `set18` `lucky-lantern-nugget-92`). All 18 answers were
submitted in a single call and accepted: the original attempt (started 10:07:48.5, accepted 10:08:09.9,
about 21 s into a 35 s window), and a ranked rerun on 2026-10-26 that took 5.17 s total, of which about 3 s
was the bundle download.

## Dead ends and gotchas

- **Local ZIP damage, not a challenge trick.** The saved `assigned_bundle.zip` was 54 bytes longer than
  the served file (every LF in the 18 members had become CRLF) and Python 3.14's `zipfile` raised
  `BadZipFile: Bad magic number`. Replacing `\r\n` with `\n` restored the exact SHA-256 from the start
  response. An earlier note blamed the server's ZIP headers; the 2026-10-24 re-check showed it was our
  own copy. Check the artifact hash before analyzing.
- **Mastery is not just binary.** The Single Solve/Time Trial format would not have decoded a Mastery
  blob; the per-record dispatch is what generalizes. We could not determine from the files why the day is
  called "Two of a Kind"; the prompt's "two sides" fits the binary encoding, nothing more was confirmed.
- **Template text in the journals** says "Android work was static: APKs were not installed or executed".
  That was copied from another lane and does not apply; these are plain text bundles.

## Takeaways

- Warmups of this kind are a layer-detection problem: look at the alphabet of the blob (`01 ` only means
  binary text) and decode from the downloaded bytes, never retyped ones.
- Numbered-record bundles (Mastery) fix the codec by record number, and the record count equals the day
  number here (4). A table-driven decoder beats per-blob guessing; the general version is the
  `codec_chain.py` tool in our warmups library ("numbered codec chain").
- Verify the SHA-256 of saved ZIPs before blaming the format.
- The Single Solve and Time Trial answers are per-attempt slugs, so values above will not match another
  team's.

## Evidence

Base path: `challenges/day04/warmups/warmups-day-04/`

- `single-solve/journal.md`, `single-solve/blob.txt`, `single-solve/artifacts/solve_blob.py` (the solver
  shown above, reused for all three modes)
- `time-trial/journal.md`, `time-trial/submission-response.json`, `time-trial/solver-output.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/start-response.json`, `mastery/submission-response.json`,
  `mastery/solver-output.json`, `mastery/ranked-1/result.json`
- `progress.md` rows for Day 4 ("Two of a Kind"): all three modes marked Solved, 2026-10-18
