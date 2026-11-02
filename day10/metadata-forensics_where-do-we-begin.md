# Where Do We Begin?

| | |
|---|---|
| Day / Lane | 10 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-10-where-do-we-begin` |
| Technique | PE optional-header parsing (`AddressOfEntryPoint` RVA); answer-width formatting |
| Single Solve | Solved (journal-recorded): the journal records our own submission as CORRECT on attempt 2; no result file was saved |
| Time Trial | Independent: `time-trial/ranked-1/result.json` is CORRECT (the original attempt is journal-recorded CORRECT on attempt 1) |
| Mastery | Independent: `mastery/ranked-1/result.json` is CORRECT (the original attempt is journal-recorded CORRECT on attempt 1) |

## Summary

Each delivery was a ZIP holding one Windows executable, and the question asked for its PE
`AddressOfEntryPoint` as lowercase hex. Reading the field is trivial. The only trap was the
format: the validator rejected the short form `0x65bd0` and accepted the full 32-bit width
`0x00065bd0`. Time Trial and Mastery repeated the same question on fresh specimens, so the solve was
one library script call per attempt.

## The challenge

The prompt reads: "Architecture tells you whether a binary can run; its entry point tells you where
execution begins. Map the start addresses and catch the report that is lying." In practice every
question was `Determine pe.address_of_entry_point_rva_hex for telemetry-agent-NNN.exe.`, with the
answer format "lowercase hexadecimal with a 0x prefix and no explanatory text".

- Single Solve: one field, `metadata-day-10-evidence.zip` with `telemetry-agent-001.exe` (1,257,984
  bytes, PE32 / Intel 80386, 7 sections, Go runtime strings).
- Time Trial: one field for `telemetry-agent-002.exe`, a fresh PE32+ (amd64) specimen. The start
  response carried a real deadline of about 50 s.
- Mastery: an outer ZIP with two nested sets (`metadata-day-10-evidence-01.zip`, `-02.zip`), one
  executable each (`-003`, `-004`), two fields prefixed `set1.` / `set2.`. Real deadline of about 80 s.

The answer IDs start with `d09-pe-NNN-...` even though this is Day 10. The prefix is not a typo and
the IDs must be copied from the live start response.

## Solution

1. Inventoried the ZIP (one member, Deflate) and extracted the executable. `file` reported a PE32
   console executable.
2. Parsed the headers without executing anything. `e_lfanew` at `0x3c` gives the `PE\0\0`
   signature (here at `0x80`). The optional-header magic (`0x10b` for PE32) selects the layout.
   `AddressOfEntryPoint` is the 32-bit value at optional-header offset `+16`, which was file offset
   `0xa8`. The same offset applies to PE32+.
3. Cross-checked the value two ways, a bounds-checked `struct` read and `objdump -p`, which both
   reported `00065bd0`.
4. Kept the RVA distinct from the preferred virtual address. `ImageBase` was `0x400000`, so the
   entry VA would be `0x00465bd0`, which is not what the question asked for.
5. Submitted `0x65bd0` (the value without leading zeros). The validator returned `incorrect`.
   `0x00065bd0` (eight hex digits, the width of the 32-bit field) returned `CORRECT`.

Answer for Single Solve: `d09-pe-001-pe-address-of-entry-point-rva-hex` = `0x00065bd0`.

The library script `pe_header_facts.py` (the workspace's PE header reader) later folded this in. Its
output key for the answer is formatted with a fixed width:

```python
"address_of_entry_point_rva_hex": "0x%08x" % self.entry_rva,
```

## Time Trial and Mastery

Both modes asked the same question on new files, so no new analysis was needed. The only work was
fitting the whole loop (start, download, parse, submit) inside the 50 s and 80 s windows.

- Time Trial: `pe_header_facts.py --member telemetry-agent-002.exe` printed `0x00063a80`, already
  padded, and it was submitted as `{"d09-pe-002-pe-address-of-entry-point-rva-hex": "0x00063a80"}`.
  CORRECT on the first call.
- Mastery: before parsing, the journal checked the first four bytes of both nested members
  (`MZ\x90\x00`) rather than trusting the `.exe` names. The CLAUDE.md notes that Day 10 Mastery sets
  in other lanes mix PE and ELF, so the check was a precaution, and here both members were PE. The
  script gave `0x00063510` for set 1 and `0x00065bd0` for set 2, submitted together. CORRECT on
  the first call. Set 2's value is the same as Single Solve's, so the specimen appears to be the
  same entry point on a different file index; the journal does not comment on this.
  The `progress.md` row describes the Mastery specimens as amd64, arm64 and 386.
- Later, the ranked sweep reran both modes with a scripted solver (`tools/ranked/solvers/`, engine
  shared across Day 6 to 10 metadata challenges). Each was CORRECT in one call with a total pass time
  of about 1.6 s, and the parse step itself took 0.03 s (Time Trial) and 0.07 s (Mastery). These
  `ranked-1/result.json` files are what back the Independent labels above.

## Dead ends and gotchas

- **Padded hex.** The only rejected answer on this challenge was the unpadded `0x65bd0`. The prompt
  says only "lowercase hexadecimal with a 0x prefix", but the validator wanted the full eight digits.
  The workspace notes record the same padded-hex convention for the Mach-O cputype on Day 20.
- **RVA versus VA.** An RVA is not `ImageBase + RVA`; do not add the image base when the field name
  says `rva`.
- **Magic bytes, not file names.** The `.exe` extension proved nothing in other lanes' Day 10
  Mastery sets, so classifying by `MZ` or `\x7fELF` is cheap insurance.
- **Journal contains NUL bytes.** The Single Solve journal has two literal NULs (a raw PE signature
  pasted in), so `grep` treats it as binary; use `rg -a`.
- **Attempt window.** The Time Trial and Mastery windows (about 50 s and 80 s) were short to finish by
  hand, so the solve was done with a one-command script call.

## Takeaways

- For PE metadata fields, validate `e_lfanew`, the `PE\0\0` signature and the optional-header magic
  before applying offsets; use the fixed-width format the field type implies (`0x%08x` for 32-bit
  values).
- Cross-check a header read with a second tool (`objdump -p` or `pefile`). It takes seconds and
  removes doubt before a metered submission.
- When a timed mode repeats a solved question, keep the answer path as one command, and check each
  nested member's magic before parsing.

## Evidence

Leaf base path: `challenges/day10/metadata-forensics/metadata-forensics-day-10-where-do-we-begin/`

- `single-solve/journal.md` (recon, forensics, 2 submissions), `single-solve/start_response.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/ranked-1/result.json`
  (CORRECT, 1 call, 1.65 s total), `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/ranked-1/result.json`
  (CORRECT, 1 call, 1.63 s total), `mastery/ranked-1/accepted_plan.json`
- Solver: `tools/ranked/solvers/metadata-forensics-day-10-where-do-we-begin/solve.py`; header reader:
  `library/metadata-forensics/pe-header-facts/pe_header_facts.py`
- `progress.md` rows for Day 10; `challenges/day10/submission-attempts.md` (Single Solve 2 of 10
  submissions; the Time Trial and Mastery counts there read 0 against the same cap)
