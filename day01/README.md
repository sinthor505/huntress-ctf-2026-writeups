# Day 1 overview

**At a glance:** 7 challenges in 6 lanes (Welcome, Warmups, Metadata Forensics, Script Analysis, Binary
Reverse Engineering, Mobile Android), 17 modes played. By provenance: 9 Independent, 2 Journal-recorded,
6 Local mark only, 0 Team; 4 modes absent (the two Welcome challenges are Single Solve only). The theme
was orientation: every challenge is a primer for its lane (decode a blob, hash a file, read plain
PowerShell, read an ELF header, read an asset out of an APK), and the hard part was answer spelling and
the short Time Trial windows, not the analysis.

Day 1 was played on 2026-10-18, before the repo had version control and before release automation, so
there is no `_run/` folder, no lane logs and no librarian notes. For five of the seven challenges the
original Single Solve left no journal, no start response and no graded result, so the evidence we hold
comes mostly from a ranked sweep of Time Trial and Mastery run later (2026-10-26, and 2026-11-01 for
metadata).

## Challenges

Provenance labels: **Independent** = our own graded CORRECT file is in the leaf; **Journal-recorded** = the
journal records our CORRECT submission but no result file was saved; **Local mark only** = only a
`progress.md` "Player-directed local mark" exists; **Team** = a teammate's solve (none on this day);
**n/a** = the mode does not exist.

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Welcome | [Welcome Robots](welcome_welcome-robots.md) | Read the answer (`issued_flag`) out of the start response | Journal-recorded | n/a | n/a |
| Welcome | [Read the Rules](welcome_read-the-rules.md) | Flag in an HTML comment of a client-rendered page; render, then search the DOM | Journal-recorded | n/a | n/a |
| Warmups | [A Little Based](warmups_a-little-based.md) | Base64 decode of one blob, answer is the slug on the last line | Local mark only | Independent | Independent |
| Metadata Forensics | [You Must Be This Hash to Ride](metadata-forensics_you-must-be-this-hash-to-ride.md) | Name, size and MD5/SHA-1/SHA-256 of a ZIP member | Local mark only | Independent | Independent |
| Script Analysis | [Power Play](script-analysis_power-play.md) | Static reading of inert PowerShell: scenario, ATT&CK ID, literal indicators | Local mark only | Independent | Independent |
| Binary Reverse Engineering | [ELF Aware](binary-reverse-engineering_elf-aware.md) | ELF header/section/symbol facts; `mov eax, imm32; ret` stub scan in `.text` | Local mark only | Independent | Local mark only |
| Mobile Android | [Exported Trust](mobile-android_exported-trust.md) | Static APK triage: exported deep-link Activity, `plain` puzzle asset holds the answer | Local mark only | Independent | Independent |

What backs each label:

- **Welcome (both):** the leaf journals record the Game Master reading CORRECT (Welcome Robots on the first
  call, Read the Rules on attempt 2), and `progress.md` dates both 2026-10-23. Neither leaf has a
  `result.json`, `accepted_plan.json` or `manual_solve.json`, so they are Journal-recorded rather than
  Independent. The `progress.md` rows add that an earlier 2026-10-18 local mark preceded any accepted
  submission.
- **Time Trial and Mastery (Independent):** each is backed by a `ranked-1/result.json` reading CORRECT,
  from the scripted ranked pass, not by the original 2026-10-18 attempt. The original attempts have only a
  local mark and no saved response, so the ranked file is the only graded evidence for these modes. All
  but metadata were issued on the legacy route (a plain `retry` was the ranked attempt); the metadata pair
  was issued on the dual route with `run_for_time:true` on 2026-11-01.
- **Single Solve (Local mark only):** the leaves hold only the raw delivery (`blob.txt`, `evidence.zip`,
  `exported_trust.apk`, or the metadata ZIP) and no journal. The solves themselves are plausible and the
  later modes confirm the method, but no graded result survives, so the pages do not claim one.
- **ELF Aware Mastery:** no mode directory exists in the leaf, and the ranked sweep found that mode
  `rotation_exhausted` (no attempts left). The only record is the local mark.

## What the day had in common

- **Primers with fixed shapes.** Each lane's Day 1 family is the simplest member of a family that recurs.
  The note-hash questions return on days 5 and 7-9 with only the container changing, the PowerShell trio
  feeds the Day 2-4 banks, and the APK skeleton is reused by every later mobile day. The ranked passes
  reused the Day 2 lane solvers for metadata, script and mobile because those already read the Day 1
  families.
- **The answer is a literal reading of the delivery.** The warmup prompt ("sixty-four is a good base") is
  a literal Base64 hint; the Android answer sits unencoded in `assets/huntress_puzzle.json` (Day 1 is the
  only day whose asset codec is `plain`); the metadata "three files without trustworthy labels" framing
  suggested a trap, but the delivery was one honest `.txt` member.
- **Short windows forced scripting.** Time Trial windows were 30 s (warmups), 50 s (metadata, script, BRE,
  mobile) and Mastery 80-110 s. Whole passes took roughly 2-7 s once the solver existed, so the solve was
  built first and issue, download, solve and submit ran as one script.

## Validator and format traps

- **Exact-text-first (Read the Rules).** The rules page states the flag format as a 32-character MD5, but
  only the full `flag{...}` text from the HTML comment was accepted. The bare hex came back `incorrect`.
- **Slug, not sentence (Warmups).** The decoded sentence is not the answer; only the slug after the last
  colon matches the pattern.
- **Spelling groups (ELF Aware).** `endianness` took bare `little`; `little-endian` and `little endian` were
  rejected. `incorrect` is batch-level, so the cause only showed up by submitting by field group. The
  other four tested fields (`ELF64`, `x86-64`, `ET_EXEC`, `exported_symbol`) were accepted on their first
  variant.
- **Mode IDs and string answers (all timed modes).** Time Trial and Mastery submit under `<id>/time_trial`
  and `<id>/mastery`; every answer value, including the metadata byte count, is a JSON string. Mastery
  answer IDs carry `setN.` prefixes and must be paired with the right delivery.
- **Decoy credential (Exported Trust).** The `SECRET-...` string the Activity attaches is active but not the
  answer; the proof value is in the puzzle asset.
- **Digests are of the decompressed member (metadata)**, lowercase hex with no prefix, not of the ZIP.

## Mistakes in our own tooling and process

The challenge pages kept these out of their gotchas sections; collected here.

- **Single Solve evidence was not kept (Warmups, Metadata, Script Analysis, BRE, Mobile).** Five of the
  seven leaves have only the raw delivery: no journal, no start response, no result. The 2026-10-18 solves
  exist as `progress.md` local marks only. As a consequence the pages cannot state the Single Solve answer
  or question set for these challenges, and the Day 1 Time Trial/Mastery fixtures had to be rebuilt later
  (see below).
- **Damaged or unreadable saved deliveries (Power Play, Exported Trust).** The saved Power Play
  `evidence.zip` is unreadable (the cause was not recorded), and the saved Exported Trust APK is one of
  the Day 1-5 copies that `CLAUDE.md` says were corrupted by LF-to-CRLF translation. Power Play's Single
  Solve could therefore not be re-analysed. Day 1 also has no recorded hash to verify a repair against
  for the APK.
- **Local marks recorded ahead of graded results (Welcome Robots, Read the Rules).** The `progress.md`
  rows say a 2026-10-18 local mark preceded any accepted submission; the graded CORRECT was only obtained
  and journaled on 2026-10-23. This is the pattern the later "local evidence only" rule exists to prevent.
- **Spelling variants searched inside the clock (ELF Aware).** The ranked Time Trial needed 12 submission
  calls (5.36 s of a 6.67 s total) because the solver's first spelling for `endianness` was wrong.
  `tools/ranked/PROCESS.md` records that the solver was changed to submit `little` first, expected to need
  one call. Also, `accepted_plan.json` still lists the rejected primary (`little-endian`); the winning
  variant is the chosen index in `result.json`, so the plan file alone is misleading.
- **Wasted first-attempt risk on a pinned challenge (Metadata).** The first ranked tries on Time Trial and
  Mastery returned 409 `legacy_attempt_pinned` (server-side, nothing issued or consumed). That was not our
  fault, but the sweep recorded Day 1 as 7 of 10 until the pin cleared on 2026-11-01.

## Tooling and process changes around this day

Day 1 itself predates version control (the repo history starts 2026-10-22), so nothing was changed in
tooling at the time. What later work added, each tied to a file or commit:

- **2026-10-22, `5b92dd2e` "Add Day 1 challenge work":** first commit touching the day; it holds only the
  warmups `blob.txt` among the Day 1 leaves (per the commit's file stat).
- **2026-10-23, `b0bb003f` "Record Day 01 Welcome Robots and Read the Rules solves":** the two Welcome
  journals, answer files and start responses. The Welcome journals carry a 2026-10-24 library promotion
  to `SHARED-ORIENTATION-001` (`library/shared/orientation-challenges.md`: issued-flag reading, DOM search,
  exact-text-first submission). The same lesson is in `CLAUDE.md` ("Welcome (Day 1) Read the Rules").
- **2026-10-25, `1df4ec2c` "Add Day 1-10 ranked-attempt results, solvers and tooling":** the ranked
  sweep tooling for Days 1-10 (`tools/ranked_pass.py`, per-challenge solvers, `tools/ranked/run_day01.sh`,
  `tools/ranked/cases/day01.json`). Day 1 got its own BRE solver
  (`tools/ranked/solvers/binary-reverse-engineering-day-01-c-elf-orientation/`) and a warmups solver; the
  metadata, script and mobile modes reused Day 2 solvers.
- **Day 1 recipe for missing fixtures** (`tools/ranked/PROCESS.md`, "Building fixtures when none were
  saved"): because the original Time Trial/Mastery deliveries were not kept, `status` on a solved mode
  returns the start-like payload and the artifacts are re-streamed from it, with `flag` and `solved_by`
  stripped. This is how offline fixtures for Day 1 were built.
- **`ranked_pass.merge_set_groups`** merges same-named alternate-spelling groups across Mastery sets; the
  `CLAUDE.md` note ties it to the Day 1 BRE `little` search.
- **2026-10-26 ranked sweep:** `PROCESS.md` section 8 records Day 1 at 7 of 10 CORRECT (warmups, mobile and
  script Time Trial and Mastery, BRE Time Trial), with metadata Time Trial and Mastery at
  `legacy_attempt_pinned` and BRE Mastery at `rotation_exhausted`. Welcome challenges are Single Solve only
  and not ranked.
- **2026-10-31, `021814b6` "Ranked reruns: group 1 gap closure":** the commit that added the metadata
  `ranked-1` results. Both modes ran on 2026-11-01 (result deadlines stamped 2026-11-01 UTC) on the dual
  route with `run_for_time:true`, CORRECT on call 1 using the Day 2 metadata solver. This lifted Day 1 to 9
  of 10 timed modes ranked CORRECT.

## Timeline

| Date | Event | Source |
|---|---|---|
| 2026-10-18 | Day 1 solved in all lanes (Welcome local marks, warmups, metadata, script, BRE, mobile, three modes each) | `progress.md` local-mark rows |
| 2026-10-23 | Welcome Robots (1 call) and Read the Rules (2 calls) graded CORRECT | Welcome journals, `progress.md` |
| 2026-10-25 | Ranked-sweep tooling and Day 1-10 results committed | `1df4ec2c` |
| 2026-10-26 | Ranked Time Trial and Mastery passes for warmups, script, mobile; BRE Time Trial only | `ranked-1/result.json` deadlines |
| 2026-11-01 | Metadata Time Trial (5.51 s) and Mastery (5.68 s) ranked, 1 call each | metadata `ranked-1/result.json`, `progress.md` |

Per-pass totals from the ranked results (seconds, `t_total`): warmups 2.18 / 2.02, script 1.91 / 2.64,
mobile 2.78 / 3.36, BRE Time Trial 6.67, metadata 5.51 / 5.68 (Time Trial / Mastery).
