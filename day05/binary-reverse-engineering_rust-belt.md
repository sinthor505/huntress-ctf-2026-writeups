# Rust Belt

| | |
|---|---|
| Day / Lane | 5 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-05-rust-runtime-shapes` |
| Technique | Static recap of five language specimens (C, C++, .NET, Go, Rust): ELF/PE header facts, symbol tables, and XOR-obfuscated `HCFG` config records found by CRC validation |
| Single Solve | Independent (CORRECT on the 3rd submission call; the first was rejected as malformed, the second was `incorrect`) |
| Time Trial | Independent (CORRECT on attempt 3; attempts 1-2 returned `expired`, per the journal and ledger) |
| Mastery | Solved per journal and `progress.md`; the graded response was not preserved in the leaf. The timed attempt was lost and the mode was finished through untimed `practice` |

## Summary

The prompt is about Rust ("mangled symbols with a distinct accent, panic strings that confess, trait objects"),
but only 8 of the 40 answer fields are about the Rust binary. The rest recap the Day 1-4 specimens in the
same evidence ZIP. The work was static header parsing plus decoding one XOR-obfuscated config record per
specimen. Single Solve was solved by hand and cost one wrong symbol-table reading. Time Trial and Mastery
were solved by a one-process pipeline, because the server windows were 50-90 seconds. Mastery's only timed
attempt expired, and it was completed through `practice`.

## The challenge

Each mode delivers an AES-encrypted `evidence.zip` (password `infected`) holding one specimen per earlier day
(file names are prefixed `d01--` to `d05--`):

- d01: C ELF, with a `recover_code` function.
- d02: C++ ELF or PE.
- d03: a .NET DLL plus its `.runtimeconfig.json`.
- d04: Go ELF or PE.
- d05: the Rust ELF.

Answers are 40 string fields per delivery. Typical fields:

- SHA-256 of a member.
- ELF class, machine, type and endianness.
- d01 section names, symbol names and the disassembly of `recover_code`.
- The CLR metadata version string.
- Config values (`campaign`, `domain`, `mutex`, `url`, CRC32, XOR key) decoded from each specimen's `HCFG` record.

Single Solve and Time Trial are one delivery of 40 fields. Mastery is two deliveries (`set1.`, `set2.`) of 40
fields each, 80 in total.

## Solution (Single Solve)

1. Extracted the ZIP with `pyzipper` (AES, not stdlib `zipfile`) and hashed each member. The d03 pair is
   answered in ZIP central-directory order (DLL first, then runtimeconfig).
2. Read header facts without running anything. d01, d02, d04 and d05 were ELF64, little-endian, `ET_EXEC`,
   x86-64. d03 was PE32 with machine `0x14c` (i386) and CLR metadata version `v4.0.30319`.
3. Found each `HCFG` record. The layout is stated in the later prompts: `HCFG`, version byte, one XOR-key byte,
   a little-endian payload length, a little-endian CRC32 of the decoded payload, then the payload from byte 12.
   Keys were `0x31` (d02), `0x41` (d03), `0x51` (d04) and `0x61` (d05). XORing with the key and checking
   `zlib.crc32` gave JSON containing `campaign`, `domain`, `mutex` and (for d02/d04/d05) `url`.
4. The d05 record sits at file offset `0x56c30` (declared length 368). The literal bytes `HCFG` also occur in
   the Rust binary's code and symbol data. Those hits are not records; the real one is the occurrence whose
   bounds, CRC32 and decoded JSON all validate. The later solver notes two CRC-invalid decoys in d05.
5. d01 section names were taken in section-header order. The prompt allows the leading empty string to be
   included or omitted, and we omitted it.
6. Submitted all 40 values as JSON strings, with array answers as compact JSON text.

## Time Trial and Mastery

The question text became much more explicit than in Single Solve. For example, `d01_analysis_symbols` now
says "every nonempty SHT_SYMTAB symbol name in symbol-table entry order ... preserve duplicates", and the
d02/d05 prompts spell out the `.hcfg` section and CRC-validation rule. The fixtures also varied:

- Time Trial d01 was stripped (no `.symtab`). The prompt covers this: `recover_code` is absent, so
  `d01_analysis_disassembly` is `[]`, and the symbol list is `[]`.
- d02 and d04 were delivered as PE32+ rather than ELF. They had to be classified by magic bytes, not by
  assuming the Single Solve layout.
- Mastery returned two deliveries, which `gm.py artifact` cannot fetch together. They had to be downloaded by
  their individual capabilities from `artifacts[]`.

The windows were 50 s (Time Trial) and about 80 s (Mastery). Attempts 1 and 2 of Time Trial expired because
the first pipeline shelled out per file. Attempt 3 imported the inspector in-process, ran the extract,
inspect and map steps in about 1.5 s, and went CORRECT. The same approach was extended to two sets
(`mastery_pipeline.py`).

**Mastery.** The first `start` already reported `unique_attempts_remaining: 0` and an 80 s deadline.
Downloading both bundles and validating the pipeline used up the window, and the submit returned `expired`.
A `retry` then returned 409 `rotation_exhausted`, "You have used every timed attempt for this challenge",
with `next_op: practice`. `practice` issued an untimed attempt (`deadline: null`, not Hall-of-Fame eligible)
over the same two deliveries, whose SHA-256 values matched the saved ZIPs. The offline map (80 string
fields, lint-clean) was submitted once and, according to the journal and `progress.md`, was CORRECT.
No graded response is saved in the leaf.

**Later ranked reruns.** `time-trial/ranked-1`, `ranked-2` and `ranked-4` each went CORRECT in one call with
the general solver (total 2.3-3.9 s from issue to graded result). `ranked-3` failed before issuing a submit
because of a missing dependency in our environment. A ranked Mastery rerun returned `rotation_exhausted`
again, so the practice result is the only Mastery solve.

## Dead ends and gotchas

- **Symbol list in Single Solve.** The first valid-shaped submission was `incorrect`, with no per-field
  feedback. The Single Solve prompt did not say which symbols `d01_analysis_symbols` meant, and the adjacent
  disassembly question pointed at `recover_code`, so the first candidate was `["recover_code"]`. The accepted
  answer was the full `.symtab` name list in table order: `variant-000.c` and the other FILE entries,
  duplicates, `marker`, the CRT/runtime symbols, `recover_code`, `main`. The same d01 bytes had already been
  accepted in an earlier Go Fish Time Trial map, and matching on SHA-256 settled it. The later modes' wording
  confirms this reading.
- **Decoy `HCFG` literals.** Searching for the string `HCFG` alone is not enough in the Rust binary. Only the
  CRC32-valid record counts.
- **Spellings that were accepted.** `little endian` for byte order (not `little-endian`), `I386` for the d03
  machine, and the `PE32_CLI` compound token for d03 format. The d04 architecture is `x86-64` for an ELF and
  `AMD64` for a PE. Pattern fields accepted these equivalents; enum-style fields did not.
- **Strip state changes between modes.** d01 had a full symbol table in Single Solve and none in Time Trial.
  Each delivery has to be read fresh.
- The Single Solve also spent one call on native JSON arrays (`invalid_answers`). Array answers must be
  JSON text inside strings. See the day overview for our own tooling mistakes.

## Takeaways

- Parse the delivered bytes with a library and a single decoder for the `HCFG` record (key, length, CRC32).
  Accept a record only when the CRC and the JSON both validate. This is the "CRC-validated carrier scan"
  technique.
- In timed modes the cost is process start-up, not analysis. Import the inspector once and run everything
  in-process (about 0.1 s per delivery in the final solver).
- Classify PE versus ELF by magic bytes per delivery, and read the "if absent, submit `[]`" rules for stripped
  specimens.
- For Mastery with no timed attempts left, check the `next_op` in the 409. `practice` is an untimed
  continuation that can still complete the mode.
- Do not try to submit while a first, uncertain pipeline is still being validated inside a short window.
  Pre-stage the multi-set pipeline before the first `start`, since the first `start` may be the only timed shot.

## Evidence

Leaf base: `challenges/day05/binary-reverse-engineering/binary-reverse-engineering-day-05-rust-runtime-shapes/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/submission-response.json`
  (`incorrect`), `single-solve/submission-response-2.json` (CORRECT), `single-solve/answers-candidate.json`
- `time-trial/journal.md`, `time-trial/fast_pipeline.py`, `time-trial/ranked-1`, `ranked-2`, `ranked-4`
  (`result.json` CORRECT)
- `mastery/journal.md`, `mastery/mastery_pipeline.py`, `mastery/start_response.json` (expired),
  `mastery/retry_response.json` (`rotation_exhausted`), `mastery/practice_response.json`,
  `mastery/practice_answers.json` (80 fields)
- General solver: `tools/ranked/solvers/binary-reverse-engineering-day-05-rust-runtime-shapes/solve.py`
- Library techniques: language fingerprints (d01-d10), config carriers by language, question-facts reader
