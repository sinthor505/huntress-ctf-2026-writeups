# Day 3 overview

**At a glance:** 5 challenges (one per lane: warmups, metadata forensics, script analysis, mobile Android, binary reverse engineering), 15 modes. By provenance: 11 Independent, 2 Journal-recorded, 2 Local mark only, 0 Team, 0 absent. Theme: the first "recap" day, where a small decoding task per lane turned into a scripting race against 20 to 110 second Time Trial and Mastery windows.

Day 3 was played before release automation existed, and we started playing well after it was released. Nothing here has a `_run/` folder, librarian notes or lane logs, so every claim comes from the leaf journals, saved response files, `progress.md` and the ranked-sweep result files. The Game Master's `status` readout was not used as evidence for any row.

A leaf named `web-exploitation-day-03-sqli` also sits under `challenges/day03/`. It belongs to Day 19 per `progress.md` and is deliberately not part of Day 3.

## Challenges

Labels: **Independent** = our own CORRECT file exists in the leaf (`result.json`, `submission-response.json` reading CORRECT, or a ranked `result.json`). **Journal-recorded** = the journal records our own submission as CORRECT but no result file was saved. **Local mark only** = only the `progress.md` local mark (or a journal saying the server already showed CORRECT) backs it. **Team** = a teammate's solve, not ours. **n/a** = mode absent.

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Hex Appeal](warmups_hex-appeal.md) | Hex blob; Mastery bundles numbered base64 / ROT13 / hex records | Local mark only | Independent | Independent |
| Metadata Forensics | [BOM Voyage](metadata-forensics_bom-voyage.md) | UTF-8 BOM (`EF BB BF`) byte-prefix check on ZIP members; hashes and magic bytes in the ranked fixtures | Journal-recorded | Independent | Independent |
| Script Analysis | [Expansion Pack](script-analysis_expansion-pack.md) | Static deobfuscation of PowerShell / JScript / Batch descriptors; cmd.exe expansion and control-flow classification | Local mark only | Independent | Independent |
| Mobile Android | [Intent Injection](mobile-android_intent-injection.md) | Static APK triage; base64 puzzle asset behind an Intent-injection decoy | Journal-recorded | Independent | Independent |
| Binary RE | [Table Manners](binary-reverse-engineering_table-manners.md) | .NET CLR metadata triage; XOR-encoded `HCFG` config record; ELF facts (cumulative recap) | Independent | Independent | Independent |

Notes on what backs each label:

- **Warmups Time Trial / Mastery:** CORRECT on the 2026-10-20 re-solve (journals, 1 submission each) and again in the 2026-10-26 ranked reruns (`ranked-1/result.json`). Single Solve has no leaf directory at all, only the `progress.md` row.
- **BOM Voyage and Intent Injection Time Trial / Mastery:** the original 2026-10-18 solves are in the journals; the ranked reruns carry the result files. Their Single Solves have journal entries recording CORRECT and nothing else.
- **Expansion Pack Single Solve:** the journal says the server already showed the challenge as CORRECT before this session's workflow and that no submission was made. We cannot tell from the files who made that earlier solve or when, so we claim nothing graded. Time Trial and Mastery have ranked result files; Mastery was also re-solved on 2026-10-20 (1 submission).
- **Table Manners:** the only mode set where the original 2026-10-18 attempts each saved a `submission-response.json` reading CORRECT (Single Solve, Time Trial and Mastery), with the Time Trial also re-run ranked. There is no ranked Mastery run: its start reported `unique_attempts_remaining: 0`, and `CLAUDE.md` records BRE Mastery on Days 2-5 as `rotation_exhausted`.

## What the day had in common

- **The prompt is a theme, the questions are the task.** Table Manners talks about .NET metadata tables but asks 24 fields that recap Days 1 to 3 of the binary lane. Intent Injection talks about nested Intents but the answer is a base64 field in `assets/huntress_puzzle.json`. BOM Voyage talks about "encoding landmines" but the first-round question is one boolean. Read the answer IDs before choosing what to analyze.
- **Single Solve is small; Time Trial and Mastery widen.** BOM Voyage went from 1 field (original attempts) to 7 per file in the ranked fixtures (name, size, three digests, first 16 bytes, BOM flag). Mastery bundles had 2 to 13 sets (warmups 13 blobs, script 2 sets of 159 fields, BRE 2 sets of 24, mobile 3 APKs, BOM 2 sets). A solver written for the Single Solve fixture needed to be per-set and manifest-driven.
- **Fixtures change per attempt.** Each Time Trial retry issued a new case, package, DLL variant or file name, so answers never carried between attempts (Intent Injection's expired first attempt gave an answer that was useless for the retry's APK). Mastery sets differed from each other (Table Manners set 2 had a PE32+ Day 2 specimen).
- **Windows were short and set by the server.** Warmups Time Trial 20 s, Mastery 30 s; metadata and BRE Time Trial 50 s, Mastery 80 s; mobile 50 s and 110 s; script 8 s on an early Time Trial attempt, 50 s and 80 s later. These came even when the attempt was started untimed (warmups journals note `hall_of_fame_eligible: true` on an "untimed" start), so start, download, SHA-256 check, map and submit had to be one scripted pass.
- **Locks.** Mobile Time Trial returned `prerequisite_required` until Single Solve was CORRECT, and Mastery until Time Trial was.

## Validator and format traps seen across lanes

- **Strings only.** Table Manners' first Single Solve submission sent JSON arrays for section names, symbols and disassembly and returned `invalid_answers`. Arrays have to be compact JSON text.
- **The prompt's rule can disagree with the validator.** Expansion Pack's Batch control prompt says a one-item `FOR` wrapper around a subroutine call is `for_once`; the accepted maps used `call_subroutine`. Treat such rules as hypotheses and submit by structural group.
- **Exact spellings.** Table Manners accepted `I386` for the machine field and `PE32_CLR` (not bare `PE32`) for the compound container, and AT&T disassembly (`mov $0x1fa4,%eax`); the Intel form was never tested. `d01_analysis_symbols` was ambiguous in the prompt and the full `.symtab` name list (including FILE entries and duplicates) was accepted.
- **Answer IDs are not derived from file names.** BOM Voyage Mastery set 2 used `set2.d03-text-001-text-utf8-bom-present` for a file called `export-001-with-bom.txt`. Take IDs from the live start response.
- **Delivery shape.** Script Mastery delivered two separate `evidence.zip` downloads in `artifacts[]`, and the `artifact` op returns only the first.
- **Damaged local copies.** The saved Day 3 ZIPs/APKs were later found corrupted by line-ending translation (BOM Voyage, Intent Injection, Table Manners, Expansion Pack pages all note it). The SHA-256 in the start response is the check; live solves used fresh bytes.
- **Look-alike facts.** BRE: CLI metadata version `v4.0.30319` versus target framework `.NET 8.0`; a backslash in a mutex that JSON shows doubled. Mobile: the exported-activity vulnerability is real but is a decoy for the answer.

## Mistakes in our own tooling and process

Collected from the pages that deferred them here:

- **Warmups, Hex Appeal.** The extractor guessed the codec per chunk from its shape (hex, then base64 with a printable check, then ROT13 as catch-all) instead of keying on the record number. It worked because Day 3 used only three codecs and no mis-decode is recorded, but it would not have survived later days; the generalized numbered-record decoder replaced it. The Single Solve left no trace at all: no leaf directory, only a `progress.md` mark.
- **Metadata, BOM Voyage.** The solver's default of deriving the answer ID from the member name would have been wrong for Mastery set 2, so IDs were passed by hand. The start responses of the original attempts were not saved (only the journals describe the questions). A scouting note carried a mistyped SHA-256; it did not affect the answer. We did not review the ranked solver for the 7-field fixtures, so how it differs from the original single-field solver is undetermined.
- **Script, Expansion Pack.** Several classifier mistakes of ours, all fixed by later structural classification: the first Single Solve and Time Trial maps followed the prompt's `for_once` rule; a regex looked for `if errorlevel` on the assignment line and called a guarded launcher `direct`; a regex called a `named_expression` JScript function a `declaration` (five fields wrong in Single Solve and Time Trial). An early Time Trial window closed with three Batch groups (dropper, uploader, relay) `incorrect`; the uploader and relay causes were never diagnosed. The first Mastery candidate file was built from bytes of an attempt that expired unsubmitted. The driver initially fetched only the first artifact. In the ranked Mastery pass the base call (317 of 318 accepted) held one `if_init` control field back, which cost a second call.
- **Mobile, Intent Injection.** The first Time Trial attempt expired during manual handling (journal records the `expired` result and no cause), costing a retry from a pool the journal shows at 14. The Single Solve's `challenge.json` is an older wrapped response shape, not a plain start response.
- **Binary RE, Table Manners.** The first Single Solve submission used native array types and was rejected as `invalid_answers`. The extractor located the config with a plain `data.find(b"HCFG")`; the table-driven `FieldRVA` explanation of where it lives was worked out only afterward during library promotion. Mastery's start reported `unique_attempts_remaining: 0`, so there was no second try.

## Tooling and process changes around this day

Day 3 predates version control (history begins 2026-10-22), so there are no commits from the solve itself. What the files do show:

- **2026-10-18 to 2026-10-20.** `progress.md` dates the Single Solves 2026-10-18 and the original Time Trial / Mastery solves 2026-10-18 for BOM Voyage, Intent Injection, Table Manners and the script Time Trial. The 2026-10-20 audit re-solved warmups Time Trial and Mastery and script Mastery from freshly assigned, hash-verified bytes (their original marks were local only). `CLAUDE.md` (2026-10-20 entries) records that an untimed `start` still produced windows of roughly 20 to 110 s on Days 2-3, and that member and field targets change by mode on Days 2-3. The library curation sweep of 2026-10-20 is cited in `library/README.md`.
- **2026-10-22.** Commit `36baa7e8` ("Add Day 3 challenge work") is the bulk import of the Day 3 leaves, and `3bcba4e6` ("Add technique library") added the library the pages cite. Neither is a record of when the solves happened.
- **2026-10-24.** Commit `3c8ce72b` touched three Day 3 Single Solve journals (BRE, BOM Voyage, mobile). We did not diff what it changed.
- **2026-10-25.** Commit `1160dffa` ("Add ranked-attempt tooling and Day 2-5 ranked results") added per-challenge ranked solvers for four of the five lanes (BRE, metadata, mobile, script) and the Day 3 `ranked-1/` results. The Days 1-5 routes are legacy: a plain `retry` is already the ranked attempt, so there was no free rehearsal.
- **Ranked sweep, 2026-10-26 (UTC deadlines in the result files).** One scripted ranked attempt per mode (`tools/ranked/PROCESS.md` rule 0: one authorized attempt per challenge+mode). 9 of the 10 Time Trial / Mastery modes have a ranked `result.json` reading CORRECT; BRE Mastery has none (`CLAUDE.md` records BRE Mastery on Days 2-5 as `rotation_exhausted`). All CORRECT results needed one submission call except Expansion Pack Mastery (two). `server_solve_s` in these files is negative (clock skew on the machine used) and is not meaningful.

## Effort and timing

Only from files that state them.

| Mode | Original attempt | Ranked rerun (`t_total`, window) |
|---|---|---|
| Warmups Time Trial | CORRECT 2026-10-20, 1 submission, 20 s window met | 1.99 s (20 s) |
| Warmups Mastery | CORRECT 2026-10-20, 1 submission, 30 s window met | 4.63 s (30 s); the artifact download alone was 2.48 s |
| BOM Voyage Time Trial | Submitted about 19 s into a 50 s window | 1.44 s (50 s) |
| BOM Voyage Mastery | Submitted about 28 s into an 80 s window | 1.30 s (80 s) |
| Expansion Pack Time Trial | Early attempt in an 8 s window had three groups rejected | 1.43 s (50 s) |
| Expansion Pack Mastery | CORRECT on 2026-10-20 re-solve, 1 submission | 2.30 s (80 s), 2 calls |
| Intent Injection Time Trial | First attempt expired; retry CORRECT | 1.38 s (50 s) |
| Intent Injection Mastery | CORRECT, 3 APKs, 1 call | 1.69 s (110 s) |
| Table Manners Time Trial | CORRECT, first submission | 1.38 s (50 s) |
| Table Manners Mastery | CORRECT | none (pool exhausted) |

The Table Manners submission responses carry server timestamps 2026-10-18T09:34:59Z (Single Solve), 09:35:11Z (Time Trial) and 09:35:50Z (Mastery), so all three modes were answered within about a minute using one extractor. The Intent Injection journal records its Single Solve CORRECT at 2026-10-18T09:37:49Z.

No `submission-attempts.md` exists for Day 3, so per-ID submission counts are only the ones the pages cite from journals and `progress.md` (for example Table Manners Single Solve took two submissions, the first being the `invalid_answers` one).
