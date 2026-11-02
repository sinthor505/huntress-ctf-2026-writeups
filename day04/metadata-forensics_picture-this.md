# Picture This

| | |
|---|---|
| Day / Lane | 4 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-04-picture-this` |
| Technique | JPEG Start-of-Frame (SOF) marker parsing; evidence-note hashes in the ranked variants |
| Single Solve | Solved (journal-recorded): the journal records our own `CORRECT` response; no `result.json` |
| Time Trial | Independent (original attempt journaled `CORRECT`; ranked re-run has a saved `result.json` `CORRECT`) |
| Mastery | Independent (original attempt journaled `CORRECT`; ranked re-run has a saved `result.json` `CORRECT`) |

## Summary

The prompt promises a dozen resized, renamed and shuffled photographs and an image whose "presentation does not
match its structure". What the first round asked was much smaller: the pixel dimensions stored in one JPEG's SOF
segment. A short stdlib marker walker answered Single Solve, Time Trial and Mastery. A later ranked re-run showed
the same challenge asking eight fields per set instead of one.

## The challenge

Prompt: "A dozen photographs have been resized, renamed, and shuffled. Reconstruct their true dimensions and find
the image whose presentation does not match its structure."

| Mode | Delivered (original attempt) | Questions |
|---|---|---|
| Single Solve | `metadata-day-04-evidence.zip`, one member `camera-001.jpg` (3,025 bytes), untimed | 1: `jpeg.sof_dimensions_pixels` for `camera-001.jpg` |
| Time Trial | a ZIP holding `camera-002.jpg`, 50 s server window | 1: the same field for `camera-002.jpg` |
| Mastery | two nested ZIPs (`...-01.zip`, `...-02.zip`), 80 s window | 2: one per set, IDs prefixed `set1.` / `set2.`, members `camera-003.jpg` and `camera-004.jpg` |

The answer format is `exact_text` with `trim` normalization, and the field description says to give the
"exact artifact-derived value only". There is no example of the shape, so `WxH` (width first) is a convention we
inferred and the validator then accepted. Only one or two JPEGs were ever delivered per attempt; the "dozen" and
the "mismatched presentation" in the prompt were never tested by any question we saw.

## Solution

1. Hash the ZIP and compare it with the SHA-256 in the start response.
2. Read the member named in the question's `artifact` field (`container.zip -> camera-NNN.jpg`).
3. `file` reported `JPEG image data, baseline, precision 8, 257x146, components 3`. `exiftool` was not installed,
   and the specimen has no JFIF or EXIF segment (the stream starts with a quantisation table), so there is little
   for it to read anyway. The dimensions are in the SOF segment, so we parsed that directly.
4. Walk the marker stream: skip to `0xFF`, read the marker, skip standalone markers (`D0-D7`, `01`, `D8`), stop at
   `D9`, otherwise read the 2-byte segment length. At any SOF marker the payload is precision (1 byte), **height**
   (2 bytes), **width** (2 bytes):

   ```python
   SOF_MARKERS = {0xC0, 0xC1, 0xC2, 0xC3, 0xC5, 0xC6, 0xC7,
                  0xC9, 0xCA, 0xCB, 0xCD, 0xCE, 0xCF}   # not C4 (DHT), C8 (JPG), CC (DAC)
   ...
   if marker in SOF_MARKERS and segment_length >= 7:
       height = int.from_bytes(data[i + 3 : i + 5], "big")
       width  = int.from_bytes(data[i + 5 : i + 7], "big")
       return width, height
   ```
5. Format as `<width>x<height>`. Single Solve: SOF marker `0xC0`, 8-bit, 3 components, `camera-001.jpg` is
   `257x146`, matching `file`. Submitted under `d04-jpeg-001-jpeg-sof-dimensions-pixels`; the Game Master
   returned `CORRECT`.

## Time Trial and Mastery

The same solver was reused unchanged, and it also recursed into nested ZIPs, which was all the Mastery bundle
needed. Fixtures differed per mode, so each answer came from the bytes of that attempt:

| Attempt | Member | SOF result |
|---|---|---|
| Time Trial | `camera-002.jpg` | `274x157` |
| Mastery set 1 | `camera-003.jpg` | `291x168` |
| Mastery set 2 | `camera-004.jpg` | `308x179` |

Both ran inside a 50 s / 80 s window, so the solve was scripted: download, parse, submit all answer IDs from the
start response in one call. Each was accepted on the first submission.

A ranked re-run (a later audit pass) delivered a different shape: 8 fields in the Time Trial and 16 in Mastery
(8 per set). Alongside the JPEG field it asked the "evidence note" family: filename, size in bytes, MD5, SHA-1,
SHA-256, the first sixteen bytes as lowercase hex, and `text.utf8_bom_present`. The JPEG member in those
variants was `camera-001.jpg` in every set (`257x146`), and the note was a 33-byte `evidence-note.txt` with no
BOM. The reusable per-challenge solver reads the question IDs from the start response and answers whatever
family is asked; both ranked passes were `CORRECT` on one submission call (about 1.5 s end to end).

## Dead ends and gotchas

- SOF stores height before width, but the answer is width first. A reader that takes the two fields in file order
  would transpose every answer. The cross-check against `file`'s `WxH` line catches this.
- Saved copies of the three ZIPs (and the extracted `camera-001.jpg`, 3,070 bytes for a 3,025-byte member) were
  damaged by LF->CRLF translation during our own download. The damage is recoverable: undoing the added CRs
  reproduces the start-response SHA-256 exactly. This was an artifact of our transfer, not of the challenge.
- JPEG has several SOF variants (baseline, progressive, lossless, arithmetic). The marker set must exclude `C4`,
  `C8` and `CC`, which sit in the same range but are not frame headers. Every specimen here was baseline `C0`.
- The prompt's "dozen photographs" and "presentation does not match structure" suggest an EXIF-orientation or
  thumbnail-versus-frame mismatch, but these specimens carried no EXIF, and the question asked only for the SOF
  value. We did not find any mismatch to look for and did not investigate further.

## Takeaways

- Read the question's field name literally: `jpeg.sof_dimensions_pixels` named the exact structure to parse. A
  generic image library would answer the same here, but the SOF walk is shorter than installing anything and
  does not depend on a decoder accepting the file.
- When a challenge description is more elaborate than its question, solve the question and keep the solver
  general: the ranked re-run showed the same challenge had a larger field set that the same family-aware tooling
  covered.
- For timed modes on this day, resolve the member and answer IDs from the start response (fixture index
  changes per mode and per set) and script download-to-submit.
- Searchable terms: JPEG SOF0 marker, `jpeg.sof_dimensions_pixels`, height-before-width.

## Evidence

Leaf: `challenges/day04/metadata-forensics/metadata-forensics-day-04-picture-this/`

- `single-solve/journal.md`, `single-solve/artifacts/solve_jpeg_dimensions.py` (the solver used in all three modes)
- `time-trial/journal.md`, `time-trial/solver-output.json`, `time-trial/ranked-1/result.json`,
  `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/solver-output.json`, `mastery/ranked-1/result.json`,
  `mastery/ranked-1/accepted_plan.json`
- Per-challenge ranked solver: `tools/ranked/solvers/metadata-forensics-day-04-picture-this/solve.py`
- Library entries: JPEG SOF reader (META-JPEG-SOF-001), Days 1-10 metadata answer conventions
  (META-ANSWER-CONV-A104)
