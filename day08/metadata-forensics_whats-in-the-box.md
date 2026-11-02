# What’s in the Box?

| | |
|---|---|
| Day / Lane | 8 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-08-whats-in-the-box` |
| Technique | Container metadata extraction from a nested evidence bundle (hashes, JPEG SOF, PNG IHDR, PDF header, ZIP EOCD, ZIP central-directory paths) |
| Single Solve | Independent (CORRECT on submission 2 of 2; the first was rejected for value types, nothing was graded) |
| Time Trial | Independent (original attempt: journal entry only, CORRECT on attempt 1; the ranked-1 `result.json` is the saved graded result, CORRECT on 1 call) |
| Mastery | Independent (original attempt: journal entry only, CORRECT on attempt 1; the ranked-1 `result.json` is the saved graded result, CORRECT on 1 call) |

## Summary

A 3 KB evidence ZIP held a text note, a JPEG, a PNG, a PDF and a nested ZIP. The challenge asked for 12 byte-level facts about them. Nothing needed more than the standard library: read each container's own header fields and hash the note. Time Trial reused the same 12-field template on a fresh archive, and Mastery ran it across five archives (60 fields). Every mode went CORRECT on its first full submission.

## The challenge

The prompt, identical in all three modes: "The count was only the beginning. Inventory every stored path, preserve directory context, and identify the archive whose names reveal an impossible provenance claim."

The delivery was `metadata-day-08-evidence.zip` (3,308 bytes, five members in this order: `evidence-note.txt`, `camera-001.jpg`, `panel-001.png`, `document-001.pdf`, `evidence-drop-001.zip`). The 12 answer fields were:

- For the note: filename, size in bytes, MD5, SHA-1, SHA-256, the first 16 bytes as hex, and whether a UTF-8 BOM is present.
- `jpeg.sof_dimensions_pixels` for the JPEG, `png.ihdr_dimensions_pixels` for the PNG, `pdf.header_version` for the PDF.
- For the nested ZIP: `zip.eocd_total_entries`, and `zip.central_directory_member_names` as a compact JSON array.

Answer formats were `exact_text`, `integer`, `pattern` (hex digests, case ignored), `enum` (`true`/`false`) and `json`.

The prompt's "impossible provenance claim" had no matching question in any mode. The only place names could "reveal" anything was the nested ZIP, where two members shared a basename under different directories (below). We could not determine from the files what the prompt's last clause was meant to point at, and it did not affect any answer.

## Solution

1. **Inventory the outer ZIP** with `zipfile.infolist()`, which gave the five names above. The downloaded bytes were checked against the start response's SHA-256 first.
2. **Note fields.** Size, MD5, SHA-1 and SHA-256 come from the member bytes. The magic prefix is `note[:16].hex()` (`43617365204139373036454146314535`, the ASCII text `Case A9706EAF1E5E`). The BOM check is a comparison with `ef bb bf`; the note starts with plain ASCII, so the answer is `false`.
3. **JPEG dimensions.** Walk the marker segments to the first SOF marker (`C0`..`CF`, excluding the non-SOF codes `C4`, `C8` and `CC`) and read height then width: `257x146`. `file` agreed.
4. **PNG dimensions.** The first chunk must be `IHDR`; read two big-endian `uint32`s at offset 16: `269x153`. Chunk CRCs were also checked and the file had no trailing data.
5. **PDF version.** `%PDF-` in the first 1024 bytes gave `1.4`.
6. **Nested ZIP.** The EOCD record is found by searching backward for `PK\x05\x06`. The field at offset 10 is the total entry count (2), and the comment length is checked so that the record ends exactly at end-of-file. Central-directory paths were `room-01/item-1.txt` and `room-02/item-1.txt`. Both members are 48 bytes with CRC `698216d4` and contain identical bytes. Only the directory component tells them apart, so the answer keeps the full stored path, not the basename. This is the "preserve directory context" hint in the prompt.
7. **Submit as strings.** Every value, including the integers, the boolean and the array, had to be a JSON string. The member-name array was sent as compact JSON text.

The decisive extractor is a plain `struct` reader, for example the nested-ZIP part:

```python
eocd = nested_bytes.rfind(b"PK\x05\x06")
comment_length = struct.unpack_from("<H", nested_bytes, eocd + 20)[0]
if eocd + 22 + comment_length != len(nested_bytes):
    raise ValueError("nested ZIP EOCD comment does not reach end of file")
eocd_total_entries = struct.unpack_from("<H", nested_bytes, eocd + 10)[0]
```

## Time Trial and Mastery

- **Time Trial** was the same five-member layout and the same 12 fields on a freshly issued archive. The Single Solve mapper was reused unchanged, and it recomputed every value from the new bytes (no constants carried over). It also asserted that its key set equalled the start response's `answer_ids` before submitting. One call, CORRECT.
- **Mastery** delivered an outer ZIP of five inner archives (`metadata-day-08-evidence-01.zip` to `-05.zip`), each with the same five members and the same 12 fields. Answer IDs were prefixed `set1.` to `set5.` (60 fields). The mapper was generalized over N sets: it takes the set number from the inner file name, selects members by name rather than by position, and prefixes the keys. One call, CORRECT.
- Both modes were issued without a deadline in the original runs. In a later scripted ranked rerun (a per-challenge solver driven by the start response), the Time Trial window was 50 s and the Mastery window 170 s. The whole pass, from issue through download, solve and submit, took about 1.6 s and 1.8 s, each with a single submission call. Both reruns were CORRECT.

## Dead ends and gotchas

- **`invalid_answers` on the first Single Solve submission.** The answers were sent as native JSON types (ints, a bool, a list). The validator wants strings throughout and graded nothing. Converting to strings fixed it. This is a lane-wide rule, not specific to this challenge.
- **Member-name order was never actually tested here.** We submitted the central-directory order and it was accepted. A later review found that every nested ZIP on Days 8 to 22 had central-directory order equal to sorted order, so those acceptances cannot distinguish the two rules. A later day (Day 23) did discriminate and accepted the ascending lexical sort, so the safe answer is the full stored paths sorted. We revised the library rule after the fact; it did not change this challenge's result.
- **Raw names, not `ZipInfo.filename`.** On Windows the standard library rewrites a stored backslash to `/`. For a field about stored names, take them from the raw central directory. This did not bite on Day 8, where the names contain only `/`.
- **Fixtures were not all alike.** Mastery sets and later attempts shift which values appear (hashes, dimensions, the note's case ID), so nothing should be hard-coded from one archive.

## Takeaways

- For "metadata of a bundle" questions, parse the container fields directly (JPEG SOF segment, PNG IHDR, PDF header, ZIP EOCD, central directory). A hand-rolled `struct` reader is shorter and safer than a general library for these fields, and it is the same reader that scales to N Mastery sets.
- Keep full paths and duplicates when asked for stored names, and sort them unless the question defines an order.
- Cross-check the EOCD count against the number of central-directory entries you actually walked; for raw-field questions, don't rely only on a library's convenience listing.
- Assert that the mapper's keys equal the start response's `answer_ids` and string-convert every value before submitting. This avoids wasted submissions.

## Evidence

Leaf: `challenges/day08/metadata-forensics/metadata-forensics-day-08-whats-in-the-box/`

- `single-solve/journal.md`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-response.json` (CORRECT), `single-solve/artifacts/start-response.json`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json` (CORRECT)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py` (N-set generalization), `mastery/ranked-1/result.json` (CORRECT)
- `tools/ranked/solvers/metadata-forensics-day-08-whats-in-the-box/solve.py` (the per-challenge solver used for the ranked reruns)
- Library: ZIP member paths (META-ZIP-MEMBER-PATHS-001), ZIP EOCD, JPEG SOF, PNG structure, PDF header and text BOM entries under `library/metadata-forensics/`
- `progress.md` rows for Day 8: Solved, 2026-10-20 (Single Solve) and 2026-10-23 (Time Trial, Mastery)
