# BOM Voyage

| | |
|---|---|
| Day / Lane | 3 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-03-bom-voyage` |
| Technique | Raw byte-prefix check (UTF-8 BOM `EF BB BF`) on ZIP members; hashes and magic bytes for the ranked variants |
| Single Solve | Solved (journal-recorded): the journal records our own submission as `CORRECT`; no result file was saved |
| Time Trial | Independent (original attempt journaled `CORRECT`; ranked re-run has a saved `result.json` `CORRECT`) |
| Mastery | Independent (original attempt journaled `CORRECT`; ranked re-run has a saved `result.json` `CORRECT`) |

## Summary

The prompt says some text files "look identical in an editor but behave differently in scripts" and asks for
the "encoding landmines". In practice the first-round question was one boolean: does a given file start with a
UTF-8 byte order mark. Reading the member's first three bytes straight out of the ZIP answered it. Single Solve,
Time Trial and Mastery were all solved this way, and a later ranked re-run of the Time Trial and Mastery showed the
same challenge asking seven fields per file instead of one.

## The challenge

Prompt: "A text export pipeline produced files that look identical in an editor but behave differently in scripts.
Find the encoding landmines before they break the investigation."

| Mode | Delivered | Questions (original attempt) |
|---|---|---|
| Single Solve | `metadata-day-03-evidence.zip` (205 bytes, one member `evidence-note.txt`, 33 bytes), untimed | 1: `text.utf8_bom_present` for `evidence-note.txt` |
| Time Trial | a ZIP holding `roster.csv` (29 bytes), 50 s server window | 1: `text.utf8_bom_present` for `roster.csv` |
| Mastery | a bundle of two nested ZIPs, 80 s server window | 2: one `text.utf8_bom_present` per set, IDs prefixed `set1.` / `set2.` |

The answer format is the enum `true` / `false`, normalized by trimming. The saved original start responses are not in
the leaf (no `start_response.json` was kept for the first attempts); the question text above comes from the journals.

## Solution

1. Hash the downloaded ZIP and compare it with the SHA-256 in the start response.
2. Read the named member directly from the archive. The BOM question is about the member's bytes, not the ZIP
   container's leading bytes (`PK`).
3. Compare the member's first three bytes with `EF BB BF`. The Single Solve member began `43 61 73 65`
   (`Case`), with a 33-byte body `Case 3DF9200ADB1A evidence note.\n`. No BOM, so the answer is `false`.
4. Submit `false` under `evidence-note-text-utf8-bom-present`. The Game Master returned `CORRECT`.

The whole check, from the solver in the leaf:

```python
UTF8_BOM = b"\xef\xbb\xbf"
content = source.read(matches[0])          # exactly one member must match
present = content.startswith(UTF8_BOM)
print(f"prefix_hex={content[:3].hex(' ')}")
print(f"text.utf8_bom_present={str(present).lower()}")
```

The same script has an `--archive-member` option to descend into a nested ZIP, and an exact-one-match guard on the
member name. Single Solve answer: `false`.

## Time Trial and Mastery

- **Time Trial (original).** `roster.csv` began `68 6f 73` (`hos...`): `false`. Submitted about 19 s into the 50 s
  window. The only change from Single Solve was the member name.
- **Mastery (original).** Two sets. Set 1 `evidence-note-copy.txt` (33 bytes, prefix `43 61 73`) was `false`. Set 2
  `export-001-with-bom.txt` (32 bytes, prefix `ef bb bf`) was `true`, the only positive in the whole challenge as far
  as the journals show. Both answers were sent in one submission about 28 s into the 80 s window. The solver was run
  once per nested ZIP.
- **Ranked re-run (2026-10-26).** The ranked Time Trial and Mastery attempts carried a larger fixture than the
  journals describe: seven questions per file (exact filename, size in bytes, MD5, SHA-1, SHA-256, first 16 bytes as hex,
  and the BOM boolean), 7 answers for the Time Trial and 14 (two sets of 7) for Mastery. Both went `CORRECT` in one
  submission call (about 1.3-1.4 s total each). In those fixtures every file was `evidence-note.txt` and every BOM answer was `false`.
  Their per-file values are recorded in the ranked `accepted_plan.json` files. The ranked solver was a general
  per-challenge solver under the ranked tooling rather than `solve_bom_voyage.py`; its source was not reviewed for this page, so we cannot say how it
  differs beyond producing those seven fields.

## Dead ends and gotchas

- **Answer IDs are not a function of the file name.** Single Solve and Time Trial IDs follow `<stem>-text-utf8-bom-present`,
  but Mastery set 2's ID was `set2.d03-text-001-text-utf8-bom-present` for the file `export-001-with-bom.txt`.
  The solver's default of deriving the ID from the member name would have been wrong there, so IDs were passed
  explicitly from the start response.
- **Member names change per mode** (`evidence-note.txt`, `roster.csv`, `evidence-note-copy.txt`,
  `export-001-with-bom.txt`). Do not hard-code one.
- **File type descriptions do not answer the question.** `file` called the Single Solve member "ASCII text", which says
  nothing about a BOM. The explicit byte comparison is the answer.
- **Do not retype digests.** An early scouting note in the Single Solve journal carried a mistyped SHA-256 (one
  stray character); the direct `sha256sum` output was the authority. It did not affect the answer.
- **Local archive copies can be damaged.** The saved Time Trial ZIP was later found to have line-ending damage
  (`BadZipFile`) and an extracted copy of the note was 34 bytes for a 33-byte member. The hash in the start response is the
  check; undoing the added CRs restores it. This did not affect the live solves, which used fresh bytes.

## Takeaways

- A "BOM" or "encoding" question is a byte-level question: read the first bytes of the exact member, never an editor view or
  a `file` summary. UTF-16/UTF-32 have different marks (`FF FE`, `FE FF`, ...), and a missing UTF-8 BOM does not say which encoding
  the file uses.
- Expect the first round of a lane to be tiny and the timed modes to widen: here one boolean in the original journaled
  attempts became seven hash/size/prefix fields in the ranked ones. A solver that already computes filename, size, digests and
  magic prefix for every member handles both.
- Take answer IDs from the start response, not from file names.
- Technique names to search: UTF-8 BOM prefix check, ZIP member byte inspection, nested-bundle triage.

## Evidence

Leaf base path: `challenges/day03/metadata-forensics/metadata-forensics-day-03-bom-voyage/`

- `single-solve/journal.md`, `single-solve/solve_bom_voyage.py`
- `time-trial/journal.md`, `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- `progress.md` rows for Day 3 BOM Voyage record all three modes `Solved` (basis "Player-directed local mark", 2026-10-18).
  Single Solve's only evidence is the journal entry; there is no local `result.json` for it.
