# Decoder Ring

| | |
|---|---|
| Day / Lane | 17 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-17-encoded-configuration-recovery` |
| Technique | XOR-obfuscated HCFG config records (CRC32-validated) across a cumulative recap bundle of ELF, PE, JAR and WebAssembly specimens |
| Single Solve | Solved (journal-recorded; attempt log reads CORRECT, no saved result file) |
| Time Trial | Independent (timed rerun `ranked-1/result.json` CORRECT; the original untimed solve is journal-recorded) |
| Mastery | Independent (timed rerun `ranked-1/result.json` CORRECT; the original untimed solve is journal-recorded) |

## Summary

Decoder Ring is the Day 17 binary challenge. It is a "recap" bundle: one encrypted ZIP holding every
earlier binary specimen (d01 to d16) plus the new d17 ELF, with 12 questions per specimen. The new
material was a WebAssembly module (d16) and an ELF whose XOR-encoded configuration record sits in
`.rodata` (d17). Because the validator-accepted conventions from the Day 15 bundle carried over, one
offline extractor and one answer mapper produced all 204 answers, and the complete map was CORRECT on
the first submission in every mode (Mastery: 1,020 answers across five deliveries).

## The challenge

Prompt: "The configuration is encoded, but the decoder ships with the binary — a ring waiting for its
secret. Recover the reversible decoder and read what it was hiding."

- Delivery: an AES-encrypted ZIP (password `infected`), about 32.8 MB, 20 members. d03 has a DLL plus a
  `runtimeconfig.json`, d15 has a symbolized/stripped pair, d16 has a "direct" and a "compact" WASM.
- Answers: 204 string fields (17 specimens x 12). Formats were mixed: `pattern` (hashes, hex values,
  architecture labels), `enum` (container format), `json` (arrays of section names, symbols, disassembly),
  `integer` (HCFG offset and version) and `exact_text` (decoded config strings).
- The d17 block asked for the SHA-256 of the d17 member, architecture, machine, container format, the
  decoded `campaign`, `domain`, `mutex` and `url` strings, the CRC32 and XOR key in the header, the
  HCFG version, and the record's file offset.

The question text spells out the record layout, so no decoder had to be reverse engineered from
machine code. An HCFG record is: bytes 0..3 ASCII `HCFG`, byte 4 version, byte 5 XOR key, bytes 6..7
little-endian payload length, bytes 8..11 little-endian CRC32 of the decoded payload, payload from byte
12 XORed with the one-byte key, then parsed as UTF-8 JSON.

## Solution

1. **Recon.** The archive's SHA-256 matched the Game Master's value, and the 20 members were listed from
   the ZIP central directory. The d01 to d15 members were byte-identical (same SHA-256s) to the Day 15
   bundle, so only d16 and d17 were new.
2. **Extract.** The AES ZIP needs `pyzipper`; a helper extracted it (the one step run outside Python's
   isolated mode, which hides user-site packages). Everything else was static parsing. Nothing was
   executed.
3. **Facts extractor.** One script (`extract_facts.py`) read ELF and PE headers (sections, `SHT_SYMTAB`
   names, entry point, `.huntress` marker, the `recover_code` bytes), the JAR classfile table, and the
   WASM section stream, and scanned every specimen for HCFG records with the library scanner. The scanner
   finds each literal `HCFG` magic, XORs the payload, checks the CRC32 and JSON, and also decodes long
   Base64 runs and rescans them.
4. **The d17 record.** The ELF had a `.symtab` with 2,353 entries and exactly one valid literal record at
   file offset 207432, in `.rodata`: version 1, key `0x73`, CRC32 `0x265cc435`, 367 payload bytes. It
   decoded to JSON with `campaign` `BR17-000`, `domain` `node-000.day17.example.invalid`, `mutex`
   `Global\HuntressBR17_000` and a matching `url`. Architecture and machine were `x86-64`, format `elf`.
5. **d16 (WebAssembly).** The scanner found the same kind of record inside the module's data section
   (section id 11), at offsets 125 (direct) and 116 (compact) in the two variants. The module's section
   stream was parsed only to confirm the record sits in data bytes.
6. **Mapper.** `answer_map.py` turned the facts into 204 string values and asserted that its key set
   equalled the start response's `answer_ids`. Arrays were emitted as compact JSON text and integers as
   base-10 strings.
7. **Submit.** One call with all 204 fields returned CORRECT.

The conventions that carried over from Day 15 and were accepted here:

- ELF container `ELF64`, architecture/machine `x86-64`, `e_type` `ET_EXEC`, endianness `little-endian`.
- Native PE: container `PE32+`, architecture/machine `AMD64`; managed DLL `PE32_CLR`, machine `I386`.
- Symbol arrays: every nonempty `SHT_SYMTAB` name in table order, including FILE entries and duplicates.
- Enum "format" questions use lowercase tokens (`elf`, `wasm`).

## Time Trial and Mastery

**Time Trial** had identical question IDs, wording and formats (diffed against Single Solve) but fresh
specimen variants: for example d01 `variant-014` was stripped (no `.symtab`), and d03, d04, d05, d06,
d07, d09 to d15 and d17 were other `variant-NNN` builds. The same extractor and mapper, unchanged,
produced a CORRECT first submission. A stripped ELF takes `[]` for symbols and disassembly and the
literal `none` for `recover_code`, as the question text states.

**Mastery** delivered five archives under one attempt (1,020 fields, IDs prefixed `set1.` to `set5.`).
The Game Master artifact call returns only the first, so the other four were fetched from the start
response's `artifacts[]` list with their SHA-256 values checked. Differences that mattered:

- Sets 1, 3 and 5 carried native PE (x86-64 COFF) members for d02 (MinGW C++), d04 (Go) and d10 (Zig);
  sets 2 and 4 were ELF.
- d01 was the stripped `variant-006` in every set.
- The extractor gained a native-PE reader: section names of the form `/<n>` are resolved through the COFF
  string table, and primary COFF symbols are listed with auxiliary records skipped. d02 PE symbol answers
  keep their section prefix (`.text$_ZNK...`, `.rdata$_ZTI...`) because the question asks for it.
- The mapper took a `setN.` prefix argument and the five per-set maps were merged.

The first Mastery submission, all 1,020 fields, was CORRECT.

Both Time Trial and Mastery were first solved untimed (no deadline in the start response), so those
attempts carry no speed ranking. Later, with the player's authorization, one timed attempt of each was
run through a scripted pipeline built on the same offline logic: Time Trial in an 80 s window
(7.05 s total, CORRECT on one call, scoreboard rank 15 at 4,598 ms) and Mastery in a 325 s window
(20.13 s total, CORRECT on one call). The ranked runs are bragging-rights only.

## Dead ends and gotchas

There were no rejected submissions: every mode was CORRECT on its first call. The notes below are traps
the data contained that the mapper handled, and one that was never validated.

- **Stripped d01 in Time Trial and Mastery.** Variant 014 (Time Trial) and variant 006 (Mastery) have no
  `.symtab`. The mapper has to branch on that rather than assume the Single Solve layout.
- **Mixed PE and ELF in Mastery.** Classify by magic bytes, not by member extension or by set number
  guesses. PE section names beginning `/N` are not the real names until the COFF string table is read.
- **HCFG offset definition.** The offset is the file offset of the literal `HCFG` magic, or, for a Base64
  carrier, the offset of the first Base64 character of the winning run. d17 is a raw carrier, so its
  offset was simply 207432.
- **Storage-token rule (earlier-day fields).** The recap blocks ask how each record is stored. The order
  used was `base64` first, then `raw_hcfg_section` (record begins in a section named `.hcfg`), then
  `raw_const_array` (another mapped section plus an exact reference to the record's virtual address,
  such as a `movabs rax, <vaddr>` in `.text`), then `raw`. The d12 record in `.data` was classified
  `raw_const_array` this way. The reference scan treats any exact address match as evidence, which can
  false-positive in tiny sections; the d12 hit was checked against the disassembly.
- **Unvalidated `raw` fall-through.** d17's record sits in `.rodata` with no code reference, but d17 was
  only asked for its offset, so the `raw` classification for that kind of record was never tested by a
  question here.
- **Day 15 builds.** The d01 to d15 members matched the Day 15 bundle byte for byte, which is why the
  Day 15 spellings were safe to reuse; that held in Single Solve, and the fresh Time Trial and Mastery
  variants kept the same layouts.

## Takeaways

- **Technique names:** XOR-obfuscated config record with a CRC32 check (HCFG); static carving by
  header-driven scan rather than finding the decoder in code; cumulative recap bundles.
- When the prompt says "recover the decoder", read the question text first: here it gave the entire
  record layout, and a scan-XOR-CRC32-JSON loop was the whole decoder. Reversing the binary's own
  routine would have been slower and added nothing.
- A cumulative bundle rewards one parameterized, manifest-driven extractor. Identical members across
  days mean accepted spellings are reusable, but only for members whose SHA-256 is unchanged; re-check
  every variant (stripped, PE instead of ELF) when Time Trial and Mastery swap builds.
- Validate every Mastery delivery separately: the Game Master's artifact call returns only the first.
- Assert that the mapper's key set equals the start response's `answer_ids` before submitting.

## Evidence

Leaf: `challenges/day17/binary-reverse-engineering/binary-reverse-engineering-day-17-encoded-configuration-recovery/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/extract_facts.py`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/candidate-answers.json`,
  `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/fetch_deliveries.py`, `mastery/artifacts/answer_map.py`,
  `mastery/ranked-1/result.json`
- No saved `result.json` exists for the untimed Single Solve, Time Trial or Mastery submissions; their
  CORRECT results are recorded in each `journal.md`, each `artifacts/submission-attempt-log.json` and in `progress.md` (grep `^| 17 .*encoded-configuration-recovery`).
  The timed Time Trial and Mastery attempts are backed by `ranked-1/result.json`.
- Repo-wide: `tools/ranked/solvers/binary-reverse-engineering-day-17-encoded-configuration-recovery/solve.py`
  (the timed-attempt solver, a thin wrapper over the shared Day 15 engine);
  library entries `BRE-RECAP-001` (recap bundle d01-d17), `BRE-HCFG-001`, `BRE-HCFG-BASE64-001`,
  `BRE-JAR-HCFG-001`, `BRE-HCFG-STORAGE-001`, `BRE-WASM-HCFG-001`.
