# Day 23 overview

**At a glance:** 8 challenges across 6 lanes (the web lane's second day), 24 modes. By provenance label: 16
Independent, 7 Journal-recorded, 1 Local mark only, 0 Team, 0 n/a. Theme: recap bundles and decoy prompts again,
plus three live-target web challenges whose Mastery bundles mixed vulnerability classes. Most of the cost sat
in validator spellings and rubric choices (sorted order for set-valued fields, the Python/Node.js staged-chain
`attack` value, `i386` and `JAR` spellings), not in finding the technique.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Eighty-Five Alive](warmups_eighty-five-alive.md) | Adobe ASCII85 (`<~ ~>`); 113 chains of 23 records in Mastery | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Page Me Maybe](metadata-forensics_page-me-maybe.md) | SQLite header parsing inside a cumulative bundle; sorted-order set fields | Journal-recorded | Independent | Independent |
| Mobile Android | [Hook Line and Signal](mobile-android_hook-line-and-signal.md) | v1 XOR asset, then v2 `analysis` codec solved by a Dalvik interpreter (plus a JNI hop) | Local mark only | Independent | Independent |
| Binary Reverse Engineering | [Trust Issues](binary-reverse-engineering_trust-issues.md) | Day 1-23 recap bundle (276 / 1,932 fields), static extraction | Journal-recorded | Independent | Independent |
| Script Analysis | [Press Start to Decode](script-analysis_press-start-to-decode.md) | Inert descriptor decoding, new GDScript family, `LYR` layered markers | Journal-recorded | Independent | Independent |
| Web Exploitation | [Template Temper](web-exploitation_template-temper.md) | Spring SpEL SSTI; answer is a bound context variable | Journal-recorded | Independent | Independent |
| Web Exploitation | [Ask Again Later](web-exploitation_ask-again-later.md) | Second-order SQL injection (plant/replay route pair) | Journal-recorded | Independent | Independent |
| Web Exploitation | [Posted Too Much](web-exploitation_posted-too-much.md) | Mass assignment via an unfiltered NestJS DTO | Journal-recorded | Independent | Independent |

How to read the labels (each page gives the evidence). The table uses the project's label set; where a page
header still carries an older phrasing, the overview label below is the one to use.

- **Independent (all 16 Time Trial and Mastery modes).** Every one has a saved CORRECT file from our own
  graded attempt: a `ranked-N/result.json` or `accepted_plan.json`, and for Hook Line and Signal Time Trial
  also `untimed-1/result.json`. For several lanes the *original* 2026-10-23/24 attempts are journal-recorded
  only (Warmups, Metadata, BRE, Script, Web); the label rests on the later ranked reruns, as on Day 11. Page Me
  Maybe's Time Trial and Script's Time Trial/Mastery each also have reruns that failed or hit the cap (below);
  those do not change the label.
- **Journal-recorded (seven Single Solves).** The journal (and, for most, a `submission-attempt-log.json`)
  records our own submission as CORRECT, and `challenges/submission-ledger.json` logs a CORRECT response for
  that call on 2026-10-23/24, but no `result.json` was saved for any of them. Two caveats: the Posted Too Much
  Single Solve journal defers the result to "Submission diagnostics below" and that section was never written,
  so its CORRECT rests on the attempt log and ledger rather than journal prose; and the Warmups Single Solve
  journal's only mention of the later automated re-start is an appended recon header.
- **Single Solves carrying a `team_solved.json` (Warmups, Template Temper, Ask Again Later, Posted Too
  Much).** Each marker is dated 2026-11-01 with `reason: start_already_correct`, written by a later automated
  resume. Our own CORRECT call for each is in the ledger on 2026-10-23/24 (Posted 23:38, Template 00:15, Ask
  Again 00:50, Warmups 23:34), about a week before the marker, and `progress.md` has plain `Solved` rows, not
  `Team solve (not ours)`. A fresh `start` against an already-solved Single Solve reads CORRECT whoever solved
  it, so the marker does not say a teammate scored it. We kept **Journal-recorded**, not Team. The three web
  pages and Eighty-Five Alive therefore need their Single Solve cell changed from `Independent` (web) or
  `Solved (journal-recorded); a later re-start read the team's CORRECT` (warmups) to `Journal-recorded`. The
  strict reason it is not `Independent`: an attempt log we wrote, and a ledger call log, are not result files.
- **Local mark only (Hook Line and Signal Single Solve).** There is no journal and no result file in the
  leaf. What exists: the extractor and its output, a `progress.md` row (CORRECT on attempt 1), and a ledger
  entry (one call, CORRECT, 2026-10-23 23:36). The ledger is corroboration, not a journal or result file, so
  this is the weakest of the seven; the page's wording "graded result not preserved" should become `Local mark
  only`, and its "progress row only" understates the evidence slightly because of the ledger entry.

## What the day had in common

- **Cumulative recap bundles, again.** Metadata (28 fields per set, 196 in Mastery) and Binary (276 fields,
  1,932 in Mastery) re-asked earlier days' fields plus one new block (SQLite header; a managed PE with CLR
  metadata). Script's Single Solve bank was Day 22's plus 44 GDScript ids.
- **Prompts as decoys.** Trust Issues talks about clock checks and environment sniffing (every question was
  static extraction), Page Me Maybe promises truncated databases and an "impossible claim" (never asked),
  Script's "level that doesn't belong" never needed cross-corpus reasoning, and Hook Line and Signal's decoys
  (`proof()`, `client_secret`, never-called `seal*` methods) were built into the APK. Warmups was the exception:
  the riddle ("Adobe might recognize the neighbors") named the codec.
- **Time Trial and Mastery changed the mechanism in two lanes.** Mobile moved from a stored XOR answer to code
  that computes it (`codec: "analysis"`), and Script switched from named ids to hash ids with `LYR` layered
  markers. In the other lanes they changed fixtures only.
- **Web Mastery bundles mixed classes.** Each web Mastery had three targets, and in all three the same two
  non-challenge targets appeared (robots.txt to `.json.bak` backup disclosure, and an activity-feed IDOR) next
  to one target of the challenge's own class. The web challenge ids embed `curriculum_day` 11, 12 and 13, but
  the leaves sit under Day 23. Time Trial targets carried a ~2 h header clock although `start` showed no
  deadline; `_run/timing_meta.json` records a nominal 120,000 ms ranked deadline for Template Temper and Posted
  Too Much (the Ask Again Later Time Trial's entry is null).
- **New library material.** Each lane promoted what it learned: the SQLite reader into the metadata toolkit, the
  Dalvik interpreter and JNI-hop support for mobile, record 23 (`ascii85`) into the warmup codec table, the
  language-sub-technique `attack` rule for script, and web entries for SpEL SSTI, second-order SQLi and NestJS
  mass assignment. CLAUDE.md carries the Day 23 lessons for Script Mastery, Template Temper, Posted Too Much,
  Ask Again Later and the Hook Line and Signal `puzzle.v2` schema.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Script | A staged chain that acquires remotely and ends in `dispatch_process` took the language sub-technique (Python `T1059.006`, Node.js `T1059.007`), not `T1105`; this appeared only in Mastery sets 2/4/5/6, so Single Solve, Time Trial and set 1 could not confirm it. The descriptor's own `attack` beat the printed graph rule for v3 families. |
| Script | Single Solve used named ids, Time Trial and Mastery hash ids with `LYR` ops undone right to left; dispatch on the live start response, not the challenge. |
| Metadata | `zip.central_directory_member_names` and `macho.cpu_type_set` wanted ascending sorted order; central-directory/fat-table order was rejected. Earlier days' specimens had natural order equal to sorted order, so Day 23 held the first cases where they differ (the journal's reading, not a validator statement). A `.wav` was really M4A, FLAC, Ogg or WAV per set; codec names are the short ffprobe style. |
| Binary | JAR `format` takes the literal `JAR`; PE `Machine` for the .NET days takes lowercase `i386` (`I386` rejected); Day 3's compound `PE32_CLI` versus the `pe_cli` enum on Days 22/23; PE `AMD64` for d04/d10 in the ranked Time Trial; a day's specimen can be ELF in one set and PE in another. |
| Mobile | v1 `xor` versus v2 `analysis` is declared in the asset JSON; the answer is `ANS-` plus 16 zero-padded hex digits from `Release.code()`, never `proof()`. |
| Warmups | `<~ ~>` delimiters must be stripped before `a85decode`; Mastery members are chains, and `blob-100` sorts before `blob-11` lexically, so map by each question's `artifact` field. |
| Web | The Spring answer is the bound `{{objective}}` variable, not RCE output; the Sinatra injected field (`filter`) is unadvertised and route slugs randomize per attempt; the NestJS privileged field name differs per instance (bare `current_role` worked on 1 of 3), so send a combined probe. |
| All lanes | `incorrect` on a multi-answer call has no per-field feedback (only `accepted` plus `remaining` persists), and every answer value must be a string. |

## Mistakes in our own tooling or process

Collected from the challenge pages and checked against the ledger and result files.

- **Trust Issues (Binary).** A shell fallback chain around `gm.py submit` sent the same `format` submission
  twice (attempts 4 and 4b, both `incorrect`, identical payloads), burning one of the 16 Single Solve calls for
  nothing (journal; ledger shows 5 `incorrect` among those 16). Time Trial and Mastery then went CORRECT in one
  call each by reusing the validated mapper.
- **Page Me Maybe (Metadata), original run.** The two sorted-order fields cost 13 Time Trial and 16 Mastery
  calls on 2026-10-24 (ledger matches), recovered by bisection. Those are discovery costs, not slips; the
  slip is in the reruns below.
- **Page Me Maybe, ranked reruns.** The reruns repeated both traps: the untimed rerun and `ranked-1` each ended
  `incorrect` after 3 calls with a long unaccepted list, the ledger shows two `expired` entries, and a fresh
  fixture then failed on `macho.cpu_type_set` before `ranked-2` went CORRECT in one call (3.43 s). The library
  already documented both orderings; the solver had not read them. Time Trial's ledger total is 43 calls,
  Mastery's 17.
- **Press Start to Decode (Script).** (a) The Mastery mapper tested `acquire_remote` before the terminal
  `dispatch_process` rule, so 11 of 10,465 fields were wrong (8 Python, 3 Node.js); the library's reference
  `derive_attack` already had the correct branch, and diffing against it before submitting would have cost no
  calls (10 Mastery calls of a 20 cap). (b) The ranked hash-id mapper did not carry that rule over and never
  wired a v3 `attack` alternate group into a submittable variant, so ranked Time Trial `ranked-1`/`ranked-2`
  ended `cap_reached` (19 and 25 calls) and Mastery `ranked-1` at 10 calls with the cap spent, with large
  un-split batches that ran the budget out before bisection could finish; the player then raised the cap
  (`challenges/day23/submission-attempts.md`), and after the fixes `ranked-3` and `ranked-2` went CORRECT in one
  call each. Ledger totals: Time Trial 46, Mastery 21. (c) One extracted hex-payload JScript specimen hit an
  antivirus file lock (`OSError` Errno 22) on re-read; an in-memory extraction cache fixed it.
- **Template Temper (Web), ranked rerun.** The general web dispatcher's first pass tried generic parameter
  names (`q`, `input`) and missed the confirmed `template` parameter; the `ranked-1` folder holds no result and
  `progress.md` records the dispatcher fix. The ledger also shows regular-attempt resubmissions at about 22:05
  on 2026-10-31 for Template Temper and Posted Too Much, one of which returned `already_solved` (a wasted
  call); the files do not say what triggered them.
- **Single Solve markers.** The resume pass writes `team_solved.json` whenever `start` already reads CORRECT,
  even where our own earlier CORRECT is in the ledger and journal (four leaves here; see the provenance
  discussion above). It also logged "Single Solve scored by the team" in the lane logs. The marker is therefore
  not evidence of who solved a mode, which is what `CLAUDE.md` already says about Game Master status.
- **Posted Too Much (Web).** Its Single Solve journal points to a "Submission diagnostics" section that does not
  exist; the result is only in the attempt log and ledger.
- **Resume-run noise.** The `_run` orchestrator log shows the resume pass invoked five times in about 20
  seconds on 2026-11-01 (one challenge each); the first Time Trial/Mastery ranked starts for Template Temper and
  Posted Too Much returned `409 active` and were not graded. Warmups' and Ask Again Later's lane pass reported
  "no candidate solver; not issuing an attempt" for the ranked Time Trial/Mastery, so those reruns came through
  the separate ranked tool instead (Warmups has a ranked solver dated 2026-10-31).

## Release timeline, escalations and what `_run` covers

- **Coverage.** `challenges/day23/_run/` covers only the **Warmups and Web Exploitation lanes** (four
  challenges). It has no Metadata, Mobile, Script or Binary lane logs, no supervisor report, no librarian notes
  and no `closeout.md`, so nothing below about those four lanes comes from `_run`.
- **What `_run` shows.** It does not describe the original Day 23 solve; its files date from the ranked
  gap-closure sweep (committed 2026-10-31 22:12 and 2026-11-01 02:05, local). Orchestrator log: resume passes at
  05:00-05:07 UTC on 2026-11-01 produced Template Temper ranked Time Trial CORRECT (17.7 s, 1 call, 131 s window)
  and Mastery CORRECT (28.38 s, 3 calls, 140.7 s window), and Posted Too Much ranked Time Trial CORRECT (11.2 s,
  1 call, 125 s window); a 06:46 UTC pass covered Warmups and Ask Again Later and issued nothing. `status.json`
  records `ss: team-solved` for all four, which is the marker discussed above. No Posted Too Much Mastery
  appears in `_run`; its ranked Mastery results came from a separate run on a second machine and the earlier
  `ranked-1`.
- **No release-to-finish timeline.** No file states the unlock time or per-lane finish times for the original
  run, and there are no lane logs for it. The best reconstruction is the ledger's first CORRECT per original
  attempt (the main machine's local clock, 2026-10-23/24): Warmups Single Solve 23:34 and Time Trial 23:35 and
  Mastery 23:36; Hook Single Solve 23:36; Metadata Single Solve 23:37; Posted Single Solve 23:38; Script Single
  Solve 23:47; then (10-24) Posted Time Trial 00:13 and Mastery 00:16; Template Single Solve 00:15, Time Trial
  00:17, Mastery 00:19; Hook Time Trial 00:24; Script Time Trial 00:28; Metadata Time Trial 00:28; BRE Single
  Solve 00:29 and Time Trial 00:31; Hook Mastery 00:33; BRE Mastery 00:36; Metadata Mastery 00:37; Ask Again
  Single Solve 00:50, Time Trial 00:51, Mastery 00:53; Script Mastery 00:56. That is about 82 minutes from the
  first to the last CORRECT, with the commit "Add Day 23 challenge work" following at 01:11 on 2026-10-24.
  Those are call times, not unlock-to-solve durations.
- **Escalation to a stronger model.** None is recorded: no Day 23 lane log, journal or result mentions one. The
  stall-escalation procedure was documented in `CLAUDE.md` on 2026-10-24, after these solves.
- **Supervisor and librarian.** There is no supervisor report (the supervisor first ran on Day 29) and no
  librarian notes or `closeout.md` for Day 23. Library promotions are in the journals' "reusable knowledge"
  sections and the entries named on each page.

## Effort

Submission calls on the original 2026-10-23/24 attempts (journals, attempt logs and the ledger agree), then the
ranked reruns that went CORRECT (from `result.json`):

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Ranked reruns that went CORRECT |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 (113 answers) | 2 TT, 2 Mastery |
| Metadata | 1 | 13 | 16 | 1 TT (after two failed reruns), 1 Mastery |
| Mobile | 1 | 1 | 1 | 1 TT (plus an untimed rerun), 1 Mastery |
| Binary | 16 | 1 | 1 | 1 TT, 1 Mastery |
| Script | 1 | 1 | 10 | 1 TT (after two `cap_reached`), 1 Mastery (after one) |
| Template Temper | 1 | 1 | 1 | 2 TT, 2 Mastery |
| Ask Again Later | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Posted Too Much | 1 | 1 | 1 | 2 TT, 2 Mastery |

Ranked rerun windows from the results were 30 s (Warmups TT), 130 s (Warmups Mastery), 50 s (Metadata, Mobile
and Script TT), 230 s (their Mastery), 80 s and 465 s (Binary), and roughly 125-141 s for the web runs recorded
in `_run`. Local totals ran from 1.5 s to 30 s. Ranked reruns were a bragging-rights sweep, not the prize
ranking.

## Tooling and process changes around this day

- `cedbe666` (2026-10-24): the Day 23 challenge work (journals, attempt logs, mappers, the Dalvik interpreter,
  the Template Temper and Posted Too Much exploit scripts) as one commit shortly after the last CORRECT.
- `10a330d9` (2026-10-24): stall escalation to Opus documented in `CLAUDE.md`, the day after these solves.
- `1160dffa`/`1df4ec2c`/`c9906b50` (2026-10-25): ranked-attempt tooling and the first ranked passes; Day 23 was
  not part of those (its ranked reruns came on 2026-10-31).
- `021814b6` and `df8f4534` (2026-10-31): the Day 23 `submission-attempts.md`, the first Day 23 ranked results
  (Warmups) and the ranked Warmups solver. `ac51598a` (2026-10-31) extended the general web dispatcher and added
  the `_run` files for the web lanes. `6af31628` and `38668151` (2026-10-31 and 2026-11-01) carried the rest of
  the gap-closure reruns; `f0e20aa1` (2026-11-01) added Warmups `ranked-2`.
- The ranked-sweep fixes recorded in `progress.md` and `challenges/day23/submission-attempts.md`: the
  per-day Script solver (`tools/ranked/solvers/script-analysis-day-23-game-resource-loader/`) with the ported
  language-sub-technique rule and the in-memory extraction cache, the Metadata solver's use of the library's
  pre-sorted `cpu_type_set`, and the BRE solver as a thin wrapper over the library's fact resolver (0 of 276 and
  0 of 1,932 mismatches against the accepted maps before any live call).
- `CLAUDE.md` now records the Day 23 lessons listed under "What the day had in common".
