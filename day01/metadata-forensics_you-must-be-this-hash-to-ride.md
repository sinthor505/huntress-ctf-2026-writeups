# You Must Be This Hash to Ride

| | |
|---|---|
| Day / Lane | 1 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-01-hash-to-ride` |
| Technique | File identity inventory: name, size and MD5/SHA-1/SHA-256 of a ZIP member |
| Single Solve | Solved, graded result not preserved (local mark only) |
| Time Trial | Independent (ranked attempt, CORRECT on call 1) |
| Mastery | Independent (ranked attempt, CORRECT on call 1) |

## Summary
The warm-up of the metadata lane. A tiny ZIP holds one text file, and the task is to report its
filename, byte length and three cryptographic digests. The only insight is to compute everything from
the bytes rather than trust any label. Day 1 was played on 2026-10-18 and recorded as a
"player-directed local mark" with no journal or graded result saved. The Time Trial and Mastery
modes were re-run on 2026-11-01 as ranked attempts and both went CORRECT on the first submission.

## The challenge
The prompt: "Three ordinary files arrived without trustworthy labels. Build an identity inventory from
the bytes themselves and prove which labels can-and cannot-be trusted." Despite the "three files"
wording, the delivery contained a single member, `evidence-note.txt`, 240 bytes, inside a 372-byte ZIP.
The questions were five fields about that note:

| Answer ID | Format |
|---|---|
| `evidence-note-filename` | `exact_text`, base filename with extension, no path |
| `evidence-note-size-bytes` | base-10 integer |
| `evidence-note-md5` | 32 hex characters, case ignored |
| `evidence-note-sha1` | 40 hex characters, case ignored |
| `evidence-note-sha256` | 64 hex characters, no prefix, case ignored |

All answers are submitted as JSON strings, including the integer.

## Solution
1. Download the ZIP and check its SHA-256 against the start response.
2. List the archive and take the one member. The question's `artifact` field names it
   (`metadata-day-01-evidence.zip -> evidence-note.txt`).
3. Read the member's uncompressed bytes and compute the length plus MD5, SHA-1 and SHA-256. The
   filename is the member's base name, with no directory part.
4. Submit all five fields in one call.

The hashes are of the decompressed member content, not of the ZIP file. Nothing else was needed. The
only fields at risk are the filename (exact text, extension included) and the digests (lowercase hex
with no prefix).

For the record, the Time Trial attempt's accepted values were `evidence-note.txt`, `240`,
`6dc938630c5942446263dea48a2aef00` (MD5), `3aaee3b204fc8901eb58279b97503a8fef87e17e` (SHA-1) and
`7a42f628fd4af2e966a29ffab37a7cfade32a04f0a5daa82d8369d0a8eb3b03c` (SHA-256). They are per attempt, so
they carry no value now. The Single Solve ZIP in the leaf has the same shape: one `evidence-note.txt` of
240 bytes. We do not have the graded Single Solve answers.

## Time Trial and Mastery
- **Time Trial:** the same five questions over a fresh ZIP with the same layout. The ranked attempt had a
  50 s window. It took 5.5 s end to end and was accepted on the first submission.
- **Mastery:** the delivery was an outer ZIP containing two inner evidence ZIPs
  (`metadata-day-01-evidence-01.zip`, `-02.zip`). Each holds its own `evidence-note.txt`. The question IDs
  were prefixed `set1.` and `set2.`, so ten answers in total. Both notes were 240 bytes and only the digests
  differed. The ranked window was 80 s, total 5.7 s, accepted on the first submission.

Both modes were played as 2026-11-01 ranked re-runs through the shared Day 2 metadata-lane solver, which
already answers this note family (filename, size, three digests). It reads the member and set structure
from the start response. A first attempt on these ranked modes had earlier returned 409
`legacy_attempt_pinned` (an "earlier version of this challenge" pin); retrying later issued the attempt
normally.

## Dead ends and gotchas
- The "three files" and "which labels can be trusted" framing suggests a trap, such as a misnamed
  extension or a spoofed header. In the data we examined there was none: a single `.txt` member with an
  honest name. The challenge was straightforward. We could not determine from the leaf whether the
  Single Solve differed.
- Mastery needs a manifest-driven reader: the member is resolved per set from the question's `artifact`
  field, and answer IDs carry the `setN.` prefix.
- Early Time Trial and Mastery windows on legacy-route days were short, so these were scripted
  download, solve and submit passes rather than interactive ones.

## Takeaways
- For "file identity inventory" questions, hash the *decompressed member bytes* with Python's `hashlib`
  and `zipfile`, and take the filename from the archive entry. Digest format is lowercase hex with no
  prefix.
- Keep one general note-family reader (filename, size, MD5, SHA-1, SHA-256, plus later fields such as magic
  prefix) and dispatch by the question's `artifact` field. It handled this day's Mastery without changes.
- The same five fields recur on later days (5, 7-9) with only the container changing.

## Evidence
Leaf base: `challenges/day01/metadata-forensics/metadata-forensics-day-01-hash-to-ride/`

- `single-solve/metadata-day-01-evidence.zip` (no journal, start response or result; `progress.md` basis
  is a player-directed local mark of 2026-10-18)
- `time-trial/ranked-1/{start_response.json,accepted_plan.json,result.json,pass.log}`
- `mastery/ranked-1/{start_response.json,accepted_plan.json,result.json,pass.log}`
- Library: the metadata-forensics answer conventions for Days 1-10 (Day 1 rows inferred, as noted there).
- The solver is the Day 2 metadata-lane ranked solver, which reuses the shared note-family engine under
  `tools/ranked/solvers/`.
