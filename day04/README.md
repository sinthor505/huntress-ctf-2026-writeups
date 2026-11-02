# Day 4 overview

**At a glance:** 5 challenges (one per lane: Warmups, Metadata Forensics, Mobile Android, Binary Reverse
Engineering, Script Analysis), 15 modes. Provenance by mode: 11 Independent, 4 Journal-recorded, 0 Local mark
only, 0 Team, 0 n/a. Theme: a day where four of the five prompts described something the questions never tested,
so the work was reading the question list and decoding the one structure it named, under 20 to 110 second
windows.

All five challenges have all three modes (Single Solve, Time Trial, Mastery). Day 4 was played before release
automation existed, so there is no per-day run folder; the sources are the leaf journals, saved responses,
`progress.md` and the ranked-sweep result files.

## Challenges

| Lane | Title | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Two of a Kind](warmups_two-of-a-kind.md) | 8-bit binary text decode; Mastery adds numbered per-record codecs (base64, ROT13, hex, binary) | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Picture This](metadata-forensics_picture-this.md) | JPEG Start-of-Frame (SOF) marker walk for pixel dimensions | Journal-recorded | Independent | Independent |
| Mobile Android | [Pending Intent Relay](mobile-android_pending-intent-relay.md) | Static APK triage; answer in a XOR + Base64 asset (`puzzle.v1`), not in the code the prompt describes | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Go Fish](binary-reverse-engineering_go-fish.md) | Cumulative evidence bundle: ELF/PE headers, symbols, XOR-encoded `HCFG` config records with CRC32 self-check | Journal-recorded | Independent | Independent |
| Script Analysis | [Dim and Dimmer](script-analysis_dim-and-dimmer.md) | Static descriptor recovery from obfuscated JScript/Batch/VBScript/PowerShell, then structural classification | Independent | Independent | Independent |

What backs each label:

- **Independent** means the leaf holds our own graded `CORRECT`: a saved `submission-response.json` reading
  `CORRECT` (Time Trial and Mastery of Warmups, Metadata and Mobile; BRE Time Trial and Mastery; Script Single
  Solve, Time Trial and Mastery), and/or a `ranked-N/result.json` from the later ranked sweep (9 modes: every
  Time Trial and Mastery except BRE Mastery; 11 result files, because Script Time Trial and Mastery each have
  two). BRE Mastery rests on its original response alone: its timed pool was `rotation_exhausted`, as
  `tools/ranked/PROCESS.md` lists for Days 2, 3, 4 and 6 BRE Mastery.
- **Journal-recorded** (the Single Solves of Warmups, Metadata, Mobile and BRE) means the journal records our own
  `{"result":"CORRECT"}` response but no result file was saved. `progress.md` marks every Day 4 mode `Solved`,
  2026-10-18; the labels here are stricter than those rows.

## What the day had in common

- **The prompt was the decoy in four of five lanes.** Mobile's "recover and redirect a mutable
  `PendingIntent`" has no `PendingIntent` API in the DEX; BRE's "Go binaries are chatty" never asks for a Go
  function name or path; Script's prompt says VBScript and the ID says `shell-contrast`, but the archive mixes
  four languages and has no shell family; Metadata promises a dozen photographs and a mismatched image but asked
  for one SOF field. Warmups' "two sides" does fit the binary encoding. In each case the question list named the
  real structure. The first thing worth doing was reading the questions, not the prompt.
- **Cumulative or recap content.** BRE's 32 fields were a recap of Days 1 to 4 (hashes, headers, `HCFG`), not
  Go analysis, and Warmups Mastery's record count equals the day number (4 records, 4 codecs).
- **The same extractor served all three modes in four lanes.** Warmups, Metadata, Mobile and BRE needed at most a
  small generalization for Mastery: per-record dispatch, nested ZIP recursion, per-APK codec/key read, and ELF
  vs PE dispatch on magic bytes. Script Analysis was the exception and took seven Time Trial windows.
- **Short, server-enforced windows.** Observed windows were 20 s (Warmups TT), 35 s (Warmups Mastery), 50 s
  (Time Trial of the other lanes), 80 s (Metadata, BRE, Script Mastery) and 110 s (Mobile Mastery). A plain
  `retry` was already the ranked attempt, so every Time Trial and Mastery was scripted download-to-submit.
- **Every retry can deliver new fixtures.** Script Time Trial issued a freshly randomized archive each window
  (CLAUDE.md cites "Day 4 Time Trial" for this), and BRE Time Trial and Mastery changed the d03/d04 variants and
  even the container type.

### Validator and format traps

| Lane | Trap |
|---|---|
| Script | Batch `control` values (`call_subroutine`, `for_once`, `if_guard`, `errorlevel_guard`, `direct`) follow the specimen's own structure, not the prompt's hint; `IF ERRORLEVEL 1 EXIT /B 1` before the join gives `errorlevel_guard`, an `IF DEFINED ... SET cfg=` guard gives `if_guard`. VBScript `control` has no `if_guard` value. |
| Script | ATT&CK primary: the bare parent `T1059` was accepted for terminal-dispatch graphs on this day. `incorrect` has no per-field feedback; narrowing needs bounded groups of IDs. |
| BRE | Accepted spellings: `PE32_CLR`, `I386`, `little-endian`, AT&T-syntax disassembly. `d01_analysis_symbols` is every non-empty `.symtab` name including `FILE` entries and duplicates. Mastery mixes ELF and PE for the same logical specimen. |
| Metadata | Answer shape `WxH` (width first) had no example in the prompt; SOF stores height first. Later ranked fixtures asked 8 fields per set (adds the evidence-note family) instead of 1. |
| Mobile | `huntress.mobile.puzzle.v1`, codec `xor`; Mastery questions pair to APKs by package name, not delivery order. |
| Warmups | Slug is the last non-empty decoded line; Mastery blobs are `NN<TAB>payload` and the record number picks the codec. |
| All | Mastery returned `prerequisite_required` until the Time Trial was CORRECT (Script Analysis journal). |

## Mistakes in our own tooling and process

These were deferred from the challenge pages.

- **Dim and Dimmer (Script Analysis): no variant corpus before the timer.** The decoder was validated against
  one fixture at a time, so seven Time Trial windows each ended with a different one-bug difference: window 1
  expired on parser gaps; window 2 appended an assembled Batch variable to `next_stage`; window 3 lacked a
  JScript reversed-array-plus-XOR decoder and then ran out of time splitting into lanes; window 4 picked a
  four-byte distractor array and sent four Batch/VBScript controls as `direct`; windows 5 and 6 guessed Batch
  controls and the 5th window's last probe arrived after the deadline (`expired`); window 7 passed. CLAUDE.md
  records the resulting rule ("test the decoder on every saved variant before the timer",
  `corpus_variant_report.py`, `accepted_map_regress.py`).
- **Dim and Dimmer: stale journal text.** The Time Trial journal's interim "Outcome" sections say the mode was
  not solved; a 2026-10-20 audit note supersedes them (attempt 7 was accepted before its deadline). The page
  follows the audit note and the saved `CORRECT` response.
- **Dim and Dimmer: environment.** The solver ran from a throwaway virtualenv because the system Python lacked
  `pyzipper`, which is needed for the AES-encrypted evidence ZIP.
- **Pending Intent Relay (Mobile): logging-template bug.** The Mastery journal printed `None from None` for each
  solver run because the template read `answer`/`artifact` while the script wrote `decoded_value`/`case`. The
  real per-APK outputs are in `mastery/delivery-evidence.json`; the 2026-10-20 sweep recorded the correction.
- **Go Fish (BRE): ELF-only parser in Mastery.** The first Mastery solver run stopped when the second set's d02
  and d04 were PE32+ files. It stopped before submitting, so nothing was lost, but the Mastery start had
  reported no unique attempts remaining, so a failed submission would have had no second try.
- **LF to CRLF damage to saved artifacts (Warmups Mastery, Metadata, Mobile, BRE, and per CLAUDE.md all Day 1 to
  5 ZIPs/APKs).** Saved copies were longer than the served files (Warmups bundle +54 bytes; Mobile APKs +22 to
  +35) and `zipfile` reported `BadZipFile`. Undoing the added CRs reproduced the start-response SHA-256 in every
  case checked. An early Warmups note blamed the server's ZIP headers; the 2026-10-24 re-check showed it was
  our own copy. The live solves ran on the downloaded bytes and were unaffected.
- **Warmups: copied template text.** The Warmups journals carry "Android work was static: APKs were not
  installed or executed", copied from another lane and wrong for a text bundle.
- **Missing start responses for four Single Solves.** The Warmups, Metadata, Mobile and BRE Single Solve leaves
  have no saved `start_response.json` (the BRE and Mobile journals say so).

## Tooling and process around this day

Day 4 was solved before the repository existed in version control, and the repository history starts on
2026-10-22. There are no commits from the solve itself. What the history and CLAUDE.md support:

- The leaf work is in one bulk commit, `12403d85` "Add Day 4 challenge work" (2026-10-22), four days after the
  2026-10-18 timestamps in the journals.
- The journals carry a "Curation sweep 2026-10-20 (retroactive, Librarian role)" block, so the library
  entries named in the pages (JPEG SOF reader, Go build-info parser, HCFG records, Batch control
  classification) were curated after the solves.
- Ranked sweep: commit `1160dffa` (2026-10-25) added the ranked-attempt tooling and the Day 2 to 5 ranked
  results, including `tools/ranked/run_day45.sh` and per-challenge solvers for four of the five Day 4 challenges
  (BRE, Metadata, Mobile, Script). No `warmups-day-04` solver directory exists; I did not trace which solver the
  Warmups ranked pass used. The ranked passes themselves ran around 2026-10-26 04:05 UTC (deadlines in the
  result files), one submission call each, 1.2 to 5.2 s end to end (5.17 s Warmups Mastery, dominated by the
  bundle download). Commit `38668151` (2026-11-01) added the second ranked reruns for Script Time Trial and
  Mastery.
- `tools/ranked/parity45.py` and `tools/ranked/_scratch/` (including `build_d04_synth.py` and `d04synth`)
  hold Day 4/5 fixtures, including hash-repaired copies of the CRLF-damaged archives.
- CLAUDE.md lines that cite this day: a retry can issue a new attempt ID, artifact and fixtures ("Day 4 Time
  Trial"), hence the variant-corpus tools; and Days 1 to 5 plain `retry` is the ranked attempt, so there is no
  free rehearsal.

## Timeline

All times 2026-10-18 UTC, from journal start stamps and the `solved_by` time in each saved `CORRECT` response.
Durations are computed from those two stamps and include the whole scripted pass.

| Mode | Window | Started | Accepted | Elapsed |
|---|---|---|---|---|
| Script Single Solve | untimed | n/a | 10:00:09 | n/a |
| Warmups Time Trial | 20 s | 10:07:36.9 | 10:07:43.1 | 6.2 s |
| Warmups Mastery | 35 s | 10:07:48.5 | 10:08:10.0 | 21.4 s |
| BRE Time Trial | 50 s | 10:08:01.3 | 10:08:08.7 | 7.3 s |
| Metadata Time Trial | 50 s | 10:08:14.2 | 10:08:18.3 | 4.1 s |
| Metadata Mastery | 80 s | 10:08:21.1 | 10:08:25.4 | 4.4 s |
| Mobile Time Trial | 50 s | 10:08:28.4 | 10:08:32.4 | 3.9 s |
| BRE Mastery | 80 s | 10:08:34.1 | 10:09:00.1 | 26.0 s |
| Mobile Mastery | 110 s | 10:08:34.8 | 10:08:40.0 | 5.1 s |
| Script Time Trial | 50 s, seven windows | 10:07:30.5 (window 1), then 10:20:50, 10:27:42, 10:29:12, 10:32:30, 10:33:58, 10:36:16 | 10:36:43.8 | 27.8 s for the winning window |
| Script Mastery | 80 s | 10:36:50.9 | 10:37:22.8 | 31.9 s |

The Script Time Trial took about 29 minutes from the first window to acceptance. The Single Solve journals of
all five lanes carry start stamps of 2026-10-08, ten days before the Time Trial and Mastery
stamps. I could not determine from the files why; the Script Single Solve was accepted on 2026-10-18, and
`progress.md` dates every Single Solve row 2026-10-18, so those start stamps are not shown as solve times.
