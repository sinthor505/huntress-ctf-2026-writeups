# Eight Is Enough

| | |
|---|---|
| Day / Lane | 6 / Warmups |
| Challenge ID | `warmups-day-06` |
| Technique | Octal-ASCII decoding; numbered multi-codec blobs (Base64, ROT13, hex, binary, decimal, octal) |
| Single Solve | Solved (journal-recorded): the journal records CORRECT on submission 1; no response or result file was saved |
| Time Trial | Independent (1 submission; plus one later ranked rerun) |
| Mastery | Independent (1 submission; plus three later ranked reruns) |

## Summary

A warmup whose theme ("Someone counted on eight fingers. Again.") is a hint at base 8. The Single
Solve and Time Trial blobs are whitespace-separated three-digit octal byte values; the Mastery blobs
mix six encodings and are split across numbered rows. Every mode was solved from the real bytes with
a few lines of Python and went CORRECT on the first submission.

## The challenge

One question per blob (`message`): decode the assigned blob completely and submit the exact slug that
its final non-empty decoded line instructs. The answer format is three lowercase words joined by
hyphens, then a hyphen and two digits (for example `word-word-word-NN`); normalization is trim.

- **Single Solve:** one `blob.txt`, untimed.
- **Time Trial:** one `blob.txt`, server-enforced 20 s window.
- **Mastery:** a ZIP of 28 blobs (`blob-01.txt` ... `blob-28.txt`), 28 answer fields
  (`set1.message` ... `set28.message`), 45 s window.

## Solution

1. Recorded the SHA-256 of the downloaded artifact and checked it against the start response.
2. Looked at the bytes. The blob was repeated three-digit tokens such as `116 151 143 ...`. Every
   digit was 0-7, which together with the day's title points at octal rather than decimal. Read as
   decimal, three-digit tokens above 127 are not ASCII and the result is junk; as octal they are
   printable bytes.
3. Parsed each token in base 8 and decoded the bytes as ASCII:

   ```python
   from pathlib import Path
   data = Path('blob.txt').read_bytes()
   print(bytes(int(x, 8) for x in data.split()).decode('ascii'))
   ```

4. The output was a fixed warmup sentence ending in the instruction: `Nice work decoding this
   warmup! To solve, submit this exact answer to the Game Master: hearty-raven-clover-39`. The slug
   is the text after the last `:` of the last non-empty line, and it matched the required pattern.
5. Submitted `hearty-raven-clover-39` as `message`; the result was `CORRECT`.

## Time Trial and Mastery

**Time Trial** was the same octal encoding with a fresh blob (new SHA-256, new slug). The only
difference was the 20 s window, so the decode ran straight after the download and the submission was
accepted before the deadline.

**Mastery** was where the day's real content was. Each of the 28 blobs interleaved six encoding
layers in numbered rows: Base64, ROT13, hexadecimal, binary bytes, decimal ASCII and octal ASCII.
The row number selects the codec (Day 6 has six rows; the library's numbered-codec notes list the
same first six codecs for later days). For each blob we decoded every row separately, concatenated
the six decoded fragments in row order, and took the slug from the last line of the resulting
sentence. All 28 answers were submitted in one request inside the 45 s window and accepted together.

The original solves used a local script that was not kept in the leaf. For the later ranked reruns
(Time Trial once, Mastery three times) we used the generic warmup solver, which drives the library
`codec_chain` tool over the artifact (a raw blob, or the ZIP of blobs) and takes the literal text
after the last `:`. Each rerun was a fresh attempt with new blobs, went CORRECT on the first call,
and took roughly 2.3 s (Time Trial) or 4.7-5.0 s (Mastery) end to end, dominated by issue, download
and submit latency rather than decoding.

## Dead ends and gotchas

- Octal versus decimal is the one trap in the Single Solve and Time Trial: three-digit tokens using
  only 0-7 are octal. Decimal parsing yields non-printable bytes, which is the tell.
- The answer is the slug alone, not the whole final sentence.
- Time Trial and Mastery blobs differ per attempt (different hashes and slugs), so an answer from one
  attempt does not carry over; each had to be decoded from its own download.
- Time Trial and Mastery submissions use the mode-suffixed IDs (`warmups-day-06/time_trial`,
  `warmups-day-06/mastery`), not the bare challenge ID.

## Takeaways

- For a warmup, identify the encoding from the byte alphabet first (digits 0-7 only means octal),
  then let the fixed template sentence confirm the decode: a correct decode produces the "Nice work
  decoding this warmup!" line.
- Numbered-record Mastery blobs are a codec per row number; build or reuse a table keyed by row
  number rather than guessing per blob. Search for "numbered codec chain" and "octal ASCII".
- With 20-45 s windows, script download, decode and submit as one pass.

## Evidence

Base path: `challenges/day06/warmups/warmups-day-06/`

- `single-solve/journal.md`, `single-solve/blob.txt`, `single-solve/start-response.json`
- `time-trial/journal.md`, `time-trial/decoded.txt`, `time-trial/submission-response.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/decoded-all.txt`, `mastery/submission.json`,
  `mastery/submission-response.json`, `mastery/ranked-1/`, `ranked-2/`, `ranked-3/` (`result.json`)
- Generic solver for the reruns: `tools/ranked/solvers/warmups/solve.py`; library entries for
  numeric text and the numbered codec chain.
- `progress.md`: Day 6 rows for `warmups-day-06` (Single Solve, Time Trial, Mastery).
