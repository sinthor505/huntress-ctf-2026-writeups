# Mangled

| | |
|---|---|
| Day / Lane | 2 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-02-cpp-and-pe-orientation` |
| Technique | Static triage of a C++ ELF/PE: symbol-table queries (Itanium-mangled names, vtable/RTTI symbols) and an XOR+CRC32 embedded config record |
| Single Solve | Solved, graded result not preserved (the journal records CORRECT on submission 1; no result file in the leaf) |
| Time Trial | Independent (three ranked attempts, each `result.json` CORRECT) |
| Mastery | Solved, graded result not preserved (the journal records CORRECT on submission 2; no result file in the leaf) |

## Summary

The prompt is about demangling C++ symbols and reading vtables and RTTI, but the graded questions
only ask for 17 plain facts about two supplied binaries. The key insight was that no real
reverse engineering is needed: one script that reads the ELF/PE symbol tables and one embedded
XOR-obfuscated config record answers everything. The same mapper carried across Single Solve,
Time Trial and Mastery. The only real trap was that one Mastery set swapped the C++ ELF for a
MinGW PE, which changes how symbol names and the container format must be spelled.

## The challenge

Prompt: "C++ on Windows. The symbols are mangled, the classes hide behind vtables, and RTTI is your
Rosetta Stone. Demangle it all with `c++filt`, reconstruct the classes, and don't forget the PE
imports."

Delivery was a small AES-encrypted ZIP (password `infected`, about 5.5 KB) holding two
binaries: `day01-c.elf` (a carry-over from Day 1, a C ELF) and `day02-cpp.elf` (the C++ specimen).
The answer set was 17 string fields:

- From the C binary: `c_sha256`, `c_marker`, `c_has_symtab`, `c_recover_code` (base-10 integer),
  `c_machine`.
- From the C++ binary: `cpp_sha256`, `cpp_format`, `cpp_xor_key`, `cpp_config_crc32`, the four
  config strings `campaign`, `domain`, `url`, `mutex`, the booleans `has_cpp_vtable` and
  `has_cpp_typeinfo`, and two symbol names, `mangled_score_symbol` and `vtable_symbol`.

So the first five fields are a Day 1 recap (the "orientation" half of the ID), and the rest are
the C++ half. Neither half asks for control-flow analysis.

## Solution

1. **Unpack and verify.** The ZIP is WinZip AES, so stdlib `zipfile` is not enough. We
   downloaded the artifact fresh and checked its SHA-256 against the start response. An older
   local copy of the bundle had been damaged by LF to CRLF translation and would have answered
   the wrong bytes.
2. **C binary fields.** Hash the file, read the ELF section table (`.symtab` present, so
   `c_has_symtab` is `true`), and take the machine from the header (`x86-64`). The marker
   string and `c_recover_code` came from the Day 1 readers (`recover_code` is a tiny
   `mov eax, imm32; ret` function, so its return value is the immediate).
3. **C++ config record.** `day02-cpp.elf` carries a dedicated `.hcfg` section. A record there is
   a short header (version, one-byte XOR key, little-endian payload length, little-endian CRC32)
   followed by an XOR-encoded JSON payload. We XOR exactly the declared payload length, check
   `zlib.crc32` against the stored value, and only then trust the JSON. The key and CRC are the
   `cpp_xor_key` / `cpp_config_crc32` answers (lowercase hex with `0x`), and the JSON gives
   `campaign`, `domain`, `url` and `mutex`. The extractor insists on exactly one CRC-valid record
   so incidental marker bytes cannot be mistaken for it.
4. **Vtable and RTTI facts.** The binary was not stripped, so this reduced to regexes over the
   `.symtab` names. This is the `c++filt` step done by pattern instead of by eye:

   ```python
   hits = lambda rx: [n for n in psyms if re.search(rx, n)]
   "has_cpp_vtable":       str(bool(hits(r"^_ZTV"))).lower(),
   "has_cpp_typeinfo":     str(bool(hits(r"^_ZTI"))).lower(),
   "mangled_score_symbol": hits(r"^_ZNK.*5score")[0],
   "vtable_symbol":        hits(r"^_ZTV\d+DiscoveryEngine$")[0],
   ```

   `_ZTV` is the Itanium ABI prefix for a vtable and `_ZTI` for type_info. In the Single Solve
   specimen this gave `_ZNK15DiscoveryEngine5scoreEPKc` (the const method `DiscoveryEngine::score(char const*)`)
   and `_ZTV15DiscoveryEngine`. The specimen also had a `BehaviorEngine` class, so the vtable
   regex is anchored on the class name rather than taking the first `_ZTV`.
5. **Submit.** One call with all 17 fields, every value a JSON string. For the Single Solve the
   answers included `c_marker` = `BR01-CATALOG-000`, `c_recover_code` = `8100`,
   `cpp_xor_key` = `0x31`, `cpp_config_crc32` = `0x6149aa00`,
   `campaign` = `BR02-000`, `mutex` = `Global\HuntressBR02_000`. The Game Master returned
   CORRECT on the first submission.

## Time Trial and Mastery

**Time Trial.** Layout was identical (C ELF plus C++ ELF) with a fresh HCFG record, so the Single
Solve mapper was reused unchanged. The "untimed" start still returned a roughly 50 s server
deadline, so the flow was scripted end to end: download, extract, map, submit in one pass. It
went CORRECT on call 1 (17/17). In a later ranked sweep we ran the same flow three more times
through a scripted solver; all three were CORRECT on one submission call each, in 1.9 s, 1.9 s
and 2.9 s of total local time (window 50 s).

**Mastery.** 34 answers: `set1.*` and `set2.*`, 17 each, with an effective window of about 80 s
(again despite an untimed start). Only the first delivery can be fetched through the generic
artifact call, so both deliveries were downloaded from the per-delivery download links and hash
checked.

- `set2` matched the Single Solve layout (C ELF plus C++ ELF). The ELF mapper with a `set2.`
  prefix was accepted for all 17 ids on the first submission.
- `set1` replaced the C++ ELF with `day02-cpp.exe`, a MinGW PE32+ with a COFF symbol table and
  an `.hcfg` section. This needed a separate mapper: parse the COFF symbol table and string
  table by hand, `cpp_format` becomes `PE32+`, and the symbol answers keep their COFF section
  prefix (for example `.text$_ZNK...` and `.rdata$_ZTV...`). `c_machine` stays `x86-64`
  because it describes the C binary, which is still an ELF in that set. Submitting it as the
  second call completed the attempt as CORRECT.

Splitting the submission by `setN.` prefix is what made the ELF/PE problem easy to isolate,
because the validator gives no per-field feedback on a failing batch.

## Dead ends and gotchas

- **Prompt wording is flavor.** "Demangle it all", "reconstruct the classes" and "PE imports"
  do not correspond to any asked field. The questions are generic ("the exact c sha256
  value for field ...") and only the field names carry meaning.
- **PE variants change the symbol spelling.** On the MinGW PE, the first regex hit in raw
  COFF order carries a section prefix, and that prefixed form was the one accepted. We did not
  test the bare (ELF-style) name against the validator for this PE, so we cannot say whether it
  would have been rejected.
- **Format spelling.** The `cpp_format` pattern accepts `ELF`/`ELF64`/`PE32`/`PE32+`/`Portable
  Executable` when they name the same container; we submitted `ELF64` and `PE32+`. `c_machine`
  is plain `exact_text`; `x86-64` was accepted.
- **Damaged local copies.** A saved `bundle.zip` had LF to CRLF damage (the undo reproduces the
  API hash). Always check the hash and the `PK` magic before analysis.
- **Short windows on "untimed" starts.** The Time Trial and Mastery starts were issued without
  the timed flag and still carried 50 s and 80 s deadlines. On these legacy early-day routes a plain
  retry is the ranked attempt, so there was no free rehearsal; the mapper was proven on the Single
  Solve copy first.

## Takeaways

- **Read the question list before the prompt.** On this lane the story text and the asked
  fields can be unrelated; the fields (hash, header facts, config record, symbol regexes)
  define the real work.
- **Embedded config record (HCFG: XOR key, length, CRC32).** Locate it by section or marker, decode
  only the declared length, and accept it only if the CRC verifies.
- **Classify every delivered member by magic bytes (`\x7fELF` vs `MZ`), not by the expected
  file name.** Mastery bundles mix ELF and PE variants of the same logical specimen.
- **Prove the mapper offline on the Single Solve, then run download, map, submit in one pass**
  for the short Time Trial and Mastery windows. Submit Mastery by set prefix.

## Evidence

Leaf base path: `challenges/day02/binary-reverse-engineering/binary-reverse-engineering-day-02-cpp-and-pe-orientation/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/answers_candidate.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`
- `time-trial/ranked-1/result.json`, `ranked-2/result.json`, `ranked-3/result.json` (each CORRECT, 1 call;
  also `accepted_plan.json` and `pass.log` per attempt)
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map_elf.py`,
  `mastery/artifacts/answer_map_pe.py`, `mastery/artifacts/answers_set1.json`, `answers_set2.json`
- No `result.json` is stored for the Single Solve or Mastery; their CORRECT results come from
  the journals and the `progress.md` rows (2026-10-20 re-solve, 1 and 2 submissions respectively),
  which also note an earlier "player-directed local mark" basis.
- Library techniques referenced: HCFG record scanning, ELF structure/symbol inventory, PE/COFF
  symbol tables, and the Days 1-10 cumulative field blocks.
