# Product Placement

| | |
|---|---|
| Day / Lane | 13 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-13-product-placement` |
| Technique | MSI `Property` table read (ProductCode), answer-format quirk (braces kept) |
| Single Solve | Solved (journal-recorded); the journal records our own CORRECT on attempt 2, no result file was saved |
| Time Trial | Independent (journal-recorded original attempt, plus two later ranked reruns with saved CORRECT results) |
| Mastery | Independent (journal-recorded original attempt, plus two later ranked reruns with saved CORRECT results) |

## Summary

Each delivery is a ZIP holding a small MSI installer, and the question is one field:
`msi.product_code_guid`. The value is the `ProductCode` row of the MSI `Property` table. The only real
trap was the answer format. The prompt advertises a brace-free canonical GUID, but the validator wanted
the value exactly as stored, braces and upper case included. Single Solve cost two submissions because of
that; Time Trial and Mastery then went CORRECT on the first call.

## The challenge

Prompt: "Installers were copied, renamed, and repackaged. Recover stable product identities,
distinguish revisions from different products, and find the collision." The flavour text suggests a
comparison across several specimens, but the first-day fixtures asked for a single field on a single file.

- **Single Solve:** `metadata-day-13-evidence.zip` (one 9,728-byte member, `installer-001.msi`), one answer
  `d13-msi-001-msi-product-code-guid`. Untimed.
- **Time Trial:** same shape, `installer-002.msi`, one answer, 50-second window.
- **Mastery:** one 6,973-byte bundle holding two nested ZIPs (`installer-003.msi`, `installer-004.msi`), two
  answers (`set1.` / `set2.` prefixed), 80-second window.
- Answer format as advertised: `pattern`, "canonical GUID text in 8-4-4-4-12 hexadecimal form, including
  hyphens", normalization `trim`.

## Solution

1. **Identify the file.** The ZIP hash matched the start response. `file` called the member a Composite
   Document File / MSI database, and `msiinfo tables` listed a `Property` table.
2. **Read the right GUID.** An MSI carries at least three identities: `ProductCode` (the installed
   product), `UpgradeCode` (the product family) and the Summary Information "Revision number" UUID (the
   package code). The field name asks for the first.
   `msiinfo export installer-001.msi Property` gave:

   ```text
   UpgradeCode    {5F63B105-6C37-46FB-B671-2AF6C4F4CD67}
   ProductCode    {DD1A80D5-0250-4F4D-B289-13096D633C61}
   ProductName    Huntress Metadata Evidence
   ProductVersion 1.13.0
   ```

   `msiinfo suminfo` separately printed a different Revision UUID (`{99BE1D60-...}`), so it is not the
   answer.
3. **Map to the answer.** The extractor runs `msiinfo export`, picks the `ProductCode` row, checks that the
   inner text is a canonical GUID, and returns the raw property text:

   ```python
   raw = properties["ProductCode"].strip()
   guid = raw[1:-1] if raw.startswith("{") and raw.endswith("}") else raw
   if not re.fullmatch(r"[0-9A-Fa-f]{8}(?:-[0-9A-Fa-f]{4}){3}-[0-9A-Fa-f]{12}", guid):
       raise ValueError(f"ProductCode is not a canonical GUID: {raw!r}")
   return raw   # braces kept: the validator wanted the stored form
   ```

4. **Submit.** Attempt 1 sent the brace-free upper-case GUID, as the prompt describes, and returned
   `incorrect`. Attempt 2 sent `{DD1A80D5-0250-4F4D-B289-13096D633C61}` and returned CORRECT.

## Time Trial and Mastery

Nothing new to solve. Both installers carried the same `ProductCode`, so the Time Trial answer (one field)
and both Mastery answers (two fields) were the same braced value, accepted on call 1 each within their
windows (the 50 s and 80 s windows were short enough that the recorded flow was extract, then submit).

The journals do not say why the "collision" in the flavour text never mattered. The library note written
afterwards records that the four Day 13 MSIs, and the MSIs in the Day 16-20 bundles, all shared one
ProductCode, UpgradeCode and package code. So the "collision" is that renaming and repackaging did not
change the identities. We did not verify that ourselves beyond the Day 13 files and the library note.

Later ranked reruns of this challenge (bragging rights only, not the prize ranking) used a different,
larger fixture: the same prompt, but the ZIP held a cumulative recap bundle. Time Trial asked 18 fields
(note and roster hashes and sizes, JPEG/PNG dimensions, PDF version, ZIP entries, PE header fields, .NET MVID,
the MSI ProductCode, and a synthesis ordering). Mastery asked 90 (5 sets of 18). The MSI field was the
same `d13-msi-001-msi-product-code-guid` and took the same braced value. Each rerun was CORRECT in one
submission, twice for each mode.

## Dead ends and gotchas

- **Braces and case.** The advertised syntax says nothing about braces, yet the brace-free form was
  `incorrect`. The stored `{...}` upper-case form is the one accepted, consistently across Single Solve, Time
  Trial, Mastery and the ranked reruns. Submit the property text verbatim. This contrasts with a .NET MVID
  (a bare lowercase GUID, seen in the same ranked recap).
- **Three GUIDs, one file.** `ProductCode`, `UpgradeCode` and the Summary Information revision UUID are all
  plausible-looking answers to "product identity". Only the Property-table `ProductCode` matches the field
  name.
- **Decoys.** The ZIP member name (`installer-001.msi`) and `ProductName` say nothing about identity.
- **Do not reuse values across days.** The same ProductCode recurred through Day 22, but Day 23 drew a
  different one per set; the value must always come from the delivered file.

## Takeaways

- For `msi.*` fields, read the `Property` table (`msiinfo export <file> Property` on Linux, or a small CFB
  reader: MSI stream names use a custom base-64 alphabet and the `_StringPool`/`_Columns` streams describe
  the rows). Keep Summary Information distinct.
- When a validator's advertised format and a stored-value form disagree, try the stored form first. It cost
  one wasted submission here.
- The flavour text of metadata challenges is often broader than the actual question; read the answer ID
  and field name first.
- Search terms: MSI ProductCode, UpgradeCode, Windows Installer Property table, `msiinfo`, OLE/CFB.

## Evidence

Base path: `challenges/day13/metadata-forensics/metadata-forensics-day-13-product-placement/`

- `single-solve/journal.md`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/diag-01-*.json`
  (rejected brace-free try), `diag-02-*.json` (accepted braced form). No `result.json` in this mode.
- `time-trial/journal.md`, `time-trial/ranked-1/result.json`, `ranked-2/result.json` (CORRECT, 1 call each).
- `mastery/journal.md`, `mastery/ranked-1/result.json`, `ranked-2/result.json` (CORRECT, 1 call each),
  `mastery/artifacts/answers.json`.
- `progress.md` rows for Day 13 Product Placement (all three modes recorded as solved on 2026-10-20).
- Library: `library/metadata-forensics/msi-product-identity.md` (META-MSI-PRODUCT-IDENTITY-B102).
