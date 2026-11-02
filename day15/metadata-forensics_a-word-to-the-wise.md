# A Word to the Wise

| | |
|---|---|
| Day / Lane | 15 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-15-word-to-the-wise` |
| Technique | OOXML relationship parsing (`TargetMode="External"`), validator-defined output ordering |
| Single Solve | Independent (journal-recorded CORRECT on attempt 3; the submission response file is preserved in the leaf) |
| Time Trial | Independent (journal-recorded CORRECT on attempt 8; later ranked reruns also CORRECT, `ranked-1/result.json`, `ranked-2/result.json`) |
| Mastery | Independent (journal-recorded CORRECT on attempt 11; later ranked reruns also CORRECT, `ranked-1/result.json`, `ranked-2/result.json`) |

## Summary

Day 15's metadata challenge asks which external dependencies a Word document reaches for. The
parsing is easy: read the `.rels` parts inside the DOCX and keep the relationships with
`TargetMode="External"`. The expensive part was the answer shape. The validator wanted the external
targets as a compact JSON array in **ascending lexical order of the target strings**, not in
document order, and it did not accept the "evidence-looking" link alone. Time Trial and Mastery wrapped the same
OOXML question in a 20-field evidence bundle (5 such bundles in Mastery) and went CORRECT once the
ordering rule and the question-specific file lists were respected.

## The challenge

Prompt: "Office documents contain relationships that point beyond the page. Separate internal
references from external dependencies and find the document reaching somewhere it should not."

- **Single Solve.** One question, `ooxml.external_relationship_targets` for `document-15-001.docx`,
  format `json` ("compact valid JSON array with exact values and order"), normalization `trim`. The
  delivered `.bin` was a 1,316-byte ZIP holding the single 3,803-byte DOCX. No timer.
- **Time Trial.** 20 fields over an 11-member `evidence.zip`, 50-second server clock. Fields covered
  the evidence note (size, MD5, SHA-1, SHA-256, first 16 bytes, UTF-8 BOM flag), roster filename,
  JPEG SOF dimensions, PNG IHDR dimensions, PDF header version, a nested ZIP (EOCD entry count and
  central-directory member names), PE facts (COFF machine, entry-point RVA, version-resource
  `OriginalFilename`), CLR MVID, MSI ProductCode, LNK target path, the OOXML external targets, and one
  synthesis question: SHA-256 each of four named files and return their filenames in ascending digest
  order. Format mix: 1 enum, 6 exact_text, 2 integer, 3 json, 8 pattern.
- **Mastery.** The same 20-field shape times five sets (100 answer IDs, IDs prefixed `set1.` to
  `set5.`), delivered as an outer ZIP of five nested `evidence-0N.zip` files, 170 seconds.

## Solution

1. **Recon.** The Single Solve ZIP held one DOCX with five parts. `word/document.xml` references two
   hyperlinks by `rId9` and `rId10`, but a hyperlink in the body carries only an ID; the real target
   and whether it is external live in the relationship parts.
2. **Read every `.rels` part.** `_rels/.rels` held two relationships with no `TargetMode`
   (internal: `word/document.xml`, `docProps/core.xml`). `word/_rels/document.xml.rels` held `rId9`
   (`https://example.test/evidence/d87ca62c5bb7`) and `rId10` (`https://example.test/control`), both
   `TargetMode="External"`, in that XML order. The extractor is plain stdlib:

   ```python
   for part in docx.namelist():
       if not part.endswith(".rels"):
           continue
       root = ET.fromstring(docx.read(part))
       for rel in root.findall(f"{{{REL_NS}}}Relationship"):
           mode = rel.get("TargetMode", "Internal")
           records.append({"part": part, "id": rel.get("Id"),
                           "target": rel.get("Target"),
                           "external": mode.casefold() == "external"})
   ```

3. **The answer shape took three submissions** (see below). The accepted value was the compact JSON
   string, sent as a string, with the targets sorted:

   `["https://example.test/control","https://example.test/evidence/d87ca62c5bb7"]`

## Time Trial and Mastery

Both modes reuse the OOXML reader as one field among 20 per set, so the work was generalizing across
fixtures rather than new analysis:

- **Time Trial.** The first timer (50 s) expired unscored; `retry` issued a different variant (a new archive), and a second
  retry produced yet another. The first full 20-field map, submitted against one variant, came back
  `incorrect` with no field-level feedback. The journal says only that a later helper download
  overwrote that variant's bytes, so the cause of that rejection could not be pinned down; a stale
  map is the likely reading (see the leaf's deep-review note), not a confirmed one. On the next
  variant the answers were submitted by family and each returned cumulative accepted IDs:
  roster (3), evidence note (4), JPEG/PNG/PDF (3), nested ZIP (2), PE metadata (3), CLR/MSI/LNK/OOXML
  (4), and finally the SHA-256 synthesis field, which returned CORRECT on submission 8 of 10 allowed.
- **Mastery.** The first assignment's five 20-field sets were submitted as set groups. Sets 1-3 and
  5 were accepted (80 fields), set 4's group was rejected, and its fields were split by family
  until 95 of 100 were accepted. The last five (CLR MVID, MSI ProductCode, LNK target, OOXML, and the
  synthesis field) were rejected together and then the timer expired. A review of the question text
  found why: the synthesis question's `Files:` list differs per set. Sets 1, 2, 3 and 5 name
  `evidence-note.txt`, `panel-001.png`, `telemetry-agent-001.exe`, `document-15-001.docx`; **set 4
  names `camera-001.jpg` instead of `panel-001.png`**, and the mapper had hashed the PNG. A timed
  `retry` issued a new 100-field variant (earlier acceptances did not carry over); re-deriving all
  100 values from that archive's bytes with the corrected file list, in one submission, returned
  CORRECT (attempt 11 on that variant, within the renewed deadline).
- Later ranked reruns of both modes (a speed-scoreboard sweep, bragging rights only) went CORRECT in
  one call each, using the same solver (`t_total` about 1.6-3.1 s).

## Dead ends and gotchas

- **Document order was wrong.** The XML-order array (`evidence` first, then `control`) was rejected
  on Single Solve attempt 1.
- **The "control" link is not a decoy.** The prompt's "document reaching somewhere it should not"
  suggested the `.../evidence/<hex>` URL alone; that one-element array was rejected on attempt 2. The
  validator counts every `TargetMode="External"` relationship, including the generic-looking one.
- **Sorting rule.** In this document the two targets sort the same way by target string and by `rId`
  string (`rId10` < `rId9` lexically), so Day 15 could not distinguish them. Later days with 3-5
  externals per document showed ascending code-point order of the target strings is the rule (taken
  from the library entry's deep review, not re-derived here).
- **Answer type.** The array is submitted as a JSON-encoded string, not a native array; every answer
  value in this lane must be a string.
- **Synthesis `Files:` lists are per question and per set** (set 4 swapped the PNG for a JPEG). Read
  the file names from the question, not from a previous set.
- **Rejected batches say nothing about which field is wrong.** Both the single-field Single Solve and
  the multi-field modes only returned batch-level `incorrect`; progress is visible only through the
  accepted-ID lists on partial submissions, which is why submitting by field family was the
  efficient route.

## Takeaways

- **OOXML hyperlinks need their `.rels` part.** Walk every `.rels` in the package and filter on
  `TargetMode` (case-insensitive `External`); do not trust link text or guess "which link is the
  suspicious one".
- **When the format says "exact values and order", ask what order the validator uses.** Here it was
  sorted targets, not document order. With a one-field question, three submissions was the price of
  learning that.
- **Per-set file lists in Mastery questions vary.** Resolve the file named by each question from the
  question text, per set.
- A generic OOXML reader and a streaming SHA-256-order helper were promoted into the library after
  this challenge and later reused on Days 16-23 bundles.

## Evidence

Leaf base path: `challenges/day15/metadata-forensics/metadata-forensics-day-15-word-to-the-wise/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/extract_ooxml_external_targets.py`,
  `single-solve/artifacts/submission-01-response.json` through `submission-03-response.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/metadata_answer_map.py`,
  `time-trial/artifacts/submission-0N-response.json`, `time-trial/ranked-1/result.json`,
  `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/mode_answer_map.py`,
  `mastery/artifacts/submission-attempt-log.json`, `mastery/ranked-1/result.json`,
  `mastery/ranked-2/result.json`
- Ranked solver: `tools/ranked/solvers/metadata-forensics-day-15-word-to-the-wise/solve.py`
- Library: OOXML relationships entry `META-OOXML-RELATIONSHIPS-B104`, SHA-256 order entry
  `META-SHA256-ORDER-B105`
- `progress.md` (grep `^| 15 .*word-to-the-wise`)
