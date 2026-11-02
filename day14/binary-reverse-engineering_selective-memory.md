# Selective Memory

| | |
|---|---|
| Day / Lane | 14 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-14-objective-c-persistence-descriptor` |
| Technique | cumulative recap bundle: static ELF/PE/JAR parsing plus CRC-validated `HCFG` config-record extraction |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The title and prompt point at Objective-C, but only 10 of the 140 questions concern the Objective-C
specimen. The rest re-ask the earlier days' binaries (Day 1 through Day 13) from a single encrypted
bundle. Every field comes from static parsing of container headers and from one small embedded
config record (`HCFG` magic plus a CRC32) that each specimen hides in a different place. We submitted
a complete offline-derived map once per mode, and each came back CORRECT on the first call (Mastery
after one transient 503 that was not a verdict).

## The challenge

Prompt: "Objective-C never forgets: every class, every selector, every string is registered and
waiting. Read the persistence descriptor and learn what this binary remembers."

The prompt is only loosely related to the questions. Each delivery was one AES-encrypted
`evidence.zip` (password `infected`) with one specimen per earlier day, `d01` to `d14`. The
specimens were ELF for Days 1, 2, 4, 5 and 8-14, a managed PE for Day 3 (plus its
`.runtimeconfig.json` as delivery support), and JARs for Days 6-7. No specimen was executed.

- **Single Solve:** 140 answer IDs, ten per day prefix `d01`-`d14`. Formats: 64 `pattern`, 53
  `exact_text`, 9 `enum`, 9 `integer`, 5 `json`.
- **Time Trial:** same shape on a fresh archive, with a clocked 80 s window.
- **Mastery:** several archives in one batch, 140 IDs per set (`setN.` prefix). Our first attempt had
  3 sets (420 IDs); the later ranked reruns had 5 (700 IDs).

The questions ask for facts a reverser would pull from a header or string table: SHA-256 of a member,
ELF class/type/machine/endianness/entry point, section names, `.symtab` names in table order, the
disassembly of one tiny function (Day 1), the PE CLR metadata version string (Day 3), ZIP
compression methods (Days 6-7), and for most days the decoded config fields (`campaign`, `domain`,
`mutex`, `url`), the record's CRC32 and an `hcfg_offset`.

## Solution

1. **Extract.** Extracted the outer archive with an AES-capable ZIP reader and recorded each
   member's SHA-256 in central-directory order (the questions say "member 1 of the `dNN--` members
   in ZIP central-directory order", so order matters). Checked the archive hash against the start
   response first.
2. **Triage by specimen type.** Parsed ELF headers, section tables and `.symtab` with a bounded
   reader. For Day 3, mapped the PE CLI data directory RVA to a section, confirmed the `BSJB`
   metadata root, and read the version string using its declared length. For Days 6-7, pulled the
   `Main-Class` constant pool from the JAR classfile.
3. **Find the config record per specimen.** The one `HCFG` record is stored differently per day:
   - Days 2, 4, 5, 10, 12 and 14: a literal record in the ELF bytes. Day 2's begins at byte zero of
     the `.hcfg` section.
   - Days 6-7: a CRC-valid JSON record found in the `Main-Class` constant pool.
   - Days 8, 9, 11, 13: a Base64 run whose decoded bytes begin with a CRC-valid JSON record. Literal
     `HCFG` hits elsewhere in those files are decoys; only the CRC-valid one counts.
4. **The Objective-C block (d14).** The specimen is a GNUstep/libobjc ELF (names such as
   `objc_msg_lookup_super`, not Apple's runtime), and its record is a literal inside an `.hcfg`
   section. The ten fields are the member hash, architecture, format (an enum, `elf`), machine,
   the four config strings, the CRC32, and `has_hcfg_section`. The solver derived the last one from
   the parsed section table rather than assuming it:

   ```python
   put(answers, "d14_analysis_has_hcfg_section", str(".hcfg" in section_names(elf)).lower())
   ```

5. **Validate before submitting.** The mapper asserted that its key set equalled the response's
   `answer_ids` exactly, and that enum, integer, JSON and hash/hex fields passed local shape checks
   (140 of 140, no errors). Then one full-map submission.

## Time Trial and Mastery

The method was identical; only the bytes changed. The Time Trial archive was fresh, so the mapper
was rerun over its own bytes, never reusing the Single Solve answers. The 80 s window was enough to
re-run the mapper and submit.

Mastery added scale and one trap: **set 2 contained PE variants for Days 2, 4 and 10** where the
other sets had ELF. The mapper therefore had to parse PE section tables, CLI metadata where present,
and COFF symbol names, and classify each member from its own bytes, not from the day number. The
requested CRC32s and carrier offsets came from CRC-valid records in each set's own archive. Our
first Mastery submission (420 fields) returned HTTP 503 `admission_unavailable`; `status` showed the
same attempt and deadline still active, and the same map was resubmitted once and returned CORRECT.

Later ranked reruns (speed-scoreboard only, bragging rights, not scored here) went CORRECT on one call
each for Time Trial (about 9 s end to end, two runs) and Mastery (5 sets, 700 fields, about 15-18
s end to end, two runs), using a per-challenge solver built on the library's shared question reader.

## Dead ends and gotchas

- **The prompt misleads on scope.** Reading "classes, selectors, strings" as a cue to dig into the
  Objective-C runtime would have spent time on 10 fields. The whole block is a recap of Days 1-13,
  and the real work is breadth: one correct reader per container/carrier type.
- **Offsets point at the Base64 text, not the decoded record.** For the Base64-carrier days the
  `hcfg_offset` is the file offset of the first Base64 character of the winning run (the decoded
  record itself would sit at offset 0).
- **`d03` has two files.** The `.dll` is the specimen; the `.runtimeconfig.json` is delivery support.
- **Label spellings.** The accepted map used `PE32_CLR`/`I386` for Day 3 and `elf` for the enum
  `format` fields; pattern fields accept equivalent spellings, enum fields are exact. We did not
  have to search spellings here, since the first full-map submission was accepted in every mode.
- **Mastery 503.** A transient `admission_unavailable` is not an answer verdict: confirm with
  `status` and resubmit the same map (it cost a call, nothing else).
- **Day 1 disassembly.** The Day 1 `recover_code` function is a constant-return stub; the answer is
  its instruction strings without addresses or byte columns. The original solve used `objdump`; the
  later ranked solver ran without one.

## Takeaways

- Treat a "cumulative recap" BRE bundle as a table of per-container readers plus a per-specimen
  config-carrier rule (literal / `.hcfg` section / Base64 run / JAR constant pool), and validate every
  candidate record by its CRC32 so decoy `HCFG` strings are rejected automatically.
- Classify members from magic bytes per delivery (Mastery mixes PE and ELF in the same day slot).
- Assert `set(answer_keys) == set(answer_ids)` before submitting and submit the complete map once;
  all three modes passed on their first verdict-bearing call.
- Searchable names: `HCFG` record, CRC32-validated config carrier, GNUstep libobjc ELF fingerprint,
  cumulative recap bundle.

## Evidence

Leaf: `challenges/day14/binary-reverse-engineering/binary-reverse-engineering-day-14-objective-c-persistence-descriptor/`

- Single Solve: `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-attempt-log.json`
  (attempt 1, 140 fields, CORRECT; no `result.json` was saved, hence "journal-recorded").
- Time Trial: `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json`
  (attempt 1 CORRECT), `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`.
- Mastery: `mastery/journal.md`, `mastery/artifacts/submission-attempt-log.json` (503, then CORRECT),
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`.
- Ranked solver (later reruns): `tools/ranked/solvers/binary-reverse-engineering-day-14-objective-c-persistence-descriptor/solve.py`.
- Local ledger: `progress.md` rows for Day 14 (all three modes marked Solved, 2026-10-20).
