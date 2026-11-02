# Zig Zag

| | |
|---|---|
| Day / Lane | 10 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-10-zig-cross-platform-services` |
| Technique | Static ELF/PE/JAR/.NET parsing, HCFG config record (XOR + CRC32) extraction, cumulative recap fields |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (journal records the original CORRECT; saved `ranked-N/result.json` files also read CORRECT) |
| Mastery | Independent (journal records the original CORRECT; saved `ranked-N/result.json` files also read CORRECT) |

## Summary

Zig Zag is a "cumulative recap" binary challenge: one archive holds ten specimens (d01 to d10), and
the new d10 block is a Zig-built service that exists as both an ELF and a PE. Nothing needed a
disassembler beyond one constant-return function. Every field comes from a bounds-checked static
parser, and the HCFG configuration record is the core of the work. The Single Solve took two
submissions; the first failed on answer-format slips only. Time Trial and Mastery reused
the same mapper, generalized so each specimen is classified by its magic bytes.

## The challenge

The prompt: "Zig promises one binary for every platform. Compare the ELF and PE service descriptors
side by side and prove they're the same program wearing different uniforms."

Single Solve delivered one AES-encrypted `evidence.zip` (password clue `infected`, so `pyzipper`
rather than stdlib `zipfile`) with 11 members: d01 ELF, d02 ELF, d03 .NET DLL plus its
`runtimeconfig.json`, d04 ELF, d05 ELF, d06 and d07 JARs, d08 ELF, d09 ELF, d10 ELF. There were
100 answer fields, ten per group, all with exact or pattern validation. Examples:

- d01 (the Day 1 recap): ELF class, machine, section names, every `.symtab` symbol, the disassembly of
  `recover_code`, type, endianness, entry point, whether a `.symtab` exists.
- d02 to d10: member SHA-256, container format, architecture, and the decoded HCFG config
  (`campaign`, `domain`, `mutex`, `url`, CRC32), plus per-group extras (C++ RTTI predicates for d02,
  CLR metadata version for d03, XOR key and HCFG version, JAR compression-method codes for d06/d07,
  the HCFG offset for d09, "has an `hcfg` section" for d10).

## Solution

1. **Extract.** Decrypt the archive with `pyzipper`, keep members in central-directory order (the
   `artifact_sha256_N` fields are indexed by that order), and group by the `dNN--` prefix.
2. **Classify by bytes, not name.** `\x7fELF`, `MZ` and `PK\x03\x04` select the ELF, PE and JAR parsers.
   d03 is a PE32 CLR assembly, read through its CLI header to the `BSJB` metadata root, which gives
   the version string `v4.0.30319`.
3. **d01.** Parse section headers and `.symtab` directly. The answer is every nonempty symbol name
   in table order (duplicates and FILE entries included). `recover_code` is a constant return:
   `["mov eax,0x1fa4","ret"]`.
4. **The HCFG record.** Each of d02 to d10 embeds a small record: a marker, version, one-byte XOR key,
   payload length and CRC32, then an XOR-encoded JSON payload. The carrier differs per language
   (a literal in an ELF section, the CLR image, a Base64 run in the Swift and Nim binaries, a
   Base64 constant in a JAR class). The solver scans for candidates and keeps only the one whose CRC32
   over the decoded payload matches, which rejects incidental marker bytes:

   ```python
   def valid_record(data):
       records = [r for r in hcfg.scan(data)
                  if r.get("crc32_valid") and isinstance(r.get("decoded_json"), dict)]
       if len(records) != 1:
           raise ValueError(f"expected one valid JSON HCFG record, got {len(records)}")
       return records[0]
   ```

   Requiring exactly one valid record meant a decoy or a parsing slip failed loudly instead of
   submitting a wrong config. d02's record is read from the section named exactly `.hcfg`; d08 tries
   the literal scan first and falls back to Base64 runs; d09's `hcfg_offset` is the character offset of
   the Base64 run containing a record that decodes at offset 0 (30696 in the Single Solve bundle).
5. **d10 (the new block).** The Zig ELF is stripped (no `.symtab`) and carries a dedicated `.hcfg`
   section; the record sat at file offset 132160 and passed its CRC. `has_hcfg_section` is true if any
   parsed section name contains `hcfg`. We identified the language from the `.comment` section (a
   `zig-bootstrap` clang string with LLD) and Zig panic strings; those do not change any answer
   but confirm which specimen is which.
6. **Emit strings only.** JSON-kind answers are compact JSON text, integers are base-10, and booleans
   are lowercase `true`/`false`. The mapper printed all 100 IDs and we asserted its key set equaled the
   start response's `answer_ids`.

The mapper never executed any specimen. d01's disassembly came from `objdump` in the Single Solve; later
modes used capstone instead (same instruction text).

## Time Trial and Mastery

Both modes kept the same 10-group layout, with larger and faster fixtures.

- **Time Trial:** 100 fields, one `evidence.zip`. The Time Trial windows were short (80 s on the
  ranked reruns, and the first attempt of the original session expired while the mapper was being
  adapted). The mapper from Single Solve was reused with its d01-to-d10 field logic unchanged.
- **Mastery:** 300 fields, three deliveries (`set1` to `set3`, one capability token each, answer IDs
  prefixed `setN.`). The ranked windows were 200 s, and each set is a ~31 MB-class static Zig bundle.
  The mapper simply loops the per-set logic.
- **ELF/PE mix.** The fields are fixed by group (d02 always asks the C++ RTTI predicates, d10 always asks
  `has_hcfg_section`), but the container behind them can change. Mastery set3 delivers d02, d04 and
  d10 as PE files (`.exe`) while set1 and set2 deliver ELF. That needed three generalizations:
  d02's C++ typeinfo/vtable checks read Itanium-mangled `ZTI...DiscoveryEngine` and `ZTV...DiscoveryEngine`
  names (Zig on a PE target still keeps them), `has_hcfg_section` reads PE section names as well as ELF
  ones, and PE machine/architecture fields use `0x8664`. The HCFG scan itself is format-agnostic.
- The per-challenge solver kept for later rerun passes answers all of this through the library's
  `question_facts.py` resolver and replays the accepted spellings: `little endian`, `PE32_CLI`,
  `0x014c` for d03, `Advanced Micro Devices X86-64` for ELF machines, `0x8664` for PE ones.
- **Ranked reruns:** later, scripted reruns were also graded CORRECT in one submission each. Total
  time per Time Trial pass was about 6 to 10 s and per Mastery pass about 7 to 16 s as recorded in
  the saved `result.json` files. The server-side rank is not recorded in the leaf.

## Dead ends and gotchas

- **Single Solve submission 1 was `incorrect` because of five formatting slips, not parsing.** Four enum
  predicates were emitted as Python `True` instead of lowercase `true`, and `d03_analysis_format` was the
  bare `PE32` instead of the compound container+runtime token `PE32_CLI`. Submission 2, the same map
  with those corrected, returned CORRECT. The batch gave no per-field feedback, so the cause was found by
  re-reading each field's format string. Both are now caught by a pre-submission linter.
- **Wrapper tokens vary by day.** `PE32_CLI` was accepted here; `PE32_CLR` was accepted on a different
  day. Include the runtime half of the token.
- **Pattern fields accept several spellings.** Here `Advanced Micro Devices X86-64`, `little endian`
  and `0x014c` were accepted; the syntax text of the live question says what is allowed, so there is
  no value in chasing every variant.
- **Mastery attempt 1 consumed an attempt on a 503.** The first Mastery submission got an HTTP 503
  `temporarily_unavailable` and the attempt had also expired, so a retry with fresh bytes was needed.
  A 503 is not a verdict on the answers.
- **Journal inconsistencies, resolved against the later deep review.** The Time Trial journal says
  d02/d04/d10 were PE in the accepted Time Trial bundle, and the Mastery journal says sets 2 and 3
  both had PE members. A later review (2026-10-24) against the saved archives corrected both: the
  accepted Time Trial bundle was all-ELF, and only Mastery set3 mixed in PE (the saved `set1`/`set2`
  extractions are all ELF and `set3` has the three `.exe` files). The Time Trial leaf's own
  `artifacts/` folder still contains `.exe` extractions, which I could not reconcile with the review,
  so I do not state which format any single Time Trial attempt used. The solver does not depend on it.

## Takeaways

- **Static, bounds-checked parsers beat bespoke tricks.** The whole challenge is container parsing
  plus one tiny record format. Check the CRC and require a single valid record.
- **Classify by magic bytes, every set.** A fixed answer schema does not mean a fixed container; a
  mapper hard-coded to ELF fails on the first PE member of a Mastery set. This also applies to the C++
  RTTI check, which has to read the format's own symbol or string source.
- **Lint answers before the first submission.** The two Single Solve failures were pure format:
  lowercase booleans, compound runtime tokens, JSON as text.
- **Recap blocks are cumulative.** The new d10 block is only ten fields; the other 90 are earlier days'
  fields. A resolver library that already answers d01 to d09 makes a new day mostly a d10 exercise.
- Searchable names: HCFG record (XOR + CRC32), `.hcfg` ELF/PE section, Zig `zig-bootstrap` `.comment`
  fingerprint, CLR metadata root (`BSJB`), JAR compression-method codes.

## Evidence

Leaf base path: `challenges/day10/binary-reverse-engineering/binary-reverse-engineering-day-10-zig-cross-platform-services/`

- `single-solve/journal.md` (two submissions, the second CORRECT; no `result.json` was saved, so the
  provenance rests on the journal and the `progress.md` row), `single-solve/artifacts/answer_map.py`,
  `single-solve/start_response.json`, `single-solve/questions.md`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-*/result.json`
  (five CORRECT, 1 call each)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/ranked-*/result.json`
  (four CORRECT, 1 call each)
- `progress.md` rows for Day 10 `Zig Zag`; `challenges/day10/submission-attempts.md`
- Per-challenge solver: `tools/ranked/solvers/binary-reverse-engineering-day-10-zig-cross-platform-services/solve.py`
- Library entries: BRE-HCFG-001, BRE-HCFG-BASE64-001, BRE-JAR-HCFG-001, BRE-PE-CLR-001,
  BRE-LANG-FP-A101, BRE-ANSWER-LINT-A107, BRE-QFACTS-B107
