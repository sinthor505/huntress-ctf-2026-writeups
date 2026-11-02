# Off the Rails

| | |
|---|---|
| Day / Lane | 18 / Warmups |
| Challenge ID | `warmups-day-18` |
| Technique | 3-rail rail-fence cipher; numbered multi-codec record chain (Morse-of-hex, a/b hex nibbles, rail fence) |
| Single Solve | Independent (journal-recorded CORRECT, 1 call; no result file saved) |
| Time Trial | Independent (journal-recorded CORRECT, 1 call; no result file saved) |
| Mastery | Independent (journal-recorded CORRECT, 1 call; no result file saved) |

## Summary

The prompt, "Three rails. One very zigzag commute.", names the cipher outright: a three-rail rail
fence. The Single Solve and Time Trial blobs are one short block of transposed English whose last
line carries the answer slug. Mastery ships 88 blobs, each an 18-record chain, where the last
record is the same rail fence and two new records (Morse-encoded hex, and a/b groups that are hex
nibbles) extend the previous warmup's 15-record codec. All three modes were CORRECT on the first
submission.

## The challenge

Each mode delivered a static blob, checked against the `sha256` in the start response:

- Single Solve: `warmups-day-18.bin`, 109 bytes, one answer field (`message`).
- Time Trial: a fresh 110-byte blob, one answer field. The start response carried a 30 s deadline
  even though we had not asked for a timed run.
- Mastery: a 50,857-byte ZIP of 88 blobs (`blob-01.txt` ... `blob-88.txt`), 88 answer fields
  (`set1.message` ... `set88.message`), with a deadline of about 105 s. It was locked
  (409 `prerequisite_required`) until the Time Trial was CORRECT.

Every question asked for the same thing: decode the blob completely and submit the slug its final
non-empty decoded line instructs, in the format "three lowercase words separated by hyphens,
followed by a hyphen and a two-digit number", normalization `trim`.

## Solution

1. **Recognize the cipher.** The Single Solve blob read as scrambled English with ordinary letter
   frequencies, which points to a transposition rather than a substitution. With "three rails" in
   the prompt, the candidate was a standard zigzag rail fence starting on rail 0 going down.
2. **Decode.** Compute each position's rail, sort positions by `(rail, index)`, and hand the
   ciphertext characters out to the sorted positions in order:

   ```python
   def dec(s, n):
       L = len(s); pat = []; r = 0; d = 1
       for i in range(L):
           pat.append(r)
           if r == 0: d = 1
           elif r == n - 1: d = -1
           r += d
       idx = sorted(range(L), key=lambda i: (pat[i], i))
       out = [None] * L
       for k, i in enumerate(idx): out[i] = s[k]
       return ''.join(out)
   ```

3. **Take the slug.** The decoded text ends with an instruction of the form "submit this exact
   answer to the Game Master: <slug>". The answer is the last token of the last non-empty line.
   Single Solve: `dapper-summit-brook-29`.
4. **Time Trial.** The same mapper, unchanged, run on the fresh blob and submitted straight away
   inside the 30 s window.

## Time Trial and Mastery

Time Trial was identical to Single Solve apart from the blob and the clock.

Mastery was the real change. Each `blob-NN.txt` held 18 tab-separated numbered records
(`NN<TAB>payload`), and the record number fixes the codec. The previous day's warmup used 15 record
types (base64, ROT13, hex, binary, decimal, octal, percent-encoding, HTML entities, JSON escapes,
base32, base64, reverse, Atbash, Caesar-7, ROT47); the mapper imported that day's `decode_record`
unchanged for records 1-15 and added three:

- **16:** Morse code (space-separated) whose symbols are hexadecimal digits; unhexlify to ASCII.
- **17:** space-separated groups of five `a`/`b` letters (a=0, b=1). Each group is a 4-bit value,
  a hex nibble (0-15), not a Baconian letter. Nibble pairs become bytes.
- **18:** the 3-rail rail fence from the Single Solve.

Concatenating the decoded fragments in record order gave text ending "...Game Master: <slug>". The
mapper asserted three things before printing anything: the ZIP SHA-256, that the ZIP's member set
matched the question `artifact` fields, and that every slug matched
`[a-z]+-[a-z]+-[a-z]+-\d{2}`. It then checked that the answer-ID set equalled the start response's
`answer_ids`. All 88 answers went in as one submission and came back CORRECT inside the deadline.

## Dead ends and gotchas

- **Record 17 is not Bacon.** Five-letter `a`/`b` groups look like Bacon's cipher, but decoding
  them as letters gives gibberish. Reading each group as a 5-bit integer gave only values 0-15, so
  they are nibbles. (The previous day's Single Solve used the same trick, so this was a known
  pattern by then.)
- **Timed windows appeared without `run_for_time`.** Time Trial came back with a 30 s deadline and
  Mastery with about 105 s on a plain `start`, so the mapper has to exist before the request.
- **The record count changes by day.** The mapper asserts 18 lines per blob rather than assuming it.
- **No answer-format surprises.** Only the slug was submitted, not the instruction sentence.

## Takeaways

- **Rail-fence (zigzag) transposition:** scrambled text that still looks like English, plus a prompt
  that mentions rails or zigzag, is a rail fence. If the rail count is not stated, try 2-8 and score
  by English text; the library's `codec_chain.py` does exactly that.
- **Numbered-record Mastery warmups** are additive: each day reused the previous day's record
  table and added a few codecs, so import the old decoder rather than rewrite it, and assert the
  record count and slug regex so a drifted codec fails offline instead of at the validator.
- Check `a`/`b` groups for a 16-value alphabet before assuming Bacon.

## Evidence

Leaf base path: `challenges/day18/warmups/warmups-day-18/`. No `result.json` was saved for any mode
(and no `ranked-N/` reruns exist); the CORRECT results are recorded in the journals, in
`challenges/day18/submission-attempts.md` (1 attempt per mode) and in the `progress.md` rows.

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/candidate.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py` (18-record mapper), `mastery/artifacts/candidate.json`
- `library/warmups/rail-fence-numbered-codec.md` (WARMUP-RAILFENCE-D18-001)
- `progress.md` (grep `^| 18 .*warmups-day-18`)
