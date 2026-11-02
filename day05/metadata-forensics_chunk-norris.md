# Chunk Norris

| | |
|---|---|
| Day / Lane | 5 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-05-chunk-norris` |
| Technique | PNG chunk walk, IHDR dimensions (plus the cumulative note/JPEG field family in Time Trial and Mastery) |
| Single Solve | Solved, graded result not preserved (the leaf has the submitted payload and a `progress.md` row, no `result.json`) |
| Time Trial | Independent (CORRECT on the second attempt; a ranked rerun on 2026-10-26 was also CORRECT) |
| Mastery | Independent (CORRECT on the first submission; a ranked rerun on 2026-10-26 was also CORRECT) |

## Summary

The prompt promised a hunt for a PNG hiding "an unexpected structural passenger", but the Single Solve asked for exactly one
literal value: the width and height stored in `panel-001.png`'s IHDR chunk. We read it with a CRC-checked chunk walk and
submitted it. Time Trial and Mastery were the same PNG question plus eight note/JPEG fields, both scripted because of 50 s
and 80 s windows.

## The challenge

Prompt, identical in all three modes: *"The PNGs render, but rendering is not proof. Walk their chunks, verify their declared
geometry, and locate the image with an unexpected structural passenger."*

- **Single Solve:** untimed, one ZIP (`metadata-day-05-evidence.zip`) holding one 414-byte file, `panel-001.png`. One answer
  field, `d05-png-001-png-ihdr-dimensions-pixels`, free-text, trimmed.
- **Time Trial:** a ZIP with `evidence-note.txt`, `camera-001.jpg` and `panel-001.png`; 9 fields; a real deadline of about 50 s
  even though we had not asked for a timed run.
- **Mastery:** an outer ZIP of two inner evidence ZIPs (`metadata-day-05-evidence-01.zip`, `-02.zip`); 18 fields
  (`set1.*`, `set2.*`); an 80 s window.

## Solution

1. Hash the ZIP and compare it with the start response. List the member: one PNG.
2. Walk the chunks by hand rather than trusting a renderer. The inspector we wrote (`inspect_png_chunks.py`) reads the 8-byte
   signature, then loops over `length | type | data | CRC`, recomputing `zlib.crc32(type + data)` for each chunk. It also flags
   non-consecutive IDATs, critical chunks after the IDAT run, and any bytes after IEND.
3. Result for `panel-001.png`: `IHDR` (13 bytes), one `IDAT` (357 bytes), `IEND`, all three CRCs valid, nothing after IEND. There
   was no passenger in this file.
4. IHDR holds width and height as big-endian 32-bit integers: `0000010d` and `00000099`, that is 269 and 153. The IHDR CRC
   matched (`baf0f8be`), so the geometry is the declared geometry.
5. Submitted `269x153` (width `x` height, lower-case `x`). Answer value for the record: `269x153`.

The IHDR read is a handful of bytes, and the decisive command was just `struct.unpack(">IIBBBBB", ihdr)` after the CRC check.

## Time Trial and Mastery

Both modes asked for the cumulative Days 1-10 template, not just the PNG field:

- `evidence-note-filename`, `-size-bytes`, `-md5`, `-sha1`, `-sha256`, `-magic-prefix-hex-16`, `-text-utf8-bom-present`
- `d04-jpeg-001-jpeg-sof-dimensions-pixels` (width x height from the first JPEG SOF marker)
- `d05-png-001-png-ihdr-dimensions-pixels`

The mapper (`answer_map.py`) is a single function that opens the evidence ZIP, hashes the note, skips JPEG markers until a
SOF0-SOF15 (excluding DHT, JPG and DAC) to read the dimensions, and reads the PNG width/height at offset 16. Integers and the BOM
boolean are converted to JSON strings before submitting, because the validator only accepts string values. Mastery generalised it
over *N* inner ZIPs: sort the outer members, take the trailing number from each inner filename, prefix the IDs `setK.`, and assert
that the produced key set equals the `answer_ids` of the live start response before printing.

The windows shaped the work. The first Time Trial `start` expired (about 50 s) before the mapper, adapted from an earlier day's
Single Solve, had been run; the submission on the expired attempt returned `expired` and graded nothing. A `retry` issued a fresh
attempt and fixtures, and the mapper went CORRECT in one call (2 submissions in total). Mastery's first attempt was lost to
503/500 responses from the Game Master on every artifact fetch for about a minute; after it recovered, a `retry` plus the mapper went
CORRECT in one call. Both modes were later re-run as ranked passes on 2026-10-26 (solve time 0.0 s, whole pass 1.4 s for Time
Trial and 1.8 s for Mastery, one call each).

## Dead ends and gotchas

- **Flavour text vs the asked field.** "Unexpected structural passenger" suggested a trailing chunk, a late IDAT or data after
  IEND. The question was only the IHDR dimensions, and the one PNG delivered was structurally clean. The chunk walk was cheap
  insurance, not the answer.
- **String-only answers.** Time Trial and Mastery values such as the note size and the BOM flag had to be sent as strings
  (`"123"`, `"false"`); native JSON types return `invalid_answers`.
- **Short real deadlines without asking.** Time Trial and Mastery returned deadlines (50 s and 80 s) on a plain `start`, so the
  mapper had to be written and tested before the first `start`.
- **Saved artifacts were damaged locally** by line-ending translation (LF to CRLF). The PNG signature itself contains a genuine
  `\r\n`, so a blanket undo corrupts it; a selective repair verified against the start response's SHA-256 is needed. This
  affected only our stored copies, not the live downloads.

## Takeaways

- Technique to search for: **PNG chunk walk with per-chunk CRC** (`IHDR -> IDAT -> IEND`), reading dimensions as big-endian
  `>II` at offset 16.
- Answer the literal field in the question; treat thematic prompt text as a hint to look, not as the target.
- For Days 1-10 metadata challenges the answer fields stack up across days, so a pre-built mapper for the note, JPEG and PNG family
  covers Time Trial and Mastery. Test it on a saved fixture before the clock starts.

## Evidence

Leaf base path: `challenges/day05/metadata-forensics/metadata-forensics-day-05-chunk-norris/`

- `single-solve/journal.md`, `single-solve/artifacts/inspect_png_chunks.py`, `single-solve/submission.json`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json`
- Library entries: PNG structure inspector (META-PNG-STRUCTURE-001), metadata Days 1-10 gotchas (META-GOTCHAS-A105)
