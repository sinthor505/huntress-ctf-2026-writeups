# Seven-Zip, No Relation

| | |
|---|---|
| Day / Lane | 7 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-07-seven-zip-no-relation` |
| Technique | Raw ZIP End-of-Central-Directory (EOCD) field read, plus byte-level metadata of a note, JPEG, PNG and PDF |
| Single Solve | Solved (journal-recorded): the journal records our one submission as CORRECT (progress mark 2026-10-20); no result or submission-response file was preserved for this mode |
| Time Trial | Independent (`open-1`, `ranked-1`, `ranked-2` all `result.json` CORRECT, 1 submission call each) |
| Mastery | Independent (`ranked-1/result.json` CORRECT; the original untimed solve is recorded in the journal and `submission-response.json`) |

## Summary

Day 7's metadata challenge introduced one new field, `zip.eocd_total_entries`: the entry count stored in
the End of Central Directory record of a ZIP nested inside the evidence bundle. The prompt hints at a
"census that does not add up", but in every fixture we saw the raw EOCD count matched the number of
central-directory entries a normal ZIP parser lists, so the field was a plain 22-byte struct read.
Time Trial and Mastery re-asked the earlier days' fields (note hashes, JPEG/PNG dimensions, PDF version)
plus the new one, and all three modes were solved offline in one submission each.

## The challenge

Prompt: "Files arrive inside and beside a collection of ZIP archives. Count what the archives claim,
distinguish empty space from evidence, and find the census that does not add up."

- **Single Solve:** one question, `d07-zip-001-zip-eocd-total-entries` ("Determine zip.eocd_total_entries
  for evidence-drop-001.zip"), integer, base-10. The delivered `metadata-day-07-evidence.zip` held a single
  member, the 410-byte `evidence-drop-001.zip`.
- **Time Trial:** 11 fields from one bundle containing `evidence-note.txt`, `camera-001.jpg`,
  `panel-001.png`, `document-001.pdf` and `evidence-drop-001.zip`: note filename, size, MD5, SHA-1, SHA-256,
  first 16 bytes as hex, UTF-8 BOM present (`true`/`false`); JPEG SOF dimensions; PNG IHDR dimensions; PDF
  header version; and the nested ZIP's EOCD total entries.
- **Mastery:** the same 11 fields for each of two sets (22 answer IDs, prefixed `set1.` / `set2.`), delivered
  as an outer ZIP holding `metadata-day-07-evidence-01.zip` and `-02.zip` (each 3,309 bytes, each containing the
  five members above).

All answers are submitted as JSON strings.

## Solution

Single Solve:

1. Hash-verified the outer ZIP against the start response, then read the nested `evidence-drop-001.zip`
   (410 bytes) from it.
2. Located the EOCD record (`PK\x05\x06`) and unpacked its fixed layout little-endian. The nested
   archive's EOCD sat at offset 388, with a 146-byte central directory at offset 242.
3. The EOCD distinguishes entries on this disk from total entries; both were 2. Python's `zipfile` listed the
   same two members (`room-01/item-1.txt`, `room-02/item-1.txt`), so the raw field and the parsed directory agreed.
4. Submitted `2`; the Game Master returned CORRECT.

The whole extractor was this struct read:

```python
EOCD = b"PK\x05\x06"
EOCD_STRUCT = struct.Struct("<4s4H2IH")   # sig, disk, cd_disk, entries_on_disk, total_entries, cd_size, cd_off, comment_len

data = outer.read(member)
offset = data.rfind(EOCD)
fields = EOCD_STRUCT.unpack_from(data, offset)
print("entries_on_disk", fields[3], "eocd_total_entries", fields[4])
```

Reading the raw field rather than trusting a ZIP library is the point of the question: a library can hide a
count/directory mismatch, and the question explicitly asks what the footer claims. The journal afterwards
generalized this into a library parser that also searches the maximum comment window, requires the comment
length to reach the file end, and reports ZIP64 sentinels.

## Time Trial and Mastery

- **Time Trial** answers came from reading the bundle's members directly: size and MD5/SHA-1/SHA-256 of
  `evidence-note.txt` (33 bytes, no UTF-8 BOM), the first 16 bytes as hex, JPEG dimensions from the SOF marker,
  PNG dimensions from IHDR, the PDF's `%PDF-x.y` header, and the EOCD count from the nested ZIP. All of these
  were techniques already in the library from Days 1-6, so the only genuinely new reader was the EOCD one.
- **Mastery** had two sets. In the accepted fixtures both sets shared the same shapes (33-byte notes without a
  BOM, JPEG 257x146, PNG 269x153, PDF 1.4, EOCD total 2), but the note text, and therefore its three digests and
  16-byte prefix, differed per set. A small script (`artifacts/answer_map.py`) turned per-set extracted facts into
  `setN.<id>` answers and asserted its key set equaled the start response's `answer_ids` before anything was
  submitted. Both sets were answered in one 22-field submission, CORRECT.
- **Timing.** Windows were not an issue on the original attempts (the start responses carried `deadline: null`).
  Later ranked reruns used a 50 s (Time Trial) and 80 s (Mastery) window and finished in roughly 1-2 s
  end to end with the per-challenge solver, which resolves ids, members and containers only from the start
  response. The rerun solver is shared with the other early metadata days.

## Dead ends and gotchas

- The prompt's "census that does not add up" and "distinguish empty space from evidence" suggest hidden or
  slack data, or an EOCD that disagrees with the directory. In every fixture we saw, the EOCD total equaled the
  central-directory count. A later review reproduced all 34 accepted fields (1 + 11 + 2x11) and found the field never
  differed from the parsed count. We could not determine whether other variants in the pool carry a real mismatch.
- A bare `rfind(b"PK\x05\x06")` is enough on these tiny archives, but it would be fooled by that signature inside
  an archive comment; the later library parser guards against that.
- The nested ZIP is a separate specimen from the other four members, not the container for them: the note, JPEG,
  PNG and PDF are siblings of it in the bundle.
- All values are strings; integers and the `true`/`false` BOM enum are submitted as text. The 16-byte prefix is
  32 lowercase hex characters with no `0x`.

## Takeaways

- **ZIP EOCD census:** read the raw end-of-central-directory fields yourself (`<4s4H2IH`) when a question asks what
  a ZIP footer claims, and keep per-disk and total counts separate. Compare against a standard parser as a
  cross-check, not as the source.
- Nested-archive questions: extract the named inner member's bytes first, then inspect those exact bytes.
- For Mastery bundles, drive the extractor from the start response's set/ID list and assert the key set matches
  before submitting.
- Faster next time: skip the per-challenge extractor and use the library EOCD parser (META-ZIP-EOCD-001) directly.

## Evidence

Leaf base: `challenges/day07/metadata-forensics/metadata-forensics-day-07-seven-zip-no-relation/`

- Single Solve: `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/artifacts/inspect_zip_eocd.py`
- Time Trial: `time-trial/journal.md`, `time-trial/answer-candidate.json`, `time-trial/submission-response.json`,
  `time-trial/open-1/result.json`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- Mastery: `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/artifacts/facts.json`,
  `mastery/submission-response.json`, `mastery/ranked-1/result.json`
- Rerun solver (outside the leaf): `tools/ranked/solvers/metadata-forensics-day-07-seven-zip-no-relation/solve.py`
- Library entries: META-ZIP-EOCD-001, META-JPEG-SOF-001, META-PNG-STRUCTURE-001, META-PDF-HEADER-001, META-TEXT-BOM-001
- `progress.md` rows for Day 7 list all three modes as Solved (2026-10-20).
