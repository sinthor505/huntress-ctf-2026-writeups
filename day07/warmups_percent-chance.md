# Percent Chance

| | |
|---|---|
| Day / Lane | 7 / Warmups |
| Challenge ID | `warmups-day-07` |
| Technique | Layered text encoding: URL percent-decoding to a fixed point; per-line codecs in the Mastery bundle |
| Single Solve | Solved (journal-recorded): the journal records our one submission as CORRECT; the leaf holds no result file |
| Time Trial | Independent (journal records CORRECT; ranked re-run `ranked-1/result.json` CORRECT) |
| Mastery | Independent (retry attempt CORRECT per journal and `retry-submission-response.json`; ranked re-run `ranked-1/result.json` CORRECT) |

## Summary

A warmup whose whole prompt is "My address bar is having a moment." The artifact is a text blob in which
every character, letters included, is written as a `%HH` escape. Percent-decoding it (repeatedly, until the
text stops changing) yields a sentence that ends with the slug to submit. Single Solve and Time Trial were
one-liners; Mastery scaled it to a ZIP of 33 blobs, each built from seven different codecs.

## The challenge

- **Prompt:** "My address bar is having a moment." The single question (`message`) asks: decode the assigned
  blob completely and report the slug that the final non-empty decoded line tells you to submit.
- **Answer format:** three lowercase words joined by hyphens, then a hyphen and a two-digit number, for
  example `witty-storm-fern-77`. Submit the slug only; normalization is `trim`.
- **Single Solve:** one `blob.txt`, one answer.
- **Time Trial:** same shape, one `blob.txt`, one answer, with a 20 second server deadline.
- **Mastery:** a ZIP of 33 members (`blob-01.txt` to `blob-33.txt`), 33 answers (`set1.message` to
  `set33.message`), 50 second deadline.

## Solution

1. **Recon.** The file is plain ASCII, and the start response's SHA-256 matched the download. Every
   character is a `%HH` triple, including letters and spaces, so the text starts `%4e%69%63%65%20...`.
   There is no binary content and nothing to execute.
2. **Decode to a fixed point.** The question says "decode completely", which suggests
   possible nested layers, so the extractor applies `urllib.parse.unquote` in a loop until a pass changes nothing.
   For this blob one pass was enough and the second pass was a no-op; the loop is just insurance against
   a double-encoded variant.

   ```python
   text = args.path.read_text()
   for layer in range(1, 11):
       decoded = unquote(text)
       print(f"Layer {layer}: {decoded}")
       if decoded == text:
           break
       text = decoded
   ```

3. **Read the last line.** The decoded text was: "Nice work decoding this warmup! To solve, submit this
   exact answer to the Game Master: witty-storm-fern-77". The answer is that trailing slug, submitted as
   `{"message":"witty-storm-fern-77"}`. It was submitted once and graded CORRECT per the journal.

## Time Trial and Mastery

**Time Trial.** Same blob style, fresh slug (`happy-starfish-quartz-38`). The window was 20 seconds, so the
solve was a script rather than a manual decode: decode to a fixed point (up to 100 passes), then pull the
slug from the last non-empty line with a regex anchored on "submit this exact answer to the Game Master:",
and print the answer JSON. One submission, CORRECT. A later ranked re-run of the same script took about
1.7 s end to end, nearly all of it issue, download and submit.

**Mastery.** This is where the challenge actually grew. The download is a ZIP of 33 text members. Each
member is seven tab-separated, numbered lines, and the line number fixes the codec of that line:

| Line | Encoding |
|---|---|
| 1 | Base64 |
| 2 | ROT13 |
| 3 | Hex bytes (UTF-8) |
| 4 | 8-bit binary groups |
| 5 | Decimal byte values |
| 6 | Octal byte values |
| 7 | URL percent-encoding (the Single Solve / Time Trial codec) |

Each line decodes to a fragment of text, and the decoded fragments are concatenated in order to form the
member's full message, whose final non-empty line contains the slug. The extractor validates each line as
it goes (for example, that binary lines are whole 8-bit groups and decimal values fit in a byte), which
makes a wrong codec guess fail loudly rather than yield garbage. A second small script maps each question's
`artifact` member to its slug and asserts that the resulting key set equals the start response's
`answer_ids`.

The 50 second window was tight for a human-in-the-loop attempt. The first attempt's candidate map was
complete, but the Game Master returned `expired` and graded nothing. A `retry` issued a fresh ZIP with
different slugs; running the same extractor against it and submitting all 33 answers in one call was
graded CORRECT. A later ranked re-run completed in about 2 s total.

## Dead ends and gotchas

- **The expired Mastery attempt.** The answers on the first attempt were never evaluated, so there is no
  evidence about them either way. The retry assigns a different ZIP (different SHA-256, different slugs),
  so the first attempt's map could not be reused; the fix was simply to rerun the extractor.
- **Fixed-point decoding vs. one pass.** Here a single `unquote` was enough, but a loop is the safer default.
  The library note warns against blindly decoding to a fixed point on real URLs, where an encoded
  delimiter can be meaningful. For a CTF blob whose prompt says "decode completely" it is the right move.
- **Numeric-text technique does not cover this.** An existing library entry for numeric character codes
  applies only to the binary and decimal Mastery lines, not to the Base64, ROT13, hex, octal or percent lines.
- The Time Trial and Mastery windows (20 s and 50 s) shaped the approach: both were scripted, not
  hand-decoded.

## Takeaways

- Technique name: **repeated percent-decoding (URL decoding to a fixed point)**, plus **per-line codec
  dispatch** for the Mastery bundle.
- Read the Mastery bundle's structure before writing code: the line numbers are the codec selectors, and
  the number of lines matches the release day (seven lines on Day 7).
- Validate each codec's input strictly. It costs almost nothing and tells you immediately if a line was
  decoded with the wrong scheme.
- Do the scripted version from the start for timed modes; the decode itself takes well under a second, and
  the clock is spent on issuing, downloading and submitting.

## Evidence

Leaf base path: `challenges/day07/warmups/warmups-day-07/`

- `single-solve/journal.md`, `single-solve/artifacts/decode_percent_layers.py`, `single-solve/start-response.json`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/extract_mixed_lines.py`, `mastery/artifacts/answer_map.py`,
  `mastery/retry-submission-response.json`, `mastery/ranked-1/result.json`
- `progress.md` rows for Day 7 `warmups-day-07` (all three marked Solved, 2026-10-20)
- Library entry: WARMUP-PERCENT-DECODING-001 (`library/warmups/percent-decoding/`)
