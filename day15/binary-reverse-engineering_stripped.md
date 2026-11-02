# Stripped

| | |
|---|---|
| Day / Lane | 15 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-15-assembly-and-symbol-loss` |
| Technique | cumulative recap bundle: static ELF/PE facts, `.symtab` presence, HCFG config records |
| Single Solve | Solved (journal-recorded): journal and attempt log record CORRECT on call 1; no graded result file was saved |
| Time Trial | Independent (own CORRECT `result.json` under six `ranked-N/` folders; the original 2026-10-20 attempt is journal-recorded) |
| Mastery | Independent (own CORRECT `result.json` under `ranked-N/` folders; the original 2026-10-20 attempt is journal-recorded) |

## Summary

The prompt promises raw assembly and calling conventions ("the stack never lies"), but no question asks
for a reconstructed function. The day's new specimen, `d15`, is a pair of C ELFs, one symbolized and one
stripped of `.symtab`, and the questions on it are static metadata: SHA-256, `e_machine`, container
format, and two strings from an embedded config record. The other 140 fields (Single Solve) replay the
earlier days' specimens, `d01` to `d14`. All three modes were answered with one offline mapper and went
CORRECT on the first submission.

## The challenge

Prompt: "No symbols. No names. Just raw assembly and the calling conventions it can't hide. Reconstruct
the stripped functions from the instructions alone."

Delivery was an AES-encrypted ZIP (password `infected`; Single Solve 32.3 MB, 17 members). The members
are named `dNN--variant-NNN.<ext>` for the specimens `d01` to `d15`. `d03` ships a .NET DLL plus its
`runtimeconfig.json`, and `d06`/`d07` are JARs. `d15` holds `...-symbolized.elf` and `...-stripped.elf`.

| Mode | Fields | Deliveries | Window |
|---|---|---|---|
| Single Solve | 150 (10 per specimen) | 1 | untimed |
| Time Trial | 150 | 1 | 80 s |
| Mastery | 750 (`setN.` prefix) | 5 | 320 s |

Formats were mostly `pattern` (66), `exact_text` (55) and `enum` (13), plus `integer` and `json`. The
Time Trial counts are 66 pattern / 55 exact_text / 13 enum / 11 integer / 5 json. Every value was
submitted as a string.

The `d15` block is ten fields: `artifact_sha256_1/2`, and for each of the stripped and symbolized members
an `architecture`, `format` and `machine`, plus `config_campaign` and `config_domain` for the stripped
member.

## Solution

1. **Extract and inventory.** Decrypt the archive with `pyzipper`, keep central-directory order (the
   questions index members "1-based in ZIP central-directory order") and hash every member. Nothing was
   executed.
2. **Pick `d15` members by suffix.** The questions say "the unique member whose basename ends
   `stripped.elf`" or `symbolized.elf`. The mapper keys on the stem suffix and fails if it is not exactly
   `{symbolized, stripped}`.
3. **Read container facts from the bytes.** Both members are ELF64, little-endian, `ET_EXEC`,
   `e_machine` 62 (x86-64). The stripped one lacks `.symtab`, the symbolized one has it (and names `main`,
   167 bytes at `0x401146`). Both keep `.text`, `.rodata`, `.dynsym` and `.hcfg`. The mapper submitted
   `x86-64`, `elf`, and `62` (decimal) for each.
4. **Config from the stripped member.** Scan for literal `HCFG` magic and take the first record whose
   bounds, CRC32 and decoded JSON all validate. Single Solve gave `campaign` `BR15-000` and `domain`
   `node-000.day15.example.invalid`. The record's payload is XOR-masked and CRC-checked
   (CRC32 `0xc348914e` in Single Solve).
5. **The other 140 fields** come from the same mapper's `d01` to `d14` blocks. `d01` reads the C ELF's
   section names, complete `.symtab` name list in table order, and the `recover_code` stub disassembly
   (`["mov eax,0x1fa4","ret"]`). `d02` to `d14` read HCFG records from ELF/PE/JAR/.NET members, with
   storage- and offset-style fields where asked.
6. **Guard against ID drift.** The mapper asserts its key set equals the start response's `answer_ids`
   (with an optional `setN.` prefix) and emits compact JSON for arrays. The 150-field map was submitted in
   one call and accepted.

The stripped/symbolized pair never needed a disassembler. The only place instruction text appears is
`d01`'s `recover_code` stub, which is a symbol on a symbolized binary.

## Time Trial and Mastery

- **Time Trial** is a fresh archive with other `variant-NNN` builds and the same layout (campaign
  `BR15-005` in the saved copy). The Single Solve mapper ran unchanged and was accepted on call 1 inside
  the 80 s window.
- **Mastery** delivers five archives, one per set, answered as `set1.` to `set5.`. The mapper is run once
  per delivery with `ANSWER_PREFIX=setN.` and the maps are merged into 750 answers (accepted in one call).
  Sets differ in which specimens are PE vs ELF. In the saved bundle, sets 1, 3 and 5 carried PE members
  for `d02`, `d04` and `d10` (`PE32+`, machine `AMD64`) and sets 2 and 4 carried ELF (`ELF64`, `x86-64`),
  so format and architecture have to be read from magic bytes per member, not assumed from the day.
  `d15` member names and config (`BR15-000` to `BR15-004`) vary per set, so the set's own members are used.
- The `tools/gm.py artifact` command only fetched the first Mastery archive. The journals record adding a
  `--from-start` option to download every entry in `artifacts[]` and verify each SHA-256.
- Both Time Trial and Mastery were later re-run ranked for speed (Time Trial six more attempts, local
  total 4.9 to 10.6 s; Mastery four, about 17 to 21 s), each CORRECT on call 1. These ranked reruns do
  not change the solve. A later Mastery rerun failed in its scripted pass because the pass ran under a
  Python mode that hid `pyzipper` and was recovered against the live attempt (see the day overview).

## Dead ends and gotchas

- **The title and prompt point at the wrong work.** "Reconstruct the stripped functions from the
  instructions alone" suggests decompilation. The graded fields are file-format facts and config strings,
  so the cheap move is the library's recap-bundle mapper, not a disassembly session.
- **Spelling fixed per day.** The accepted forms for this day's pattern fields were `little-endian` for
  `d01` endianness, `I386` and `PE32_CLR` for the `d03` .NET DLL, `AMD64` for PE machine/architecture labels,
  and `elf` for the `d09`/`d11` container field (the engine emits `ELF64` otherwise, and the solver
  respells it). The questions' own syntax says pattern fields accept equivalent spellings, so these are
  the values that were accepted, not the only possible ones. Enum fields (`true`/`false`, `elf`) are exact.
- **`machine` is decimal.** `d15` `*_machine` is an `integer` field (`62`), not a label.
- **Symbol list is the whole `.symtab`.** `d01_analysis_symbols` is every nonempty `.symtab` name in table
  order, including FILE entries and duplicates (as learned on Day 6). It was correct on call 1 here.
- **Do not carry values between modes.** Variants change offsets, keys, JAR contents and `d15` config, so
  the mapper is rerun on each delivery.

## Takeaways

Technique: **cumulative recap bundle** answered by a single deterministic mapper over the archive, with
specimens chosen by name suffix or magic, HCFG records accepted only when bounds, CRC32 and JSON all
validate, and a hard assert that the mapper's key set equals the live `answer_ids`. A pair of
symbolized and stripped builds is a prompt for `.symtab` presence and container facts, not necessarily for
assembly. Read the question list before choosing a methodology. The following days reused this mapper by
adding only the new day's block.

## Evidence

Leaf base path: `challenges/day15/binary-reverse-engineering/binary-reverse-engineering-day-15-assembly-and-symbol-loss/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/candidate-answers.json`, `single-solve/artifacts/submission-attempt-log.json`
  (one attempt, 150 ids, CORRECT)
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json` to
  `ranked-7/result.json` (those present)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/artifacts/set1/` to `set5/`
  (per-set candidate maps), `mastery/ranked-1/result.json`, `ranked-3/result.json`, `ranked-4/result.json`,
  `ranked-2-recover/result.json`, `ranked-4-recover/result.json`
- `tools/ranked/solvers/binary-reverse-engineering-day-15-assembly-and-symbol-loss/solve.py` (later
  speed-run solver built on the shared `question_facts.py` engine)
- Library: cumulative recap bundles (BRE-RECAP-B105), HCFG records (BRE-HCFG-001), ELF structure
  (BRE-ELF-STRUCT-001), ELF symbol inventory (BRE-TECHNIQUE-001), AES ZIP extraction (SHARED-AESZIP-001)
- `progress.md` (grep `^| 15 .*symbol-loss`)
