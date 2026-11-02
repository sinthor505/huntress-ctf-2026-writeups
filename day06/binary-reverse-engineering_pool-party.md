# Pool Party

| | |
|---|---|
| Day / Lane | 6 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-06-java-bytecode-loader` |
| Technique | Cumulative multi-specimen recap (ELF, PE/CLR, JAR); Java class-file constant pool holding a Base64 HCFG config |
| Single Solve | Independent (24 submissions; graded CORRECT response preserved, no `result.json`) |
| Time Trial | Independent (first run CORRECT in 1 submission; two later re-runs also CORRECT, `ranked-1`, `ranked-2`) |
| Mastery | Independent (CORRECT in 1 submission; graded response preserved, no `result.json`) |

## Summary

The title and prompt point at Java class-file constant pools, but the 48 answer fields cover the
Day 1-6 specimens, and only the last six are about the JAR. The Java part was easy: the config sits
as a Base64 string constant in the main class. The expensive part was one underspecified Day 1
field, `d01_analysis_symbols`, which cost 24 submissions in Single Solve. Time Trial and Mastery
were then solved by one offline script inside very short timers.

## The challenge

Prompt: "Every Java class file carries its own phone book: the constant pool. Names, methods,
invocations - it's all in there. Dive in and take attendance."

Single Solve delivered one AES-encrypted `evidence.zip` (password `infected`) with seven members:
`d01`, `d02`, `d04` and `d05` ELF64 x86-64 binaries, a `d03` .NET 8 PE32 DLL plus its
`runtimeconfig.json`, and a `d06` JAR (two classes: `Day06Variant000` and a `Result` record). There
were 48 answer fields, all derived from those members:

- Day 1 (8 fields): artifact SHA-256, ELF class, machine, section names, symbols, disassembly of
  `recover_code`, ELF type, endianness.
- Days 2-5 (6-8 fields each): SHA-256, container format or architecture, and for most the decoded
  HCFG config values (campaign, domain, mutex, URL, CRC32, XOR key). Day 3 asks for PE machine and
  CLR metadata version instead of a URL.
- Day 6 (8 fields): SHA-256, format (`JAR`), campaign, domain, mutex, URL, CRC32 and HCFG version.

Answer kinds were mostly `exact_text`, plus pattern fields that accept several standard spellings
(machine, endianness, format), two JSON-array fields and one integer. Time Trial reused the same 48
IDs with a different assigned archive. Mastery was two deliveries (96 fields, IDs prefixed `set1.` and
`set2.`).

## Solution

1. Verified the archive SHA-256 against the start response, then opened the AES ZIP with `pyzipper`
   (stdlib `zipfile` cannot read WinZip AES) and hashed every member.
2. Day 1: read the ELF header and section table with `readelf`. `recover_code` is a 6-byte function,
   `mov eax,0x1fa4; ret`, found with `objdump -d`.
3. HCFG records (Days 2-5) were located by scanning for the `HCFG` magic and validating
   candidates: version byte, one-byte XOR key, 16-bit length, CRC32 of the *decoded* JSON. The CRC
   check matters on Day 5, where the first `HCFG` byte sequence is ordinary x86 code and the real
   record is a later data occurrence (file offset 355376).
4. Day 3 (PE32 + CLR): Machine `0x014c`, Optional Header magic PE32, CLR metadata root `BSJB` with
   version string `v4.0.30319`; the container answer is a compound PE32-plus-runtime token.
5. Day 6, the namesake: the JAR's main class carries a 508-character Base64 string constant, and a
   `decodeConfig` method that Base64-decodes it and checks the first four bytes against `"HCFG"`.
   The constant pool shows `java/util/Base64` and the string, so the approach is: regex the class
   bytes for long Base64 runs, decode, and feed the result to the same HCFG validator. It decodes to
   version 1, XOR key `0x71`, 368 bytes, CRC `0x121bc931`, campaign `BR06-000`.
6. Answers were submitted as strings, with arrays as compact JSON text. The Single Solve was
   accepted field group by group (see below) and went CORRECT with flag
   `95ee5ed87b0ac6fa7ce7c899ac7c83f7`.

The `d01_analysis_symbols` field was submitted as every nonempty `.symtab` name in table order,
including `FILE` entries and duplicates (`crt1.o`, `variant-000.c`, `crtstuff.c` twice, ...).

## Time Trial and Mastery

- **Scale.** Time Trial was the same 48 fields over a different archive with a different set of
  variants; Mastery was two such archives (96 fields) under one timer. Variant values (hashes, config
  fields) change, so nothing from Single Solve carries over except the layout.
- **Windows.** The first Time Trial window was about 48 s, and Mastery about 78 s. Both were solved
  by `static_solver.py`, which takes the start response plus the assigned archives, verifies each
  archive hash, extracts offline, validates HCFG CRCs and prints one JSON object keyed by the exact
  answer IDs (`set1.*` to delivery 1, `set2.*` to delivery 2). It was dry-run beforehand against the
  Single Solve archive and matched the accepted Single Solve answers exactly. A driver script did
  start, download, verify, solve and a single batched submit in one pass.
- **Time Trial:** CORRECT on the first and only submission, 48 of 48 fields.
- **Mastery:** CORRECT on the first submission after a parser fix (below), with about 36 s left on
  the server clock. That start used up the last unique attempt (remaining 0 after start); a later
  retry was not possible for this mode.
- **Re-runs.** Two further Time Trial attempts were later scripted with the shared solver in
  `tools/ranked/solvers/`, using the library's `question_facts.py` mapper. Each was CORRECT in one
  call with a wall-clock total of about 2 s (issue 0.6 s, download 0.6-0.7 s, solve 0.1 s, submit
  0.8 s). Their spellings were: `little endian` for endianness, the section list without the
  leading empty name, `PE32_CLR` and `x86` for the Day 3 format and machine.

## Dead ends and gotchas

- **`d01_analysis_symbols` (24 submissions).** The question text read only "What is the exact
  analysis symbols value" with no filter or order. Every other field was accepted by the first
  grouped submissions (batch results are all-or-nothing, so we split the 48 fields by day, then by
  field, to find the single bad one). Rejected readings, all tried against the same Day 1 ELF:
  `["main","recover_code"]`, its reverse, `["recover_code"]`, `marker`+`recover_code` with and
  without `main`, all ten defined FUNC symbols, the six global FUNC symbols, every defined
  GLOBAL-binding symbol, defined default-visibility FUNCs, and the full named `.symtab` without
  `FILE` records. Accepted: the full `.symtab` name sequence including `FILE` entries. The default
  cap of 20 submissions was exhausted on attempt 20, and four more were made after an extension was
  granted. Later fetches of the question carried an explicit rule, so the Day 6 wording was a
  vague-question trap that was later fixed in the live text (we could only date this from saved
  responses; we do not know when the server changed).
- **A Single Solve cross-check that paid off.** The Day 3 and Day 4 accepted answer maps already
  held the full-table symbol value for a member with the identical SHA-256. Comparing member hashes
  across days would have saved most of the 24 calls.
- **Native PE versus managed PE (Mastery).** The first Mastery solver stopped before submitting
  because it treated every PE file as a .NET assembly and failed on a native PE32+ Day 2 member,
  whose CLI header RVA is zero. The fix was to test the CLR data directory first. Nothing was
  submitted by that failed run, and the timer was still running.
- **Prerequisites and bytes.** Early Time Trial and Mastery starts returned `prerequisite_required`
  and then team-state responses with no attempt, artifact or answer IDs; only a response with an
  attempt ID, answer IDs and an archive hash is a usable start. The fix was to complete Single Solve
  first and re-start.
- **Pattern fields.** `d03_analysis_format` took `PE32_CLR` (compound token with the runtime half)
  and the machine field accepted `x86`; we did not map out which other spellings the validator
  would accept.

## Takeaways

- A "recap" challenge's title tells you about the newest specimen only. Count the answer IDs first
  (`dNN_` prefixes), then map the earlier days' specimens by SHA-256 against previously accepted
  maps.
- Validate HCFG records by CRC32 of the decoded payload, never by the first `HCFG` string, and
  look inside class-file string constants for Base64 runs (long `[A-Za-z0-9+/]` strings) when a JAR
  hides a config.
- For an underspecified "symbols" field, submit the complete table of the named section in table
  order before any filtered subset (technique: ELF symbol inventory).
- For short timers, prove the mapper on the previous mode's archive offline, then script
  start, download, verify, solve and one batched submit.

## Evidence

Leaf base: `challenges/day06/binary-reverse-engineering/binary-reverse-engineering-day-06-java-bytecode-loader/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/artifacts/submission-answers.json`,
  `single-solve/artifacts/submission-symbol-extra-04-response.json` (graded CORRECT)
- `time-trial/journal.md`, `time-trial/artifacts/static_solver.py`, `time-trial/artifacts/fast_driver.py`,
  `time-trial/attempt-01/` (first CORRECT), `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/attempt-01/submission-response.json` (graded CORRECT), `mastery/start-response-resumed.json`
- Repo-level: `tools/ranked/solvers/binary-reverse-engineering-day-06-java-bytecode-loader/solve.py`;
  library entries on ELF symbol inventory, class-file inspection, PE/CLR detection and
  cumulative recap bundles.
