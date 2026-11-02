# Managed Expectations

| | |
|---|---|
| Day / Lane | 12 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-12-managed-expectations` |
| Technique | read the .NET Module.Mvid from CLR metadata (PE -> CLI header -> `#~` Module row -> `#GUID` heap) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Solved (journal-recorded) |
| Mastery | Solved (journal-recorded) |

## Summary

Every mode asked for one `dotnet.mvid_guid`: the Module Version ID of a small managed PE. The
assemblies share a name, version and build ancestry, so none of those strings help. The value has to
be read from the Module table row and the `#GUID` heap, and the first three GUID fields have to be
formatted little-endian. One stdlib parser solved all three modes. Each was accepted once an
answer-ID slip in Time Trial was fixed.

## The challenge

Prompt: "The .NET assemblies share names, versions, and source ancestry, but their module identities
remain distinct. Separate rebuilds from renames." The delivery was a ZIP holding one 5,120-byte
`managed-NNN.dll`, a PE32 console assembly. Each question was `Determine dotnet.mvid_guid for
managed-NNN.dll.`, with the answer a canonical `8-4-4-4-12` GUID, trim normalization.

- Single Solve: 1 field (`managed-001.dll`), untimed.
- Time Trial: 1 field (`managed-002.dll`), a fresh ZIP, deadline about 50 s after start.
- Mastery: 2 fields (`set1.` / `set2.` prefixed IDs) in an outer ZIP wrapping two nested ZIPs, one DLL
  each (`managed-003.dll`, `managed-004.dll`), deadline about 80 s after start.

Answer IDs embed the file number and are per mode (`d12-dotnet-001-...` through `-004-...`).

## Solution

1. **Recon.** `file` called the member a "Mono/.Net assembly"; `objdump -x` showed a nonzero CLR
   Runtime Header directory. `strings -el` turned up the lure: `Huntress.Managed.<hex>`, version
   `2.0.2.6`, and a `--huntress-mvid` string. These are assembly-level metadata, not the Module
   identity, so they were treated as leads only. Version-resource parsing was ruled out for the same
   reason.
2. **Where the MVID lives.** The CLR metadata Module table has exactly one row. Its `Mvid` column is an
   index into the `#GUID` stream. There is no checksum or file-level field that holds it.
3. **Parse it by hand with the standard library.** The extractor walks:
   - PE section table, with an RVA-to-offset mapper that bounds-checks every read;
   - data directory 14 for the CLI header, then its metadata RVA, then the `BSJB` root and stream headers;
   - the `#~` header: `HeapSizes` byte (string/GUID index widths of 2 or 4 bytes), the `Valid` mask, and
     per-table row counts, to find where the Module row starts;
   - the Module row layout `Generation(2) Name(strings idx) Mvid(guid idx) ...`, so the MVID index sits at
     `row + 2 + string_width`.
4. **Index and byte order.** GUID heap indexes are 1-based, so the entry is at `heap_off + (idx-1)*16`.
   The 16 bytes are stored with the first three fields little-endian:

   ```python
   guid_off = heap_off + (mvid_index - 1) * 16
   raw_guid = data[guid_off:guid_off+16]
   return str(uuid.UUID(bytes_le=raw_guid))
   ```

   For `managed-001.dll` the raw bytes `f310e42177c95c4680eb6890ddd3085c` became
   `21e410f3-c977-465c-80eb-6890ddd3085c`. Formatting them in file order would give a different,
   wrong GUID.
5. **Submit.** A small mapper put the fact under the exact answer ID and printed JSON, separately from
   the extractor. Single Solve was CORRECT on the first submission.

## Time Trial and Mastery

Nothing changed except scale and packaging. Time Trial was the same parse on `managed-002.dll`
(`9e0e197f-d610-4ec9-bd1c-d28da985893a`). Mastery needed the outer ZIP unpacked to two nested
archives, with each DLL parsed on its own and kept with its set's answer ID. Results were
`managed-003.dll` -> `2658bcc7-f07c-48b1-8cb9-2bb645f8680e` and `managed-004.dll` ->
`ecc37f87-9668-43c5-b9f4-011ee0d07c7c`. Both went in one batch and were CORRECT on the first call, well
inside the window. The journal's point stands: shared metadata labels and build ancestry do not make
MVIDs interchangeable, so each DLL has to be tied to its own nested archive and answer ID.

## Dead ends and gotchas

- **Answer ID per mode.** The first Time Trial submission reused the Single Solve ID
  (`d12-dotnet-001-...`) and returned `incorrect`. This was a mapping slip, not a parsing error. An ID
  the attempt does not contain is judged `incorrect`, not rejected as malformed, so one call was wasted.
  The corrected `d12-dotnet-002-...` was CORRECT (see the day overview for the shared lesson).
- **Lure strings.** `Huntress.Managed.<hex>`, `2.0.2.6` and `--huntress-mvid` look like identifiers but
  are not the Module row's MVID.
- **GUID byte order.** The big trap: a byte-for-byte hex dump of the heap entry is not the canonical
  text. The `#GUID` heap indexes from 1, and a 0 index means nil.

## Takeaways

Technique: **.NET MVID extraction from CLR metadata** (PE data directory 14, `BSJB` root, `#~` Module
row, `#GUID` heap, `uuid.UUID(bytes_le=...)`). It is about 100 lines of stdlib Python and needs no
dnfile or dnSpy. If a question names a metadata field, find the table row that defines it rather than
searching strings. Assert `set(answer_map) == set(answer_ids)` before submitting. The parser was later
promoted to the workspace library and also reproduced all four accepted values in a later review.

## Evidence

Base: `challenges/day12/metadata-forensics/metadata-forensics-day-12-managed-expectations/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/extract_mvid.py`
  (the parser), `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/mvid_facts.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/set1_facts.json`,
  `mastery/artifacts/set2_facts.json`
- Provenance: no `result.json` or `accepted_plan.json` was saved for the original attempts. Each journal
  records our own submission and CORRECT result, and `progress.md` has Solved rows dated 2026-10-20
  (`^| 12 .*managed-expectations`). The later ranked-sweep `ranked-1/result.json` files for Time Trial and
  Mastery read `practice_only` (attempt pool already spent, nothing issued), so they are not graded solves.
- Later library entry `library/metadata-forensics/dotnet-mvid/` (META-DOTNET-MVID-001) and the ranked
  solver `tools/ranked/solvers/metadata-forensics-day-12-managed-expectations/`
