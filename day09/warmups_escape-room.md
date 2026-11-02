# Escape Room

| | |
|---|---|
| Day / Lane | 9 / Warmups |
| Challenge ID | `warmups-day-09` |
| Technique | JSON string-literal decoding; numbered multi-codec fragments (Mastery) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (journal for the original attempt; later ranked reruns have `result.json` CORRECT) |
| Mastery | Independent (`submission-response.json` CORRECT; later ranked and untimed reruns have `result.json` CORRECT) |

## Summary

Escape Room is a warmup whose prompt ("These characters would very much like to escape") is a pun:
the artifact is text written as one JSON string literal full of `\uXXXX` escapes. Parsing it with a
real JSON parser yields the instruction and the answer slug. Mastery scaled this to 43 blobs, each
built from nine fragments in nine different encodings, and all three modes went CORRECT on the first
submission.

## The challenge

Single Solve and Time Trial each delivered one `blob.txt` and one question (`message`): decode the
blob completely and submit the slug that the final non-empty decoded line tells you to submit. The
answer format was three lowercase words, a hyphen and a two-digit number (`word-word-word-NN`),
normalized by `trim`. Single Solve was untimed. The Time Trial window was 20 s, and its question
embedded the artifact SHA-256 (`[assigned-artifact-sha256:...]`), which let us confirm the saved
bytes were the assigned ones.

Mastery delivered a ZIP (24,249 bytes) containing `blob-01.txt` to `blob-43.txt` and 43 answer IDs,
`set1.message` to `set43.message`, each pointing at its own member.

## Solution

1. Recon: the SHA-256 of the downloaded bytes matched the start response, so the file was the
   assigned artifact and nothing had been mangled in transit.
2. The first bytes were a JSON opening quote followed by `\u00XX` escapes. No labeled codec was
   present: no Base64 alphabet, no percent escapes, no numeric byte list, no HTML references. The
   only layer was the JSON string escaping itself.
3. Parsing the whole file with `json.loads` returned one top-level string, which decoded to a
   single non-empty line. That line states the slug to submit. The decoder is a few lines of standard
   library code that also rejects a top-level value that is not a string:

   ```python
   decoded = json.loads(path.read_bytes())
   if not isinstance(decoded, str):
       parser.error("top-level JSON value is not a string")
   print(decoded)
   ```

4. Single Solve answer: `happy-piano-whale-26`, submitted under `message` in one call. The Time Trial
   used a distinct variant (different SHA-256), decoded the same way, and gave
   `quick-piano-butter-96`, also one call, accepted 4.553 s after start inside the 20 s window.

## Time Trial and Mastery

The Time Trial changed only the bytes. Mastery changed the shape. Each member was nine
tab-separated lines, `NN<TAB>payload`, and the line number fixed the codec:

| Line | Encoding |
|---|---|
| 01 | Base64 |
| 02 | ROT13 |
| 03 | hexadecimal UTF-8 |
| 04 | eight-bit binary groups |
| 05 | decimal byte values |
| 06 | octal byte values |
| 07 | URL percent encoding |
| 08 | numeric HTML character references |
| 09 | a JSON string literal (the Single Solve codec) |

These are the encodings of Days 1 to 9 in order. Each line decodes to a short fragment, often cut
mid-word (`Nice work de`, `coding this w`, ...), and the nine fragments concatenated in order form
one sentence: "Nice work decoding this warmup! To solve, submit this exact answer to the Game Master:
<slug>". The extractor strictly validated each representation, required exactly nine sequentially
numbered lines per member, joined the fragments, took the slug from the final line, and checked that
the mapped ID set equalled the start response's `answer_ids` exactly before anything was submitted.
All 43 answers went out in one call and were CORRECT.

Later ranked reruns used a 20 s Time Trial window and a 60 s Mastery window, and the saved per-attempt
results show them going CORRECT in about 1.4 to 4.7 s end to end. By then the generic warmup solver
(the codec-chain decoder with a per-record codec table) handled both modes; the Day 9 specifics did not
need changing.

## Dead ends and gotchas

- The question says "decode completely" and "final non-empty line"; here that is almost a formality.
  The instruction template is the same every time and the slug is the text after the last `:`.
- The decoded text of a single Mastery fragment is not an answer. The slug is split across fragments
  (in the first member the slug starts with the `d` that ends the seventh fragment, `me Master: d`, and
  continues in `ashing-garden` and `-waterfall-18`), so the fragments must be joined before the slug is
  extracted.
- Each Mastery member decodes only with the codec its line number names. Applying a generic
  auto-detector per line risks misreading the short fragments, so the extractor dispatched by line
  number instead.
- Time Trial and Mastery artifacts are different variants from Single Solve. Answers from one mode do
  not carry over.
- We did not hit a rejected answer in any mode.

## Takeaways

- When a warmup's decoded content is "just" a string literal, use the language's own parser instead of
  stripping escapes by hand; it validates the syntax and handles the escapes consistently.
  Technique name: JSON string decoding.
- For numbered multi-codec Mastery bundles, dispatch on the record number, join fragments in order, and
  assert that the mapped IDs equal the start response's `answer_ids` before submitting.
- A scripted decode-and-submit pass was the right shape for the short timed windows.

## Evidence

Leaf base path: `challenges/day09/warmups/warmups-day-09/`

- `single-solve/journal.md`, `single-solve/artifacts/decode_json_string.py`. There is no saved result
  file for Single Solve; the journal records one submission and a `CORRECT` result, so it is
  journal-recorded rather than file-backed.
- `time-trial/journal.md` (original attempt: one submission, `CORRECT`),
  `time-trial/ranked-1/`, `ranked-2/`, `ranked-4/` and `time-trial/open-1/` (`result.json` CORRECT,
  one call each). `ranked-3/NOTE.md` says that attempt's result files were overwritten by mistake,
  and the note records it as CORRECT from a batch log.
- `mastery/journal.md`, `mastery/artifacts/extract_mixed_lines.py`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/candidate-answers.json`, `mastery/submission-response.json` (CORRECT, 43 fields,
  one call), `mastery/ranked-1/`, `ranked-2/`, `ranked-3/` and `mastery/open-1/` (`result.json`
  CORRECT).
- Per-day solver used for the reruns: `tools/ranked/solvers/warmups/solve.py`.
- Library entries: `WARMUP-JSON-STRING-001`, `WARMUP-TEXT-CODECS-001`, `WARMUP-NUMERIC-TEXT-001`,
  `WARMUP-PERCENT-DECODING-001`, `WARMUP-HTML-ENTITIES-001`.
- `progress.md` rows for Day 9: three `Solved` rows dated 2026-10-20.
