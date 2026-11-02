# Hex Appeal

| | |
|---|---|
| Day / Lane | 3 / Warmups |
| Challenge ID | `warmups-day-03` |
| Technique | Layered text encodings (hex single blob; numbered base64 / ROT13 / hex records in Mastery) |
| Single Solve | Solved, graded result not preserved (player-directed local mark, 2026-10-18; no leaf directory) |
| Time Trial | Independent (CORRECT, 1 submission, 2026-10-20; ranked rerun CORRECT, 2026-10-26) |
| Mastery | Independent (CORRECT, 1 submission, 2026-10-20; ranked rerun CORRECT, 2026-10-26) |

## Summary

A warmup whose prompt ("Sixteen seems like a perfectly reasonable number of fingers.") says the
encoding: base 16. The Time Trial blob is plain hex of a UTF-8 message, and the answer is the
slug on the last decoded line. Mastery repeats that across 13 bundled blobs, each cut into
numbered records in three different codecs. Both modes were solved with a short offline script on
the first submission. The Single Solve was marked solved locally on 2026-10-18, but no graded
result was kept, so this page cannot show how it was done.

## The challenge

Every mode asks the same question: "Decode the assigned blob completely. What exact answer slug
does its final non-empty decoded line instruct you to submit?" The answer format is three
lowercase words joined by hyphens, then a hyphen and a two-digit number (for example
`jovial-island-copper-93`), and the value is trimmed before grading.

- **Time Trial:** one `blob.txt`, one answer field (`message`). The start response carried a
  20 second deadline (`2026-10-21T05:07:46Z` to `05:08:06Z`) even though the attempt was started as
  untimed, and `hall_of_fame_eligible` was true.
- **Mastery:** one ZIP containing `blob-01.txt` through `blob-13.txt`, 13 answer fields
  (`set1.message` to `set13.message`), and a 30 second window.

## Solution

1. **Time Trial.** The blob is hex. `bytes.fromhex(...)` followed by UTF-8 decoding gives
   multi-line text. The final non-empty line is a sentence ending in the slug. We took the last
   whitespace-delimited token of that line.
2. **Mastery layout.** Each line of a `blob-NN.txt` is `NN<TAB>chunk`. The chunks are fragments
   of one message (split mid-word), each encoded separately, and the leading number gives the
   order. Decode every chunk, sort by the number, concatenate, and read the last non-empty line.
3. **Codec choice per chunk.** At this day the numbering used only three codecs: base64, ROT13
   and hex. The extractor guessed the codec from the chunk's shape instead of using the record
   number:

   ```python
   def dec(c):
       if re.fullmatch(r'(?:[0-9a-fA-F]{2})+', c):          # hex, even length
           try: return bytes.fromhex(c).decode('utf8')
           except Exception: pass
       if re.fullmatch(r'[A-Za-z0-9+/]+={0,2}', c) and len(c) % 4 == 0:   # base64
           try:
               s = base64.b64decode(c, validate=True).decode('utf8')
               if s.isprintable(): return s
           except Exception: pass
       return codecs.decode(c, 'rot13')                      # fallback
   ```

   The order matters: a chunk that is valid hex is tried first, base64 second (with a
   printable-text check, so a hex-looking or random base64 string is not accepted by accident),
   and ROT13 last because it succeeds on any string.
4. **Submission.** The mapper prints `{"set1.message": "<slug>", ...}` with every value a string.
   We sent all 13 in one call and got CORRECT. The Time Trial sent `{"message": "<slug>"}` in
   one call.

## Time Trial and Mastery

The only real change from a single blob is scale and the deadline. The 20 and 30 second windows
were issued by the server regardless of the untimed default, so the work was scripted end to end
(start, download, check the SHA-256 against the start response, map, submit). The decode itself
took about 10 ms. In the 2026-10-26 ranked reruns of the same day the whole pass took 1.99 s
(Time Trial) and 4.63 s (Mastery), dominated by issue, download and submit round trips.

## Dead ends and gotchas

- The 20 to 30 second windows on a nominally untimed `start` were the only trap we recorded:
  there is no time to explore a blob by hand, so the mapper has to exist before the attempt is
  issued.
- Guessing the codec per chunk worked here because only three codecs were in play. The journals
  do not record any mis-decoded chunk, but the approach would not hold up once later days added
  more codecs (the later generalized tool keys the codec on the record number instead).
- The answer is the last token of the last non-empty line, not the whole sentence. The journals
  record this as the rule used and do not mention a rejected alternative.

## Takeaways

- For a "decode this blob" warmup, the title and prompt name the layer ("Hex Appeal", "sixteen
  fingers"), so start with the obvious codec and look at the final line.
- For numbered-record Mastery bundles, sort by the record number before decoding, and concatenate
  after decoding, because the split can fall inside a word.
- Keep a mapper ready and make start-to-submit a single scripted pass when the window is under a
  minute. The reusable form of this is the numbered codec-chain decoder (WARMUP-CODEC-CHAIN-AUTO-001),
  which this day's Time Trial and Mastery both reproduce slug for slug.

## Evidence

Leaf base path: `challenges/day03/warmups/warmups-day-03/`

- Single Solve: no `single-solve/` directory exists. The only record is the `progress.md` row
  (basis "Player-directed local mark"). Nothing in the leaf shows the solve or its result.
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/artifacts/answers.json`,
  `time-trial/start_response.json`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/artifacts/answers.json`,
  `mastery/start_response.json`, `mastery/ranked-1/result.json`
- Both modes' local copies of the artifacts matched the SHA-256 in the start responses.
- The leaf holds no `result.json` at the top level of either mode, only the `ranked-1/` ones. The
  2026-10-20 CORRECT results are recorded in the journals and the `progress.md` rows (1 submission
  each, within the server window).
