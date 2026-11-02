# Day 25 overview

**At a glance:** 8 challenges across 6 lanes (three of them Web Exploitation), 24 modes. By provenance label: 22 Independent, 2 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a.
Theme: prompts that promise more than the questions ask (a bounds-error proof, an integrity-statement forgery, a
regrouping of registry hives), one new decoder or reader per lane, and a Web lane whose Mastery bundles again
mixed unrelated vulnerability classes. Six lanes were finished by the release automation within about 12
minutes of unlock; the two web challenges whose recon hit a 503 were finished by hand about 11 hours later.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [You Have a Point](warmups_you-have-a-point.md) | `U+XXXX` Unicode code point list; new codec added before submitting | Independent | Independent | Independent |
| Metadata Forensics | [Hive Mind](metadata-forensics_hive-mind.md) | `regf` root key name from the header's root-cell offset, not the embedded file name | Independent | Independent | Independent |
| Mobile Android | [Integrity Signals](mobile-android_integrity-signals.md) | v1 `hex` asset (Single Solve); running the APK's own `Release.code()` in a Dalvik interpreter (TT/Mastery) | Independent | Independent | Independent |
| Script Analysis | [Ashes to Assets](script-analysis_ashes-to-assets.md) | Elixir/Phoenix descriptor decoding; declared descriptor fields win | Independent | Independent | Independent |
| Binary Reverse Engineering | [Out of Bounds](binary-reverse-engineering_out-of-bounds.md) | Cumulative recap bundle; vulnerable/corrected JAR pair, HCFG record in a companion classfile | Independent | Independent | Independent |
| Web Exploitation | [Shell Company](web-exploitation_shell-company.md) | OS command injection via an unsanitized `path` parameter | Journal-recorded | Independent | Independent |
| Web Exploitation | [Reset Expectations](web-exploitation_reset-expectations.md) | Password-reset token not bound to the account (account_id IDOR) | Independent | Independent | Independent |
| Web Exploitation | [Resolver Revolt](web-exploitation_resolver-revolt.md) | GraphQL field-level authorization gap (single-item resolver skips the list's check) | Journal-recorded | Independent | Independent |

Web challenge IDs embed `curriculum_day` 16, 17 and 18; all three leaves sit under `challenges/day25/`.

How to read the labels:

- **Independent (22 modes).** The leaf holds our own `accepted_plan.json` and/or `result.json` with CORRECT. For
  the five static-artifact lanes and Reset Expectations that includes the 2026-10-25 original attempts
  (`accepted_plan.json` in the Single Solve leaf, `regular/` and `ranked-1/` results for Time Trial and Mastery).
  For Shell Company and Resolver Revolt the Time Trial and Mastery labels rest on the later `ranked-N/result.json`
  files (2026-10-31 to 2026-11-01); the original 2026-10-25 attempts of those two are journal-recorded and each
  has one CORRECT call in `challenges/submission-ledger.json`.
- **Journal-recorded (Shell Company and Resolver Revolt Single Solve).** Each journal records our own submission
  as CORRECT on the first attempt, `progress.md` has a hand-written `Solved` row ("CORRECT on attempt 1", not an
  auto-recorded row), and the ledger holds exactly one Single Solve call for each, result CORRECT (16:59 local on
  2026-10-25, two seconds apart). No `result.json`, `accepted_plan.json` or `manual_solve.json` was saved.
  Both leaves also hold a `team_solved.json` (`reason: start_already_correct`, written 2026-11-01 by a later
  automation pass). We did not label them Team: the marker comes from reading the `start` response, which cannot
  tell our own earlier solve from a teammate's, that pass submitted nothing, and our only Single Solve
  submission was graded CORRECT rather than `already_solved`. We did not label them Independent either, because
  no graded result file exists. **Page discrepancy:** the Shell Company and Resolver Revolt pages label Single
  Solve "Independent" (Resolver Revolt: "Independent (journal-recorded...)"); under the brief's definitions both
  should read Journal-recorded. A `manual_solve.json` could upgrade them, but only the player can say the
  evidence in the journal and ledger is enough.

Ranked (speed-board) attempts were bragging rights and did not count toward the prize ranking; the ranked times
quoted below describe our runs, not scoring.

## Timeline (unlock 2026-10-25 13:00:00 UTC)

From `watch.log`, `orchestrator.log` and the lane logs (UTC). `status.json` was overwritten by the later rerun
passes (its ranked times are the 2026-11-01 reruns), so it does not preserve this timeline.

- 12:55 watcher up; **13:00:04.2** day 25 detected after 227 polls (T+4.2 s); 24 listed entries, 8 challenges in
  6 lanes, submissions authorized with a cap of 50 per ID+mode. Recon succeeded on six lanes by T+4.4 s.
- Single Solve CORRECT: Warmups T+1:31 (1 call), Metadata T+2:09 (1), Mobile T+2:09 (1), BRE T+4:06 (6 calls),
  Script T+4:06 (7 calls), Reset Expectations T+5:26 (1 call; the live target was `preparing` until T+90 s).
- Lane finished (regular and first ranked Time Trial and Mastery all CORRECT): Warmups T+2:08, Metadata T+2:25,
  Script T+4:32, BRE T+5:13, Mobile T+10:41, Reset Expectations T+12:28. **13:12:27: "all lanes finished".**
- Shell Company and Resolver Revolt: the first `start` returned 503 (T+1:28 and T+1:38) and the lane made no
  second attempt; the closeout lists them as "not CORRECT yet". The ledger (local time, UTC-7: the same Reset
  Expectations call reads 06:05:25 there and 13:05:25Z in the lane log) shows the hand-submitted Single Solves at
  16:59, Time Trials at 17:00 and Mastery at 17:04-17:05, roughly 11 hours after unlock. The journals do not say
  when the exploits were found.
- Librarian passes ran 13:12:30-13:30:42 for the six finished lanes (128-251 s each); the two web lanes were skipped
  as not CORRECT. Their library entries (OS command injection, GraphQL authorization gap) were written from the
  journals and landed in the same repository commit as the rest of the day (`2e99c6a5`, 2026-10-25).
- Later passes of the same automation, all ranked-only reruns: 2026-10-31 23:09Z (16 Time Trial/Mastery modes, list
  in `ranked-only.txt`), 2026-11-01 04:28Z (BRE Time Trial), 04:29Z (Warmups Mastery), 05:08Z (the two web
  Time Trials) and 06:48Z (seven lanes, BRE excluded).
- No `supervisor-report.md` exists for Day 25 (the supervisor first ran on Day 29).

Ranked windows (`timing_meta.json`): Warmups 30 s / 140 s, Metadata 50 s / 295 s, Script 50 s / 290 s, Mobile
50 s / 290 s, BRE 80 s / 595 s (Time Trial / Mastery); the three web challenges nominally 120 s for both
(observed 121.8-126.6 s for Time Trial, 131.6-159.0 s for Mastery). Every non-web ranked pass finished in a few
seconds to under a minute locally; the slowest were Script and Warmups Mastery reruns (43.9 s and 56.7 s).

## Escalation to the stronger model

One lane escalated: **Mobile (Integrity Signals).** Fix round 1 repaired the Time Trial solvers and fix round 2
(13:04:25, regular Mastery) ran both solvers on Opus; Mastery went CORRECT at 13:10:30 and the lane resumed later
with the higher cap (70) because it had escalated. Reset Expectations needed a fix round 1 on the regular Mastery
without escalating. No other lane needed a fix round.

## What the day had in common

- **Prompt versus questions.** Out of Bounds promised a bounds-error proof and a reachability trace (every
  question was a static recap field), Integrity Signals talked about forging an attestation (the APK holds a
  puzzle), and Hive Mind talked about regrouping systems and isolating a planted hive (one root-name field per
  hive). Only Warmups was literal ("a point" is a code point). Reading the question text before the prompt
  settled each of these.
- **Reuse over new analysis.** Four of five static lanes were solved by a previous day's resolver: the Day 24
  JAR role-pair block, the Day 24 descriptor mapper (plus two table entries for Elixir), the existing metadata
  reader (plus a registry-hive reader added during the Single Solve window), and the shared warmup codec tool
  (plus one new codec). Only Mobile needed real new work, and it came from Mastery, not Single Solve.
- **Single Solve is the narrow case.** Integrity Signals' Single Solve was a stored v1 `hex` asset while Time Trial
  and all nine Mastery sets were v2 `analysis` builds; Hive Mind's Single Solve asked one field while TT/Mastery
  asked 30 per set; Ashes to Assets' Single Solve used named IDs while TT/Mastery used hash IDs with layered
  markers. A mapper built from the Single Solve alone would have failed the later modes in three lanes.
- **Web Mastery bundles mixed classes again.** Each web Mastery delivered four targets and only one matched the
  challenge's own class. The 2026-10-25 bundles for all three challenges carried the same three extra classes
  (robots.txt to `.json.bak` backup disclosure, activity-feed IDOR, paren-wrapped SQLite UNION injection). The
  later ranked bundles differed: `progress.md` records a mass-assignment, a reflected-XSS callback and a
  stored-XSS moderator-bot target in Shell Company's, and a stored-XSS moderator-bot target in Resolver Revolt's.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | The `regf` base block's embedded file name is a decoy (differs from the real root key in all 21 hives seen); the answer is the `nk` the root-cell offset points at, as stored. |
| BRE | The `d25` block copies Day 24's questions; PE machine fields were accepted as `AMD64` where our resolver emits `x86-64` (equivalent under the pattern format). Planner variant lists were never exercised: variant 0 was accepted everywhere. |
| Script | Declared descriptor fields win over the step graph even when self-inconsistent; Elixir `attack` is the bare `T1059`; dispatch the mapper on the question-ID schema, not the day. |
| Mobile | `client_secret`, `proof()` and unexecuted `seal*` helpers are decoys; a gate-failed native export returns 0 yet still yields a well-formed `ANS-` string. |
| Warmups | Auto-detection silently fails on a new encoding; every existing layer rejected the `U+` prefix, so the codec had to be added before the first submit. |
| Web | The full `proof_<hex>` string was accepted as returned; the bare-hex alternate was carried but never needed. A GraphQL endpoint can be POST-only (a `GET` 404s). The moderator-bot target in a Mastery only fired for us on timed attempts (see below). |
| All lanes | `incorrect` on a multi-answer call has no per-field feedback; submitting in groups (base set, then risky groups) isolates a wrong field, at the cost of ledger calls. |

## Mistakes in our own tooling or process

Collected here from the challenge pages and the logs.

- **Integrity Signals (Mobile).**
  - Both Time Trial solvers on the first regular attempt read an `{"answers": {...}}` wrapper, but the Dalvik VM
    writes a flat `{id: value}` map, so neither resolved the answer ID (13:02:11, nothing submitted). The
    contract between solver and VM output was not checked; fix round 1 cost about two minutes.
  - The VM lacked `packed-switch`, `Class.forName` for classes in the same DEX and a branching two-argument
    x86-64 native export, which failed Mastery sets 6 and 7 (regular attempt only). That took the Opus round and
    about six minutes. Those additions were later validated by CORRECT results; the `payload.dex` loader hop
    first ran on a graded attempt here.
- **Hive Mind (Metadata).** One of the two solvers was a single-field root-name reader. On Time Trial and every
  Mastery set it failed with `unsupported field` for the other 29 fields, in every run on 10-25, 10-31 and
  11-01. The library-based solver carried every graded submission, so nothing was lost, but the second solver
  never served as the independent check it was meant to be.
- **Shell Company and Resolver Revolt (Web).**
  - The recon `start` 503 was treated as a final failure; the attempts went `cleaned`, the lanes never solved, and
    the Single Solves, Time Trials and Mastery attempts had to be run by hand. On 2026-10-31 the automation then
    found "no usable candidate" for these two lanes' Time Trial because none had been produced in-run, so ranked
    reruns needed a general web dispatcher (`master_web_solve.py`, per `progress.md`).
  - The automation's `start_already_correct` handling relabelled our own Single Solve as "team-solved" in
    `status.json` and in `team_solved.json`; see the provenance note above.
  - **Mastery stored-XSS target (route template).** On the 2026-10-31 Shell Company ranked Mastery the dispatcher's
    stored-XSS path guessed the note detail URL and missed the `receipt_route_template` field some skins use, so
    3 of 4 targets were left unresolved and an untimed attempt remained active (`attempt_active`) behind the
    failed lint; the fix polled the template and the moderator data route, and the blocking attempt was closed
    with the fixed solver. For Resolver Revolt `progress.md` says its first two timed Mastery retries still left
    `proof-3` unresolved after that fix, and that a long untimed practice attempt never saw the moderator bot
    visit, which it took to mean the bot only runs on timed attempts; the third ranked attempt resolved all
    four. The files do not establish why the first two timed retries failed. The Resolver Revolt page says the
    route-template mismatch cost those two attempts; `progress.md` places that bug on the Shell Company run and
    says it was already fixed, so that sentence should be softened.
- **Ashes to Assets (Script) and Out of Bounds (BRE): grouped submissions on reruns.** The first regular and
  ranked attempts went in as one call each, but later ranked Time Trial reruns went in as base set plus
  per-group calls (Script 7 calls, BRE 7 and 6), although every group was accepted at variant 0. We could not
  determine from the files why the planner chose grouping on reruns. It matters because of the cap: Script Time Trial
  was at 16 ledger calls (1 + 1 + 7 + 7), and a ranked attempt on a second machine stopped at 4 calls
  (`cap_reached`, 1,616 of 1,625 IDs accepted, nothing wrong; nine IDs left unsent). The attempt on that machine
  after it went CORRECT in 7 calls and the ledger now shows 27 for that mode; the files do not say how the
  cap was raised for it.
- **Reset Expectations (Web).** The first solver implemented only the reset-IDOR strategy, so the regular Mastery
  crashed on the other three target classes (13:05:40) and cost about six minutes plus a fix round; the fix
  resolved each answer through the question's `artifact` field and tried all four known strategies per target.

## Tooling and process changes around this day

- `2e99c6a5` (2026-10-25, "Add Day 25 challenge work"): the day's leaves and library updates in one commit: the
  registry-hive reader and entry, role-member-pair notes, the Dalvik VM additions (about 300 changed lines), the
  warmup codec for record 25, the Elixir table entries in the descriptor decoder, and web entries for command
  injection, GraphQL authorization and reset-token IDOR. In the same commit, `tools/release/closeout.py`
  (automatic `progress.md` rows and the librarian pass; the six finished lanes show "progress.md: recorded"
  lines), `orchestrate.py` changes and a librarian settings file.
- `9a19811e` (2026-10-26): escalated cap (20, 70 once a lane escalated) for release automation. The 2026-10-31
  rerun log for this day already reads "cap 20 (70 once escalated)", and Mobile resumed at 70.
- `021814b6`, `6af31628`, `ac51598a` (2026-10-31), `38668151`, `f0e20aa1` (2026-11-01): the ranked rerun and
  gap-closure sweeps. For Day 25 they added the `ranked-N` results used above, the general web dispatcher, and
  the second-machine Script Time Trial runs.
- The librarian proposed `CLAUDE.md` changes for BRE (Day 25 bullet under Patch Diffing), Metadata (Hive Mind),
  Mobile (wrapper contract, VM gaps, validated `payload.dex` hop) and Warmups (record 25); none for Script, and
  none for Reset Expectations. None of them appears in `CLAUDE.md` today (a search for the Day 25 titles finds
  nothing); the lessons live in `library/` entries instead.

## Effort

Submission calls. "Original" is the 2026-10-25 run (`progress.md`, or the ledger's first entries for the two
hand-solved web challenges); "ledger total" is the local ledger today and includes every later ranked rerun.

| Challenge | Single Solve | Time Trial original / total | Mastery original / total |
|---|---|---|---|
| You Have a Point | 1 | 2 / 5 | 2 / 6 |
| Hive Mind | 1 | 2 / 4 | 2 / 4 |
| Integrity Signals | 1 | 2 / 4 | 2 / 3 |
| Ashes to Assets | 7 | 2 / 27 | 2 / 4 |
| Out of Bounds | 6 | 2 / 15 | 2 / 3 |
| Shell Company | 1 | 1 / 7 | 1 / 4 |
| Reset Expectations | 1 | 2 / 4 | 5 / 6 |
| Resolver Revolt | 1 | 1 / 5 | 1 / 7 |

"Original" counts one regular and one ranked call per mode where both ran (Reset Expectations Mastery: four
regular calls to isolate targets, one ranked). The Single Solve counts for Ashes to Assets (7) and Out of Bounds
(6) are grouped submissions with every group accepted at variant 0, not corrections; the only lane with a real fix
round and a model escalation was Mobile, which used one submission call per mode.
