# Day 24 overview

**At a glance:** 7 challenges across six lanes (two in Web Exploitation), 21 modes. By provenance label: 21
Independent, 0 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Day 24 was the second day run
unattended by the release automation, and everything on it was solved by our own submissions. Theme: new
fields and new languages on top of cumulative banks (netstrings, EVTX record counts, JSP, JAR pairs), plus
two web challenges whose Mastery bundles mixed unrelated classes. Most cost came from the Time Trial and
Mastery fixtures, not from the Single Solve.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [String Theory](warmups_string-theory.md) | Concatenated netstrings (`len:data,`); per-record codec dispatch in a 118-blob Mastery | Independent | Independent | Independent |
| Metadata Forensics | [Event Horizon](metadata-forensics_event-horizon.md) | EVTX `ElfChnk` record walk; manifest-driven bundle extraction | Independent | Independent | Independent |
| Script Analysis | [Web of Lies](script-analysis_web-of-lies.md) | Inert-descriptor decoding of a JSP family (`--%>` terminator); declared `attack` wins | Independent | Independent | Independent |
| Mobile Android | [Emulator Alibi](mobile-android_emulator-alibi.md) | v1 `split` asset decode; v2 `analysis` by running the APK's own `Release.code()`; `InMemoryDexClassLoader` hop | Independent | Independent | Independent |
| Binary Reverse Engineering | [Patch Diffing](binary-reverse-engineering_patch-diffing.md) | Cumulative recap bundle; HCFG config in JAR constant pools; role-member pairs | Independent | Independent | Independent |
| Web Exploitation | [The API Has Opinions](web-exploitation_the-api-has-opinions.md) | Per-field/action authorization matrix with one uncovered cell | Independent | Independent | Independent |
| Web Exploitation | [Fetch Quest](web-exploitation_fetch-quest.md) | Loopback SSRF via a feature/secondary route pair | Independent | Independent | Independent |

Both web challenge IDs embed `curriculum_day` (14 and 15), not the release day; the leaves sit in `day24`.

### Provenance basis

I checked each label against the leaf files (no `team_solved.json` or `manual_solve.json` exists anywhere
under Day 24, and `progress.md` has no "Team solve" row for the day):

- **Single Solve, all seven:** the leaf holds `single-solve/accepted_plan.json`. None holds a `result.json`;
  the CORRECT is in the lane log (for example BRE's six calls end in `CORRECT`).
- **Time Trial and Mastery, all seven each:** `time-trial/regular/result.json` and `mastery/regular/result.json`
  read CORRECT. Ranked and rerun attempts (`ranked-N/result.json`) are further CORRECT results and are
  bragging rights only.
- **Mobile Mastery** is the one mode where the original solve is not backed by a file of its own. The two
  release-day ranked attempts never submitted (below), the regular rehearsal has `result.json` CORRECT, and
  the fresh untimed retry that followed the interpreter fix has only `reattempt-1/answers.json` and a
  `progress.md` row. The label rests on the regular `result.json` and the later `ranked-3/result.json`.

All seven challenge pages carry the same labels, so there was nothing to correct. Three factual slips in the
pages and one in `progress.md` are listed under "Discrepancies" below instead of being edited.

## Release timeline (2026-10-24, UTC)

Day 24 unlocked at 13:00:00 UTC. The watcher detected it at T+4.9 s after 231 polls, saved a 21-entry release
list (7 challenges x 3 modes) and handed off to the orchestrator with a 50-submission cap per ID+mode. Each
lane ran two independent solver candidates per challenge. Times below come from the lane logs. `status.json`
cannot be used for release day: later reruns overwrote it (its `updated` stamps are 2026-10-31 to
2026-11-01).

| Lane | Single Solve CORRECT | Regular Time Trial CORRECT | Regular Mastery CORRECT |
|---|---|---|---|
| Warmups | 13:01:01 (T+61 s) | 13:01:05 | 13:06:10 (after one fix round) |
| Mobile | 13:01:11 (T+71 s) | 13:05:25 (after one fix round) | 13:05:31 |
| Metadata | 13:01:29 (T+89 s) | 13:05:45 (after one fix round) | 13:05:51 |
| Script | 13:03:13 (T+193 s) | 13:07:30 (after one fix round) | 13:07:36 |
| BRE | 13:05:07 (T+307 s) | 13:05:18 | 13:05:38 |
| Fetch Quest | 13:05:08 (T+308 s) | 13:05:19 | 14:04:10 (three fix rounds) |
| API Has Opinions | 13:05:17 (T+317 s) | 13:06:01 | 13:36:38 (one fix round, usage-gate wait) |

- Six of seven lanes had Single Solve, Time Trial and regular Mastery CORRECT by 13:07:36, under eight
  minutes after unlock. The two web Mastery modes were the tail: API Has Opinions at +36 min and Fetch Quest
  at +64 min (the orchestrator's "all lanes finished" line is 14:04:34).
- Release-day ranked (Hall-of-Fame-eligible) Time Trial and Mastery passes ran in the same session, right
  after each regular rehearsal. Six of seven went CORRECT in one call; the exception was Mobile Mastery.
- The same automation was re-run for reruns on 2026-10-31 and 2026-11-01 (ranked passes only, all lanes
  resumed from the existing candidates, caps 20 per ID+mode and 70 once escalated). Three Time Trial leaves
  also hold a `ranked-vm-1` result from a second machine. These are speed reruns, not the release-day solve.

### Escalations, supervisor, librarian

- **Escalation to a stronger model:** one. Fetch Quest Mastery went through three fix rounds; the log records
  "stall escalation: round 2 workers -> model opus" at 13:37:35, and the lane resumed later as "escalated
  (3 fix rounds), cap 70". The log does not say which model produced the round-3 fix; the winning candidate
  was the one that emitted every answer ID (round 3, ready at 14:02, CORRECT at 14:04:10).
- **Supervisor:** none. There is no `supervisor-report.md` for Day 24 (the supervisor first ran on Day 29).
- **Librarian:** no `_run/librarian/` folder or `closeout.md` exists for Day 24. Library promotion notes are
  in the leaf journals (the Mobile and Metadata journals carry a "Library promotion" section covering all
  three modes), and the day's lessons were added to `CLAUDE.md` in the "Day 24 ..." entries.

## What the day had in common

- **The new block is a recap or a single field, and prompts are cover stories.** Patch Diffing promises a
  before/after function diff but asks only static container facts; Emulator Alibi promises defeating
  emulator checks and has none; Event Horizon promises duplicated/truncated records and every specimen was
  clean (all 19 EVTX members were single-chunk, 69,632 bytes). Read the questions first.
- **Single Solve, Time Trial and Mastery are different draws.** Script Analysis switched from named ids
  (Single Solve) to hash ids with `LYR` payloads (Time Trial, Mastery); Mobile was v1 `split` in Single Solve
  and v2 `analysis` in the others; regular, ranked and untimed-retry Mastery bundles differed (Mobile day mix,
  `payload.dex` only in a ranked draw).
- **Declared fields beat graph rules.** JSP `attack` values came from the descriptor's own field; the BRE
  resolver's `x86-64` and the accepted `AMD64` were the same fact under different spellings.
- **Mastery bundles mix classes (web).** API Has Opinions Mastery carried 3 targets (backup disclosure,
  activity-feed IDOR, the challenge's own authorization gap); Fetch Quest carried 4 (backup disclosure,
  IDOR, paren-wrapped SQLi, SSRF). Answers had to be mapped to targets by each question's `artifact` field.

### Validator and format traps

| Lane | Trap |
|---|---|
| Warmups | Record 24 is netstrings; auto-detect read it as a column transposition. A `COUNT:CHAR,` reading is byte-identical because every length was 1, so the validator never separated them. One ranked Mastery pass (117 of 118 sets accepted) lost `set45` for a reason the saved files do not show. |
| Metadata | `roster-<hex>.csv` replaced the bare roster name; a stem-equality lookup silently misses it. Per-set member resolution must use the set's own container. |
| Script | JSP closes the descriptor comment with `--%>`. JSP `attack` took the declared value (`T1059`) where graph reading differs. The Single Solve and Time Trial/Mastery ids use different schemas. |
| Mobile | The cover story is unused; `proof()` and `seal*` helpers are decoys. The ranked build can add a second-stage `payload.dex` that a regular draw never shows. |
| BRE | `d24_*` fields are static container facts. The HCFG record sits in the companion `Name$.class`, not in `Main-Class`. |
| Web | Answers were the full `proof_<hex>` string (`exact_text`/`trim`). Route slugs and ports randomize per instance. A SQLi UNION probe that checks for a literal marker is fooled when the page echoes the query. |

## Mistakes in our own tooling or process

Collected from the lane logs, journals and result files.

- **First regular Time Trial/Mastery failures on new schemas or fixtures (5 of 7 lanes).** Single Solve
  solvers were written against one specimen and failed on the first Time Trial or Mastery draw:
  - *Warmups Mastery:* the first solver only knew the new codec and crashed on records 1-23 of every set
    (13:01:08). Fixed by per-record dispatch; regular Mastery CORRECT five minutes later.
  - *Metadata Time Trial:* one candidate failed on `roster-<hex>.csv`, the other was dropped by lint for a
    missing id (`roster-filename`). Both lost to an exact-stem lookup.
  - *Script Time Trial:* the named-id extractor choked on the hash-id `LYR` payloads (`marker present but no
    HEX/B64/HB64/FRAG payload found`; the other candidate failed on a non-string value). A schema dispatcher
    was added after one fix round.
  - *Mobile Time Trial:* the Single Solve solver used only the `split` asset decoder (exits 2 on v2), and
    the other candidate invoked the interpreter CLI with the contract's `--artifact` flag, which that CLI did
    not accept. The solver was changed to call `run_code` in-process, and the CLI was taught the flag
    during library promotion.
  - *BRE Mastery:* seven of eight sets failed in one candidate with "no delivery for set 1": the per-set
    pairing put set 1's ZIP in every slot. The other candidate solved it, so Mastery was still one call. The
    resolver's pairing was fixed afterwards.
- **Web Mastery solvers assumed one target (both web lanes).** The first Single Solve solvers read only
  `targets[0]` and copied one value to every answer ID. The API Has Opinions regular Mastery was dropped by
  lint ("missing 3 ids") and recovered after fix round 1. Fetch Quest hit a 404 on `/api/clues` of the
  wrong target at 13:05:29, then lint failures in rounds 1 and 2, including the Opus round. Causes found in
  the journal: the harness drops a candidate whose answer set is incomplete (not only a crashing one);
  a "never exit nonzero" fix was needed; the round-2 candidate's `DONE` ready-marker was never created, so
  the grader re-ran the previous solver; and a SQLi arity probe was fooled by the page echoing the query.
- **Usage gate idle time.** Fetch Quest slept 1,530 s (5-hour window at 82%, gate at 80%) and API Has
  Opinions 1,460 s (84%) before spawning fix workers, which is why API Mastery landed 31 minutes after its
  Time Trial. Fetch Quest Mastery then took until 14:04.
- **Mobile ranked Mastery lost.** Set 7 of the ranked bundle was a rebuilt Day 20 "Loaded Later" APK that
  shipped `assets/payload.dex`; the interpreter raised `unsupported external invoke ...getAssets()` and no
  submission was made. The harness then issued a second ranked attempt 3 s later (13:05:41) and it drew the
  same bundle and failed identically, spending two ranked pool entries on an unfixed solver. The Mastery
  expired. We extended the interpreter (nested VM over the payload) and an untimed `retry` was CORRECT; a
  ranked rerun on 2026-10-31 later went CORRECT in 50.5 s of a 260 s window. The `payload.dex` path was only
  validator-confirmed on Day 25 (see the Mobile page).
- **Single Solve `grp:` accounting.** The Script lane log shows the last Batch `control` id submitted alone;
  nothing in the files says whether that was a deliberate holdout. BRE's journal "285 of 288 accepted" is a
  running count, not unresolved fields.
- **`status.json` is overwritten by reruns,** so release-day state for Mobile (`solver_failed`) survives only
  in the lane log.

### Discrepancies found while reconciling (not edited)

- `writeups/day24/script-analysis_web-of-lies.md` says "Three later reruns of each mode (2026-10-31 and
  2026-11-01)". The leaf holds ranked-1 (2026-10-24) plus ranked-2 and ranked-3, so there were two later
  reruns of each mode.
- `writeups/day24/web-exploitation_fetch-quest.md` says the Mastery ranked attempt was "solved in under 6 s
  locally". `mastery/regular/result.json` reads 5.88 s; the ranked-1 result is 13.65 s and ranked-2 110.74 s.
- The Fetch Quest page does not state that its regular Mastery took 4 submission calls (one per `proof-N`
  group); the API Has Opinions regular Mastery took 3.
- `progress.md` describes the Script Mastery as "12,480 fields across 7 AES-nested sets". Every
  `start_response.json` for that mode has 8 sets x 1,560 ids; the page's eight is right.

## Tooling and process changes around this day

From `git log` and `CLAUDE.md` (dates are commit author dates, local time):

- `3c8ce72b` (2026-10-24 15:47) "Add Day 24 challenge work": all seven leaves plus library updates (BRE
  `question_facts.py` role-member rules and `role-member-pairs.md`, the EVTX record-count reader and
  `gotchas-d21-d24.md` in Metadata, and the interpreter extension for Mobile) and the "Day 24 ..." paragraphs
  added to `CLAUDE.md`.
- `10a330d9` (2026-10-24 15:48) "Document stall escalation to Opus in CLAUDE.md". It points to the workflow
  skill and `orchestrate.py --escalate-model/--escalate-from`; Day 24's Fetch Quest was the escalation in
  this run (the mechanism appears in the log at 13:37, so the code predates the documentation; the commit
  history for `orchestrate.py` shows edits on 2026-10-22, 10-24, 10-25 and 10-26, and I could not determine
  from the files which one added it).
- Library changes visible in the Day 24 journals: the netstring layer and record 24 added to `codec_chain.py`;
  hex-tagged `member_by_stem` and `evtx.record_count` in the metadata toolkit; a `--%>` terminator for
  `descriptor_decode.py`; the VM's Assets/class-loader hop with an allowlist and a negative test, and a
  `--artifact` flag on the VM CLI; per-set delivery pairing in BRE's `question_facts.py`.
- Later: `9a19811e` (2026-10-26) added the escalated submission cap (20, 70 once escalated), which is why the
  October 31 and November 1 reruns report "cap 20 (70 once escalated)".

## Effort

Submission calls (regular attempt for Time Trial and Mastery), from the lane logs and result files:

| Lane | Single Solve | Time Trial | Mastery | Ranked reruns (calls) |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | TT 1-1-1-1 (+1 second-machine); Mastery 1-1-1, one `incorrect` (4 calls, 117/118), then 1 |
| Metadata | 1 | 1 | 1 | all 1 call |
| Script | 3 | 3 | 1 | TT 1-2-3, Mastery 1-1-3 |
| Mobile | 1 | 1 | 1 | TT 1-1; Mastery: 2 release-day attempts never submitted, then 1 |
| BRE | 6 | 1 | 1 | TT 1-7-7, Mastery 1-1 |
| API Has Opinions | 1 | 1 | 3 | all 1 call |
| Fetch Quest | 1 | 1 | 4 | all 1 call (TT x4 + 1 second-machine, Mastery x2) |

Ranked windows (`ranked_deadline_ms` in `timing_meta.json`): Warmups 30 s / 135 s, Metadata 50 s / 265 s,
Script 50 s / 260 s, Mobile 50 s / 260 s, BRE 80 s / 530 s, web 120 s / 120 s (the live targets' own clocks
read about 125-213 s). Ranked pool sizes were finite and small for the web and BRE Mastery modes (8 to 16 in
`timing_meta.json`); the files do not show any pool running out on Day 24. Release-day regular Time Trials
took 1.7 to 44.5 s locally and regular Mastery 5.9 to 20.2 s. Every ranked pass but Mobile Mastery landed
inside its window. Only Fetch Quest was flagged as escalated (cap 70 on resume); the highest call count in
any mode was 7, far below the 50-call release-day cap.
