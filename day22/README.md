# Day 22 overview

**At a glance:** 7 challenges (one per non-web lane, plus two Web Exploitation challenges), 21 modes. By
provenance label: 20 Independent, 1 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Day 22 was the first
run where release automation ran unattended and the first with the web lane included. Theme: every lane added
only a small new piece (a codec, a field, a file family, a 12-field block, two reskinned web bugs), so the
cost was in our own tooling: two bugs in the submission layer, a web solver that assumed one target, and one
spelling (`i386`) that the automation could not reach.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Deflated Ego](warmups_deflated-ego.md) | Base64 over zlib; per-record codec numbering in Mastery (record 22) | Independent | Independent | Independent |
| Metadata Forensics | [Main Character Energy](metadata-forensics_main-character-energy.md) | `jar.main_class` (manifest vs. class files); 27-field recap bundle | Independent | Independent | Independent |
| Mobile Android | [Root of Trust](mobile-android_root-of-trust.md) | Asset-codec decode (base64); root-signal theme is dead code | Independent | Independent | Independent |
| Binary Reverse Engineering | [Bridge Troll](binary-reverse-engineering_bridge-troll.md) | 264-field recap bundle; new managed .NET block with HCFG record; `i386` spelling | Journal-recorded | Independent | Independent |
| Script Analysis | [Next, Next, Finish](script-analysis_next-next-finish.md) | Inert descriptors in a new NSIS family; declared descriptor wins | Independent | Independent | Independent |
| Web Exploitation | [Forged With Care](web-exploitation_forged-with-care.md) | Session-cookie-as-CSRF-token behind `SameSite=Lax`; mixed-skin Mastery | Independent | Independent | Independent |
| Web Exploitation | [Uploading Expectations](web-exploitation_uploading-expectations.md) | Upload content-type bypass into `{{field}}` template; mixed-skin Mastery | Independent | Independent | Independent |

The two web challenge ids embed `curriculum_day` 9 and 10 (`web-exploitation-day-09-...`, `-day-10-...`); they
were released on Day 22 and live in `challenges/day22/`.

How to read the labels (each page gives its evidence):

- **Independent (20 modes).** Each leaf holds our own graded `result.json` (CORRECT) or `accepted_plan.json`
  for the mode, and for Time Trial and Mastery also `ranked-N/result.json` files. Two web Mastery leaves need
  a note: `mastery/regular/result.json` was overwritten by a later re-issued attempt and now reads `None`
  (incorrect, 4 calls), but the regular `accepted_plan.json` and ranked CORRECT results remain, and the ledger
  shows the release-day regular Mastery as CORRECT in 1 call for both challenges (06:14 local).
- **Journal-recorded (Bridge Troll Single Solve).** See the next section.

### Bridge Troll Single Solve: why Journal-recorded, not Team

The leaf has a `team_solved.json` (reason `start_already_correct`, written by a resume run) and
`_run/status.json` says `team-solved`. The rest of the local evidence says the original CORRECT was ours:

- The lane log's first submission (2026-10-22 13:04:46 UTC) returned a graded `incorrect`, followed by a bisection
  that returned `accepted` for 262 of 264 fields. A Single Solve already scored by someone else answers
  `already_solved`, not graded results, so nobody had solved it at that point.
- `challenges/submission-ledger.json` has 35 entries for the ID. Entry 35, tagged `FIX machine try`
  (2026-10-22 09:34:09 local), is `CORRECT`, immediately after the isolated `0x14c` and `332` rejections. The
  journal's "RESOLVED" section says the same, and the progress row is a hand-written "Solved" row dated 2026-10-22.
- The lane log at 2026-10-22 16:37 UTC reads "resume: Single Solve already CORRECT", before any later resume.
- The `team_solved.json` marker comes from the resume logic: from 2026-10-31 it reads a `start` that returns
  CORRECT and writes "scored by the team", which cannot tell our own earlier solve from a teammate's. The file
  records only "seen 2026-11-01T04:13:53Z"; it does not say who solved it. `status.json` is rewritten by every
  resume run, so it reflects the last resume, not release day.

So the evidence supports our own solve. It is Journal-recorded rather than Independent because no result file
for the CORRECT call was saved: the saved `fix-submit*.log` files end before the last call, and the ledger is a
call log (same reasoning as the Day 11 overview). The Bridge Troll page's label is correct as written.

## Timeline (2026-10-22, UTC; local was UTC-7)

From `watch.log`, `orchestrator.log`, the lane logs and the ledger. Unlock was 13:00:00.

| Time | Event |
|---|---|
| 12:50 / 12:55 | Watcher up; polling window opens at T-300 s. |
| 13:00:03.65 | Day detected (21 mode rows, 7 challenges) after 220 polls; orchestrator starts with cap 20. |
| 13:00:06-13:00:21 | Recon done on all lanes (T+2.6 s to T+17.6 s; the web lane was slowest). |
| 13:01:11 | Mobile Single Solve CORRECT (about 71 s after unlock). |
| 13:01:33 | Warmups Single Solve CORRECT; 13:02:10 Metadata; 13:03:26 Script (1,417 fields, 1 call). |
| 13:04:46 | BRE first 264-field submission `incorrect`; 20/20 calls used at 13:04:51 (`cap_reached`, 262 of 264 accepted). |
| 13:06:46 / 13:06:57 | Forged With Care / Uploading Expectations Single Solve CORRECT (about 6.8-7 min; the first candidates were ready at 13:04:42 and 13:04:53, and the lane log shows a "web target refreshed" step before the first submission). |
| 13:06:27 | Warmups regular Mastery CORRECT after record 22 was added (first attempt failed closed at 13:01:39). |
| 13:07:14 | Metadata regular Mastery `cap_reached` (20 calls, 120 of 162 accepted). |
| 13:14:17 / 13:14:33 | Web regular Mastery CORRECT for both challenges, after a first crash at 13:07. |
| 13:14:44-13:14:56 | Web ranked Mastery: `solver_failed` twice per challenge, before any submission. First pass ends; 17 of 21 modes CORRECT. |
| 16:31-16:34 | BRE: player-authorized extension (cap 50); isolated `d22_analysis_machine`, `i386` CORRECT at 16:34:09 (about 3 h 34 min after unlock). |
| 16:37-16:38 | BRE Time Trial (6 calls) and Mastery (1 call) regular CORRECT, then ranked CORRECT. |
| 16:42-16:47:50 | Metadata: second extension (cap 50), extractor and bisection fixed, regular Mastery CORRECT at 16:47:50 (call 29). Last first-CORRECT of the 21 modes, about 3 h 48 min after unlock. |
| 16:48-16:49 | Metadata ranked Time Trial (6 calls) and Mastery (1 call) CORRECT. |
| 21:28-21:36 | Web lanes resumed (cap 50): fix round 2, ranked Time Trial rerun CORRECT, ranked Mastery still blocked (`attempt_active` 409, then waiting for a live target). |
| 2026-10-31 to 11-01 | Ranked reruns and the web Mastery hand work (below). |

Regular (untimed) CORRECT on every Single Solve, Time Trial and Mastery was therefore in hand within about 3 h 48 min
of unlock, and 17 of 21 modes within 15 minutes. These are unlock-to-first-CORRECT times for one release day;
the prize ranking (time to our own CORRECT, once per mode) was defined later, from Day 30.

### Ranked windows (`timing_meta.json`)

Time Trial / Mastery ranked deadlines: Warmups 30 s / 125 s; Metadata 50 s / 200 s; Script 95 s / 200 s;
Binary 80 s / 400 s; Mobile 50 s / 200 s; both web challenges about 120 s for each (observed 124-128 s).
Ranked pools were 8-16 for web, 16-64 for Binary, 32 for most others and 256 for Warmups.

## What the day had in common

- **Small new piece, familiar frame.** Warmups: a base64-zlib codec. Metadata: one JAR field. Script: one new
  host language (NSIS) with the existing four marker encodings. Binary: one 12-field block on top of a
  22-block recap. Mobile: base64 asset codec. Web: two bugs already in the library classes.
- **Single Solve is a weak gate for the one-answer lanes.** Warmups, Metadata and Mobile Single Solve have one
  answer, so passing it says nothing about Mastery. Warmups' record 22 and Metadata's new
  `roster-magic-prefix-hex-16` field showed up only in Mastery.
- **Decoy prompts.** Root of Trust (no root detection to bypass), Bridge Troll (no P/Invoke question),
  Next, Next, Finish (no wizard), Uploading Expectations (traversal hint, real bug is a content-type
  decision). Warmups ("inflation" = DEFLATE) was the exception.
- **Mastery bundles are heterogeneous.** Mobile Mastery mixed earlier-day builds (Days 1, 6/7, 12, 13, 16/17/21 and
  22 across attempts); both web Mastery bundles drew three skins from a larger pool that changed per attempt;
  Binary Mastery was six copies of the 264-field bundle (1,584 ids); Script Mastery six copies of 1,417 (8,502).

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Binary | `d22_analysis_machine` took lowercase `i386` only; `I386`, `0x14c` and `332` were rejected in the same call where d03's `I386` was accepted. |
| Metadata | New `roster-magic-prefix-hex-16` has to read `roster.csv`, resolved from the question id; the BOM field is `true`/`false`, never empty. Zero-padded hex, braced MSI codes and angle-bracketed Message-IDs all held. |
| Script | NSIS declared `attack`/`role`/`adapter` beat the printed graph rule (benign utility declared `none` despite a `write_file>dispatch_process` graph; launcher `T1059`; staged chain parent `T1059`). Batch/VBScript controls change per fixture. |
| Warmups | The slug is the last line; Mastery needs record 22 (`base64-zlib`) in the record table, and answers map by artifact name. |
| Mobile | Answer is only in `assets/huntress_puzzle.json`; `rootSignals(ZZZ)I` has no caller. Older-day builds in Mastery need older codecs. |
| Web | Proof strings go in verbatim (`exact_text`). A Mastery id-to-skin mapping is not stable across attempts; the privileged route or field differs per skin. |
| All | `incorrect` on a composite call carries no per-field feedback; every value must be a string. |

## Escalation and supervision

- **Escalation to a stronger model.** The two web lanes ran fix rounds up to round 2 (worker logs for `r2` exist,
  dated 14:30 local, but are empty). Their later resume logs call them "escalated in an earlier run (2 fix rounds)"
  and apply the 70-call cap. The Opus escalation policy in `CLAUDE.md` is dated 2026-10-24, two days after this
  run, and no Day 22 file names a model, so we cannot say Opus ran on Day 22. Warmups and Metadata had one fix
  round (`r1`); Binary, Mobile and Script ran only round 0. BRE and Metadata Mastery were closed interactively
  after the player authorized extra calls (cap 50), not by escalation.
- **Supervisor, closeout, librarian.** None for Day 22: there is no `supervisor-report.md`, `closeout.md` or
  librarian folder in `_run/` (the supervisor first ran on Day 29, automatic closeout from Day 25). The one
  post-run write-up is the hand-written "Lessons from the Day 22 live run" section in the release tooling's
  README. The Script page notes the same absence.

## Effort

Submission calls to first CORRECT on the regular (untimed) route, from lane logs and the ledger:

| Lane | Single Solve | Time Trial | Mastery | Notes |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | Mastery's first attempt failed closed (0 calls) |
| Metadata | 1 | 1 | 29 (20 automated, 9 manual) | Ledger entry 29 is the first CORRECT; the Metadata page says 31, which counts later ranked calls in the same ledger |
| Script | 1 | 1 | 1 | |
| Binary | 35 (20 + 15 under the extension) | 6 | 1 | 35 of an approved 50 |
| Mobile | 1 | 1 | 1 | |
| Forged With Care | 1 | 1 | 1 | |
| Uploading Expectations | 1 | 1 | 1 | |

Ranked and rerun attempts (speed board only, not the prize ranking): the leaves hold `ranked-N/result.json` for all
14 Time Trial and Mastery modes (`ranked-only.txt`), from release day, a full-day rerun on 2026-10-31 and
per-lane reruns on 2026-11-01, plus a few from a second machine (`ranked-vm-*`). Counts of CORRECT ranked results:
Warmups 4 TT + 4 Mastery (one more Mastery `incorrect`, 13 calls, `set25`), Metadata 4 + 3 (one TT
`cap_reached`), Script 5 + 3, Mobile 4 + 3, Binary 3 + 2, Forged With Care 2 + 2, Uploading Expectations 2 + 2.

## Mistakes in our own tooling or process

- **Declared alternates dropped when candidates agree (Bridge Troll).** `merge_plans()` only built an alternate
  group for ids where the two solvers disagreed. The rubric solver had declared `i386` for the machine field, but
  both solvers answered `I386`, so the right reading was never tried. A one-field spelling cost 15 extra
  Single Solve calls and a player-authorized extension. Fixed in `rl.py`: self-declared groups are kept even
  when the primary answers agree (BRE journal, dated 2026-10-22; release README).
- **Premature variant bump (Main Character Energy Mastery).** On a composite failure `submit_plan()` moved every
  group straight to variant 1 before testing variant 0 alone, and this API returns no partial `accepted_ids` on
  `incorrect`. One unrelated bad field therefore made six correct primaries look wrong and discarded them. Fixed
  (variant 0 now always gets its own call). The README warns earlier days may have skipped variant-0 tests the
  same way; that was not re-checked.
- **A wrong-file hard-code hid in a bare branch (Metadata).** `*-magic-prefix-hex-16` was answered from
  `evidence-note.txt` for every file; the hex decoded to readable `Case FA695E7CC3F...`, which was the tell. The
  adapter solver instead crashed on every set at this field, so only the rubric solver's plan was submitted and no
  second reading existed to compare. The Mastery journal first blamed the padded-hex rubric rules; that was wrong.
  Mastery used 20 automated calls (120 of 162 accepted) and 9 more by hand.
- **Bisection log kept counts, not ids (Bridge Troll).** The first bisection recorded only `n=` per chunk, so the
  two unresolved ids could not be read off the log, and an offline re-verification of all 12 new fields (which
  were right) pointed away from the machine field. The journal recommended logging id lists; we did not check
  whether that was implemented.
- **Web solvers assumed one target (both web lanes).** The round-0 solvers read `targets[0]`/`answer_ids[0]`. That
  was right for Single Solve and Time Trial and crashed with a 404 on the three-target Mastery (13:07), then
  `solver_failed` twice on the ranked Mastery (13:14). Fixes: resolve each `proof-N` through the question's
  `artifact` to its target, and dispatch on structural signals per target. The unattended run therefore did
  not finish the ranked web Mastery.
- **Web skin coverage was never complete.** Later draws brought skins the solvers did not know, so on 2026-10-31
  the Forged With Care ranked Mastery needed a hand-recovered field: the "Pocket SOC" IDOR proof (`/api/activity`
  and `/api/incidents/<id>`), recovered by hand before the live target idled out and submitted directly; the PHP
  LFI field was closed by adding a handler and a direct call (ledger entries tagged `close-proof1-lfi` and
  `proof2-pocketsoc-idor`). The solver's feed and detail-route lists were then broadened and a later ranked draw
  resolved it automatically. Uploading Expectations went through a placeholder answer (`unresolved_lfi_target`), an
  `attempt_active` 409, and added LFI/mass-assignment skins before its ranked Mastery went CORRECT.
- **Warmups: fail-closed, but two solver faults.** Record 22 was missing from the record table (cost no calls, about
  5 minutes). The second solver called a nonexistent helper and returned one slug for every set. A later ranked
  Mastery rerun was `incorrect` on `set25` only (13 calls); one solver variant reported no slug for that set and
  we could not determine why.
- **Script: undiagnosed JScript lint failures.** The adapter solver exited with lint errors on JScript `relay`
  and `impact` specimens in some Mastery sets; the second solver covered them.
- **Ranked results credited to a solo account.** Release-day ranked CORRECTs for Warmups Mastery, Metadata Mastery
  and both web Time Trials landed on a solo account, not the team, on the public speed board. We found it in the
  2026-11-01 sweep and reissued ranked attempts under the team identity. The ledger cap then blocked some reruns
  (Metadata Time Trial `cap_reached` at 20, Mastery "ledger 31 >= cap 20; not issuing"), as designed.
- **Automation records misread by later readers.** `status.json` and `team_solved.json` mislabel Bridge Troll's
  Single Solve (above), and `status.json` shows `cap_reached`/`solver_failed` for ranked modes because every resume
  overwrites it; neither is the release-day state.

## Tooling and process changes around this day

The repository history begins 2026-10-22; what it shows:

- `1247a44f` (2026-10-23): "Add Day 22 challenge work" (leaf journals, champion files and the lane logs for the first
  run). Release automation itself first ran live on Day 21 and this was its first full six-lane run.
- The post-run fixes to `tools/release/rl.py` (merge keeps self-declared alternates; no premature variant bump)
  are described in the release README's "Lessons from the Day 22 live run". The journals date the merge fix to
  2026-10-22; we did not locate a separate commit for either.
- `10a330d9` (2026-10-24): `CLAUDE.md` gains the stall-escalation-to-Opus procedure, after this run.
- `021814b6` (2026-10-31): ranked reruns including "Day 21/22/24 team reruns"; it adds the 10-31 lane log lines
  and ranked results. `38668151` and `f0e20aa1` (2026-11-01) continue the gap-closure sweeps, and `f0e20aa1`
  carries the cap-reached and webhook-dispatch fixes for other days.
- Library entries the pages cite and that this day's web and BRE work fed: the upload-bypass template-field class
  (WEB-UPLOAD-001), the PHP LFI class (WEB-LFI-001), the warmup codec table, and the BRE machine-label ruling
  (lowercase `i386` for Days 22-24).
