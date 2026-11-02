# By the Numbers

| | |
|---|---|
| Day / Lane | 5 / Warmups |
| Challenge ID | `warmups-day-05` |
| Technique | Decimal ASCII decode; Mastery chains the Day 1-5 codecs across numbered records |
| Single Solve | Solved, graded result not preserved (`progress.md` row only; the journal records the candidate but no submission, and no result file or ledger entry exists) |
| Time Trial | Independent |
| Mastery | Independent (attempt 1 expired unscored, attempt 2 CORRECT) |

## Summary

A warmup whose only job is to turn a list of decimal numbers back into text. Single Solve and Time
Trial were one decimal-byte blob each. Mastery was 23 deliveries of a five-record chain, and its
real difficulty was a 40 s clock, not the codec. The first Mastery attempt expired while we were
still fetching files by hand. A scripted retry, fetch, decode and submit pass went CORRECT.

## The challenge

The prompt is one line: "These characters have been reduced to numbers." Single Solve delivered
`blob.txt`, ASCII text of space-separated decimal values. The single question (`message`) asked for
the slug that the final non-empty decoded line tells you to submit. The answer format was a pattern:
three lowercase words joined by hyphens, then a hyphen and a two-digit number, normalized with `trim`.

Time Trial had the same shape with a fresh blob. Mastery had 23 questions (`set1.message` ..
`set23.message`) and 23 downloads, every one named `blob.txt` and referenced by the question as
`delivery N: blob.txt`.

## Solution

1. Checked the downloaded blob's SHA-256 against the start response. It was ASCII text, a run of
   integers all within 0-255.
2. Read each token as a decimal byte and decoded the byte string as UTF-8:

   ```python
   values = [int(t, 10) for t in data.split()]   # each must be 0..255
   text = bytes(values).decode("utf-8")
   ```

   The leaf's `decode_numbers.py` also rejects any non-numeric or out-of-range token, so a wrong
   guess at the encoding would fail loudly. For example, octal-looking input would trip the range
   check.
3. The text was a fixed template: `Nice work decoding this warmup! To solve, submit this exact answer
   to the Game Master: sturdy-river-grove-57`. The answer is the slug on that last line, submitted
   as `{"message": "sturdy-river-grove-57"}`.

## Time Trial and Mastery

**Time Trial.** Same codec, fresh blob, one answer ID. The server assigned a 20 s window even
though we had not asked for a timed run, and `hall_of_fame_eligible` was `true`. We decoded the blob
with the shared codec-chain tool (a single `decimal` layer), mapped the slug onto `message`, and got
CORRECT on the first submission.

**Mastery.** Two things were new.

- *Delivery shape.* Later warmup Mastery days ship one ZIP of `blob-NN.txt` members. Day 5 shipped
  23 standalone files in the start response's `artifacts[]`, all named `blob.txt`, so the file name
  cannot be used to tell them apart. The only link between question and file is the delivery index.
  The Game Master client's `artifact` command fetches only the first one, so the rest had to be
  fetched from their individual capability links, each SHA-256-checked.
- *Record chains.* The first journal entry called these "single decimal blobs". That was wrong. Each
  delivery was five numbered records (`NN<TAB>payload`), and the record number picks the codec.
  Records 1-5 were base64, rot13, hex, 8-bit binary and decimal, which are the codecs of Days 1-5.
  Each record carried a slice of the same sentence. Concatenating the five decoded slices gave the
  usual template, and the slug fell at the end, inside the decimal record. The codec-chain tool
  reported `records=5` for every file.

  ```
  01  TmljZSB3b3JrIGRlY29kaW5nIHRo        -> "Nice work decoding th"
  02  vf jnezhc! Gb fbyir, f              -> "is warmup! To solve, s"
  03  7562...                             -> "ubmit this exact answe"
  04  01110010 00100000 ...               -> "r to the Game Master: "
  05  106 111 118 105 ...                 -> "jovial-basket-maple-45"
  ```

  This is delivery 1 from the retry attempt. The slice boundaries are our reading of the decoded
  output, and the slugs equal the submitted `candidate.json`.

All 23 slugs matched the standard pattern and full template with no repairs. The whole Mastery map
went in as one 23-field submission.

The Mastery window was 40 s, and the first `start` issued a deadline of exactly that. Fetching 23
files and decoding them by hand ran past it, so the attempt went `expired` and the submission was
not graded. The retry also hit transient 503 `temporarily_unavailable` responses on retry and
download calls, so we retried with short backoff until a fresh attempt stayed open. The attempt
that succeeded was one scripted pass: retry, fetch all `artifacts[]`, decode, submit.

Later ranked reruns of both modes were CORRECT on the first call, with one submission each. The
Time Trial took 1.48 s end to end (20 s window), and Mastery took 4.35 s (40 s window).

## Dead ends and gotchas

- **"Untimed" start was not untimed.** Time Trial and Mastery both returned short server-enforced
  deadlines (20 s and 40 s) without a timed-run request. Anything interactive burns the attempt.
  An `expired` submission is not a wrong answer, but it counts against the attempt budget.
- **Mastery file names are useless.** All 23 deliveries are called `blob.txt`. Map by position in
  `artifacts[]` (`delivery N` is `setN`), not by name or sorted order.
- **The single-codec assumption breaks on Mastery.** A decoder that handles only decimal would work
  for Single Solve and Time Trial and fail on every Mastery delivery. Mastery needs the per-record
  codec table.
- **Decimal and octal layers look alike** (digits and spaces). The library's layer cheat sheet lists
  a decimal-versus-octal tell; we did not need it here because the 0-255 parse produced valid UTF-8
  with the expected template.

## Takeaways

- A "numbers" warmup is a decimal-bytes decode: `bytes(int(t) for t in blob.split()).decode()`.
- In this lane the same answer is always the last-line slug of one fixed template. Mastery just
  splits that template over numbered records and uses the earlier days' codecs.
- When a mode might be timed, script the whole pass (issue, download every artifact, decode, submit)
  and verify every download against its SHA-256. Do not fetch by hand.
- Search terms: decimal ASCII decode, numbered-record codec chain, multi-delivery Mastery.

## Evidence

Leaf base path: `challenges/day05/warmups/warmups-day-05/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/artifacts/decode_numbers.py`,
  `single-solve/submission.json`. There is no saved graded result file for the Single Solve. The
  `progress.md` row records it as Solved (2026-10-18), so provenance here rests on that row.
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/start_response.json`,
  `time-trial/ranked-1/result.json` (CORRECT, `t_total` 1.48 s).
- `mastery/journal.md` (including its "Deep review" correction), `mastery/start_response.json`,
  `mastery/retry_response.json`, `mastery/artifacts/candidate.json`,
  `mastery/artifacts_raw/set1.txt` .. `set23.txt`, `mastery/ranked-1/result.json` (CORRECT,
  `t_total` 4.35 s).
- Library entries: WARMUP-NUMERIC-TEXT-001, WARMUP-CODEC-CHAIN-AUTO-001 and WARMUP-PLAYBOOK-001
  (`library/warmups/`).
