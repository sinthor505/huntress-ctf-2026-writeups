# Day 9 overview

**At a glance:** 5 challenges, 15 modes (Single Solve, Time Trial and Mastery for every lane). By provenance: 14 Independent, 1 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a "recap" day, where several lanes re-asked earlier days' fields under a new title and the work was getting the validator's spelling and rubric right, not finding a new trick.

Web Exploitation and Welcome are absent: the web lane went live on Day 19, so Day 9 has five lanes.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Escape Room](warmups_escape-room.md) | JSON string-literal decoding; Mastery is 43 blobs of nine fragments in nine codecs, dispatched by line number | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Built Different](metadata-forensics_built-different.md) | Byte-level header reads: hashes, JPEG SOF, PNG IHDR, PDF header, ZIP EOCD, PE COFF Machine | Independent | Independent | Independent |
| Script Analysis | [All the World's a Stager](script-analysis_all-the-worlds-a-stager.md) | Static decode of per-file encoded descriptors in 83 scripts, then rubric classification (role, ATT&CK, wrapper style) | Independent | Independent | Independent |
| Mobile Android | [Trust Everyone](mobile-android_trust-everyone.md) | Static read of a declared Base64 asset codec; the TLS-trust theme is decoration | Independent | Independent | Independent |
| Binary Reverse Engineering | [Nimble](binary-reverse-engineering_nimble.md) | Static recap of ELF/PE/CLR/JAR facts and HCFG records (XOR + CRC32, Base64 carrier runs) | Independent | Independent | Independent |

What backs each label (from the pages' evidence sections):

- **Journal-recorded**: Escape Room Single Solve. The journal records one submission and `CORRECT`; no result or response file was saved.
- **Independent** rows rest on a saved `CORRECT` submission response, a ranked/rerun `result.json`, or both. Escape Room Time Trial's original attempt is journal-only, and the row is Independent because later ranked reruns of that mode have `result.json` `CORRECT`.
- Single Solve rows for the other four lanes each have a saved `CORRECT` submission response (Script Analysis on submission 43 of 43).
- Nothing on Day 9 was marked solved from `progress.md` alone, and no mode was team-scored.

## What the day had in common

- **Titles and prompts were themes.** "Nim runtime fingerprints" (BRE) and "a permissive trust implementation" (Mobile) never appeared in the questions. The Nimble questions are a recap of the Day 1-9 binary challenges, and the Trust Everyone answer sits in a Base64 asset, and the DEX contains no trust-manager code. Read the question list and the asset or header first. Nimble's page also says we never established that the specimen is a Nim build.
- **Deterministic extraction, uncertain labelling.** Warmups, Metadata, Mobile and the extraction half of Script and BRE were read straight from the bytes with standard-library code (`json`, `zipfile`, `struct`, `hashlib`) and cross-checked against a second tool. The calls that were spent went on classification fields: Script Analysis used 43 Single Solve calls and 10 Time Trial requests, BRE Mastery 8.
- **Each mode is a fresh fixture.** Time Trial and Mastery regenerated the specimens (new digests, case ids, CRCs, offsets), so no answer carried across modes and the Mastery bundles had to be solved per set (Metadata's per-set note digests, BRE's PE-versus-ELF d02 specimens).
- **Batch feedback is all-or-nothing.** A mixed batch only returns `incorrect`. The workable method, in Script Analysis and BRE Mastery, was to submit by family or by set so accepted groups bank their fields and the failing group shrinks.

### Validator and format traps seen on this day

- **Descriptor `attack` wins (Script Analysis).** For PHP/Lua polyglots and `host-*` containers the descriptor's own `attack` field was accepted even where the steps pointed elsewhere; two PHP files with identical decoded graphs needed `none` and `T1041`. For the five `dispatch_language` graphs only the descriptor's value passed.
- **Python chains ending `dispatch_process` take `T1059.006`, not `T1105`.** The ingress reading `T1105` was accepted for Bash and Node.js chains on this day. CLAUDE.md lists Day 9 among the days where `T1105` was rejected for Python.
- **Batch controls follow structure, not the prompt.** A one-item `for /l ... do call :LABEL` wrapper was `call_subroutine` although the question said to answer `for_once`; `if defined` on the reconstruction line was `if_guard`.
- **Ordered transforms do not commute (Script Analysis).** Reversal in the enclosing call, reversal in the decoder and XOR must be applied in source order.
- **BRE answer spellings.** Accepted: d01 endianness `little endian`, d03 format `PE32_CLR`, d03 machine `Intel 80386`, d04/d05 format `ELF`. Enum and exact-text fields are exact.
- **BRE offset semantics.** `d09_analysis_hcfg_offset` is the offset of the first Base64 character of the carrier run, not the position of the record inside the decoded bytes. The literal `HCFG` strings in d08/d09 are decoys.
- **BRE PE symbol predicates.** "A nonempty symbol name" applies to PE COFF symbol tables too, not just ELF `SHT_SYMTAB`.
- **Metadata formats.** `pe.coff_machine_u16_hex` wants `0x` plus four lowercase hex digits, member lists are compact JSON in stored order, booleans are lowercase strings. The first question's text gains an `[assigned-artifact-sha256:...]` tag in Time Trial, so classify questions by ID tail.
- **Mode IDs.** Time Trial and Mastery calls must carry `<id>/time_trial` or `<id>/mastery`; the bare ID returns `already_solved` and scores nothing (CLAUDE.md cites Days 9 and 13).
- **Mastery delivers several artifacts.** The BRE page warns that the generic artifact operation can return a different sample than the delivery's own link; use each delivery's own link and verify its SHA-256.

## Mistakes in our own tooling and process

Collected from the pages that deferred them here.

- **All the World's a Stager, Time Trial: wrong mode ID and a late submit.** The original timed attempt's first submission went in a couple of seconds after the 50 s deadline and carried the bare challenge ID. The service answered `already_solved` (the earlier Single Solve completion) and scored nothing, so the 695-field map was correct but unscored. Retry 2 was timed and expired after a 689-field request, because the first mapper pass left 6 fields unmapped (it had not followed source order for reversals). Retry 3 was untimed and went CORRECT on request 10 for that mode, the first two requests having been spent on the unscored and expired attempts.
- **All the World's a Stager, Single Solve: first Batch-control mapper.** It searched only lines before the reconstruction assignment and answered `direct`, so the `if defined` files needed a separate fix (call 19). The 124- and 34-field batches that came back `incorrect` with no hint are why the bisection took 43 calls.
- **Nimble, Mastery: the mapper only read ELF symbol tables.** For any PE it fell back to `false`, which was wrong for the d02 predicates in sets 2 and 4. The full 450-field batch, set 2 and set 4 came back `incorrect`; sets 1, 3 and 5 were accepted; the corrected sets 2 and 4 then went through (set 4 returning `CORRECT`). The fix was a COFF symbol-table reader. The journal says this was the only substantive miss.
- **Trust Everyone, Mastery: positional pairing.** The mapper read each set's asset facts by question index. That was correct for this bundle but fragile; the later per-challenge solver pairs questions to APKs by the package name in the question text.
- **Nimble, later ranked reruns: solver run under `python -I`.** Two passes (Time Trial `ranked-6`, Mastery `ranked-4`, both timestamped 2026-11-01) failed with a `needs pyzipper` error because the runner's isolated mode hides user-site packages. Each folder holds a saved `start_response.json`, so the attempt had been issued before the solver failed. This is the failure mode `tools/ranked/PROCESS.md` documents ("can strand a live attempt"); we did not determine from the day's files how these two attempts were afterwards resolved. Other passes in the same folders are `CORRECT`.
- **Escape Room, Time Trial: lost result files.** `ranked-3` was a CORRECT rerun from the PC on 2026-11-01, but its `result.json`, `start_response.json`, `pass.log` and `accepted_plan.json` were overwritten when a copy of `ranked-3` from the second machine was synced in. Its `NOTE.md` records the outcome from a batch log, which is why this page's label rests on `ranked-1`, `ranked-2`, `ranked-4` and `open-1`.
- **Ranked reruns landed under the wrong scoreboard identity.** `tools/ranked/PROCESS.md` records that seven Day 9 modes had ranked rows only under our solo identity, not the team's, and that five were blocked by a stale open untimed attempt (409 `attempt_active`). See the tooling section below.
- **Record-keeping.** Escape Room Single Solve saved no response file, so its provenance is journal-only. The Script Analysis page's evidence list names the Single Solve submission files without their `artifacts/` directory (they live under `single-solve/artifacts/`).

## Tooling and process changes around this day

The Day 9 solves predate version control (progress rows are dated 2026-10-20; history starts 2026-10-22), so there are no commits for the original work. What git and the docs show afterwards:

- `0b886c57` (2026-10-22) first commits Day 9's challenge work.
- `1df4ec2c` (2026-10-25): adds the Days 1-10 ranked-attempt results, per-challenge solvers and tooling. The four per-challenge Day 9 solvers under `tools/ranked/solvers/` (Metadata, Script Analysis, Mobile, BRE) first appear in this commit, with the Metadata fixture list in `tools/ranked/cases/metadata-d06-10.json`; the Warmups rerun uses the generic warmups solver. CLAUDE.md records Day 9 as a dual-pool route for all modes (`run_for_time:true` is the ranked call, a plain `retry` is an untimed rehearsal).
- `bf74045d` (2026-10-28), with `118b0921` saving the open-attempt payloads: the team-attributed Day 9 reruns. Blame DNS had no ranked row for seven Day 9 modes. The commit message says five were blocked by a stale open untimed attempt, that `tools/ranked/finish_open.py` was added to solve and submit such an attempt from its status payload, that `run_day610.sh` gained `ONLY` and `AGAIN` switches, and that all seven reruns went CORRECT and now rank under the team. `PROCESS.md` documents the shared-attempt rule. `open-1` folders exist in the Warmups Time Trial and Mastery, Metadata Time Trial, Mobile Time Trial and Script Analysis Time Trial leaves; we did not check which of the seven reruns each one corresponds to.
- 2026-11-01 (`38668151`, `f0e20aa1`): the top-30 gap-closure reruns touched the Day 9 leaves again (the Escape Room `ranked-3` loss and the two BRE `pyzipper` failures above are from this sweep).
- CLAUDE.md lessons that cite this day: the mode-ID rule (bare ID returns `already_solved`, Days 9 and 13) and the Python `T1059.006` rule (`T1105` rejected on Days 6, 8, 9, 11). The `elf` format spelling for d09 BRE is also recorded in the Days 11-15 ranked-pass lessons.

## Effort

Submission counts per mode, from the pages and `progress.md` rows (all rows dated 2026-10-20, status Solved):

| Lane | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Warmups | 1 | 1 | 1 (43 fields) |
| Metadata Forensics | 1 (13 fields) | 1 | 1 (65 fields) |
| Script Analysis | 43 (695 fields) | 10 requests (the first two went to the unscored and expired attempts) | 1 (3,475 fields) |
| Mobile Android | 1 | 1 | 1 (5 APKs) |
| Binary RE | 1 (90 fields) | 1 | 8 (450 fields) |

Timeline, from the journals (2026-10-20, UTC, only where a journal states a time):

| Time | Event |
|---|---|
| 05:06:39 | Script Analysis Single Solve recorded as completed (quoted in the Time Trial journal) |
| 05:07:57 | Script Analysis original Time Trial deadline; our submit came just after |
| 05:14:05 to 05:17:21 | BRE Mastery window, ending on the corrected set 4 submission `CORRECT` |
| 05:15:02 to 05:15:52 | Script Analysis Time Trial retry 2 window; its late request returned `expired` |
| 05:17:22 | Script Analysis Time Trial retry 3 started, untimed |

One date we could not reconcile: the BRE Single Solve journal states a start time of 2026-10-11 from the Game Master response, while its `progress.md` row is dated 2026-10-20. We do not know which attempt the earlier timestamp refers to.
