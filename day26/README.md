# Day 26 overview

**At a glance:** 7 challenges across six lanes (two of them Web Exploitation), 21 modes. By provenance label: 20
Independent, 0 Journal-recorded, 0 Local mark only, 1 Team, 0 n/a. The one Team mode is the Trust the Webhook Single
Solve, scored by a teammate about two hours after unlock. Theme: five of the seven challenges added a small new
field or format on top of a cumulative bank, so the solves were quick and the day's cost came from our own
automation (a truncated answer, a stale solver subclass, a lane that produced nothing), plus one web challenge
that took three days to crack.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Food for Thought](warmups_food-for-thought.md) | `+`/`-`/`.` accumulator stream (record 26); literal-tail slug extraction | Independent | Independent | Independent |
| Metadata Forensics | [Before You Go](metadata-forensics_before-you-go.md) | Windows 10 `MAM` compressed Prefetch (XPRESS-Huffman), header name | Independent | Independent | Independent |
| Script Analysis | [Mixed Signals](script-analysis_mixed-signals.md) | Erlang `.escript` added to the descriptor bank; declared fields win | Independent | Independent | Independent |
| Mobile Android | [Split Decision](mobile-android_split-decision.md) | First `.apks` split bundle; v1 `split` codec, then v2 `analysis` via the Dalvik interpreter | Independent | Independent | Independent |
| Binary RE | [Crash Course](binary-reverse-engineering_crash-course.md) | 312-id recap bank; Groovy JAR HCFG, method table, `crash/trace.json` | Independent | Independent | Independent |
| Web Exploitation | [Serialized Offender](web-exploitation_serialized-offender.md) | Unsigned client-controlled state blob (not deserialization) | Independent | Independent | Independent |
| Web Exploitation | [Trust the Webhook](web-exploitation_trust-the-webhook.md) | Webhook trust gate keyed on short body field names the clue endpoint does not advertise | Team | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Independent (20 modes).** Every Single Solve except the webhook one holds an `accepted_plan.json`. Every Time
  Trial and Mastery except the webhook ones holds a `result.json` reading CORRECT (a regular rehearsal, a ranked
  attempt, or both). Nothing here rests on a journal or a progress mark alone.
- **Independent (Trust the Webhook Time Trial and Mastery).** These two modes were solved interactively on
  2026-10-29, after the release run gave up. The leaf carries a `manual_solve.json` marker for each (written from our
  own graded CORRECT, per its `basis` field), and later ranked attempts (2026-10-31 and 2026-11-01) hold their own
  `result.json` files, all CORRECT.
- **Team (Trust the Webhook Single Solve).** `team_solved.json` exists and `progress.md` says "Team solve (not
  ours)". A teammate's attempt was graded CORRECT at 14:54:57 UTC on release day; a fresh Single Solve attempt
  cannot be issued once that happens (retry and practice were refused, per `progress.md`). We derived the mechanism
  ourselves later, but the graded Single Solve is the teammate's.

## Timeline (UTC, 2026-10-26)

Day 26 unlocked at 13:00 UTC (09:00 ET). The watcher detected it 3.9 s after unlock and handed seven challenges
(21 modes) to the orchestrator. Times below come from `_run/watch.log`, `_run/orchestrator.log` and the lane logs.

| Time | Event |
|---|---|
| 12:50 | Watcher starts, preflight against the Day 25 list |
| 13:00:04 | Day 26 detected; recon of all seven challenges starts |
| 13:01:33 | Mobile Single Solve CORRECT (T+1:33) |
| 13:01:49 | Script Single Solve CORRECT (T+1:50) |
| 13:02:16 | Warmups Single Solve CORRECT (T+2:16) |
| 13:02:54 | Metadata Single Solve CORRECT (T+2:54) |
| 13:05:30 | Binary RE Single Solve CORRECT, 3 calls (T+5:30) |
| 13:07:45 | Serialized Offender Single Solve CORRECT (T+7:45) |
| 13:02:18 | Script lane finished (Time Trial and Mastery, regular and ranked) |
| 13:03:10 | Warmups lane stopped: ranked Mastery `incorrect` twice (see below) |
| 13:05:51 / 13:06:48 / 13:07:15 | Mobile, Binary RE and Metadata lanes finished |
| 13:12:20 | Trust the Webhook: no candidate finished in round 0, lane dropped |
| 13:15:59 | Serialized Offender lane finished (after a Mastery fix round) |
| 13:16:02 | Orchestrator: "all lanes finished" (T+16:02) |
| 13:16-13:36 | Librarian pass for the five lanes that were fully CORRECT (167-495 s each); warmups and the webhook challenge were skipped as not yet CORRECT |
| 14:54:57 | A teammate's attempt scores Trust the Webhook Single Solve (from `progress.md` and the journal) |

Each lane's finish covers Single Solve, a regular Time Trial and Mastery rehearsal, then a ranked Time Trial and
Mastery attempt. Ranked windows from the lane logs: Warmups 30 s / 145 s, Metadata 50 s / 325 s, Script 50 s / 320 s,
Binary RE 80 s / 660 s, Mobile 50 s / 320 s, Serialized Offender about 128 s for both, Trust the Webhook about 125-133 s.
These ranked runs were bragging-rights attempts, not the scoring rule used later in the event.

The release automation ran again later against the same Day 26 leaves, always resuming from the existing results
(`orchestrator.log`): 2026-10-31 23:09 (all seven challenges, ranked reruns), 2026-11-01 04:29-04:30 (Binary RE Time
Trial, Trust the Webhook), 05:07 (Trust the Webhook Time Trial) and 06:44 (Metadata, Script, Warmups, Serialized
Offender). A few reruns were run from a second machine. These reruns are why most modes hold several `ranked-N`
folders; all that have a result file read CORRECT except the two incorrect Warmups Mastery attempts below.

## Submission effort

Calls per mode from `progress.md` (the ledger count: regular rehearsal plus the first ranked attempt, before
later reruns):

| Challenge | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Food for Thought | 1 | 2 | 19, then 1 more on a fresh ranked attempt (ledger 20 of an authorized 30) |
| Before You Go | 1 | 3 | 2 |
| Mixed Signals | 4 | 2 | 2 |
| Split Decision | 1 | 2 | 2 |
| Crash Course | 3 | 2 | 2 |
| Serialized Offender | 1 | 2 | 2 |
| Trust the Webhook | team | not recorded | not recorded |

Locally measured end-to-end time of the first CORRECT ranked pass (download, solve, submit), Time Trial / Mastery:
Warmups 4.27 s / 6.61 s (the Mastery figure is the fixed attempt), Metadata 3.93 s / 4.89 s, Script 5.18 s / 8.98 s,
Mobile 2.65 s / 6.3 s, Binary RE 11.15 s / 28.05 s, Serialized Offender 14.22 s / 21.82 s. Trust the Webhook's first
ranked attempts took 132.6 s (Time Trial, interactive) and 183.6 s (Mastery), then 9.85 s for a later scripted Time
Trial.

## Escalations to the stronger model

- **Warmups Mastery.** After two incorrect ranked passes with the ledger at 19 of 20, an Opus diagnosis (offline,
  no submissions, `escalation-diagnosis.md` in the leaf) found the cause was slug extraction, not a missing codec.
  The player then extended the cap to 30 and a fresh ranked attempt went CORRECT on one call.
- **Trust the Webhook.** Both round-0 solvers stalled and recommended an Opus escalation. The journal records several
  Opus passes on 2026-10-26 and 10-27 (numbered live reviews, differential analysis of the handler, and a single
  player-authorized ranked Time Trial retry used to test the Host-header theory). None found the bypass. It fell to an
  interactive session on 2026-10-29.
- **Everything else.** The lane logs record no automated escalation: the three other fix rounds (Mobile, Metadata,
  Serialized Offender) were ordinary second rounds, and we could not determine from the logs which model ran them.
- **Supervisor.** None for this day; no `supervisor-report.md` exists (the supervisor first ran on Day 29).

## What the day had in common

- **Small deltas on cumulative banks.** Metadata's 31 per-set questions, Binary RE's 312 ids (26 blocks of 12) and
  Script's 1,615 ids were all recap banks where only the day's block was new: one Prefetch field, three JAR/trace
  reads, 44 Erlang ids. Warmups Mastery likewise grew a record 26.
- **Modes of one challenge changed schema.** Mobile Single Solve was a v1 `split` asset while Time Trial and Mastery
  were v2 `analysis`; Metadata's compressed `.pf` appeared only in Single Solve; Script's Time Trial and Mastery
  switched to hash-style ids and layered markers. Solvers had to dispatch on the data, not the day.
- **Prompts oversold or misdirected.** Crash Course talks about unwinding a stack and a faulting instruction (the data
  is a JSON list of three names); Mixed Signals promises collision decoys we never saw; Serialized Offender is
  dressed as Java deserialization (it is unsigned base64 JSON); Trust the Webhook's own clue endpoint named the wrong
  field keys.
- **Web challenge IDs lag the release day.** Both web challenges carry earlier curriculum days (`day-19`, `day-20`)
  but live in `day26`. Both Mastery bundles mixed vulnerability classes across targets (four targets each), so only
  one target matched the challenge's own class.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Warmups | The slug is the literal tail after the last colon. One ranked Mastery set ended `word-word-word-NN-NNNNN`; the validator rejected the `word-word-word-NN` prefix. |
| Warmups | Record 16 (Morse for hex) looks like a `+`/`-`/`.` stream to a loose check; the new codec must require a `+`. |
| Metadata | `.pf` may be `MAM`-compressed or plain `SCCA` within one challenge; the stored upper-case name (`CMD.EXE`) was accepted. |
| Script | For Erlang, the descriptor's declared `role`/`attack` (bare `T1059` for a launcher) beat the step-graph reading, which could even produce invalid roles. |
| Mobile | The `.apks` extension broke suffix-based delivery selection; the Mastery bundle mixed `.apk` and `.apks`. |
| Binary RE | "Primary classfile" means the one named by `Main-Class` (method count 10, not 5 or 15). The `false` branch of the frames predicate was never exercised. |
| Web | Role spelling (`administrator`, not `admin`) and field-name vocabulary (`event`/`owner`, not the clue's `event_type`/`owner_scope`). |
| All lanes | A rejected multi-answer batch reports no per-field detail; the Warmups bad id cost 15 calls of halving. |

## Mistakes in our own tooling or process

Collected from the challenge pages and the lane logs.

- **Split Decision (Mobile): late ranked failure from a stale subclass.** Both leaf solvers subclassed the library's
  interpreter and injected the split assets as a class attribute. When the library later gained native split-bundle
  support, its state reset overwrote the attribute with an empty list. The 2026-10-31 ranked reruns failed with
  `split asset ... not found` on the Time Trial and Mastery set 10 (`solver_failed` in `status.json`); the original
  2026-10-26 runs had passed. The fix was to delete the subclass and call the library directly, checked offline
  against saved fixtures and the library's regression runs (177 of 177, per `progress.md`). The same check on
  2026-11-01 found the original ranked results had been credited to a solo account instead of the team, so both modes
  were re-run (fresh ranked attempts, one call each). The first Time Trial rehearsal had also failed (`unexpected
  codec 'analysis'`) because the solver only knew the Single Solve codec.
- **Food for Thought (Warmups): two solver bugs, one lost cap, one attribution re-run.**
  - The first solver tried the new `+`/`-`/`.` decoder on every record before the codec table. Record 16 passed the
    loose check and produced garbage, so its slug assertion fired on every Mastery set in the regular rehearsal. The
    second solver made the same record-16 mistake but survived it.
  - Both solvers extracted the slug with a search for the first `word-word-word-NN`, which truncated set 6 of the
    ranked bundle. Ranked Mastery went `incorrect` twice (the second pass reattached to the same attempt), using 15 and
    3 calls and leaving the ledger at 19 of 20. The orchestrator stopped, and the 2026-10-31 rerun logged "ledger 20 >= cap
    20; not issuing". The fix needed an Opus diagnosis and a player-authorized cap extension.
  - The fixed run's CORRECT was credited to a solo account; a fresh re-run on 2026-11-01 (one call) landed under the team.
- **Serialized Offender (Web): round-0 solver crashed on the Mastery bundle.** It only implemented the single-target
  state-blob recipe, so the 4-target Mastery rehearsal ended `solver failed: no usable candidate` (one solver raised,
  the other failed lint with three ids missing). A fix round that added the other three classes took about seven
  minutes (13:08 to 13:15) and then went CORRECT in one call.
- **Before You Go (Metadata): a one-field solver written into every id.** The first Time Trial candidate answered all
  31 ids with the Prefetch name and failed our own enum lint before anything was submitted. It cost one fix round
  (13:02 to 13:07). Nothing was graded wrongly.
- **Trust the Webhook (Web): a silent lane drop and a wrong library entry.**
  - Round 0 produced no candidate and the orchestrator dropped the lane instead of escalating; the closeout then
    skipped it. `tools/release/orchestrate.py` was changed on 2026-10-26 so a Single Solve round with no finished
    candidate escalates (`stall()`).
  - The release run's pinned team attempt was an unsolved one, so the challenge looked unsolved from here for days even
    though a different team attempt had scored it.
  - A Host-header CVE theory was recorded in the library as this challenge's vector and was later disproved by the
    short-field-name result; `progress.md` and the library were corrected on 2026-10-29. See the Trust the Webhook
    page for the reasoning.
  - Later scripted reruns hit our dispatcher: the 2026-10-31 run found the Time Trial target already cleaned (409) and
    the Mastery solver had no candidate; the dispatcher also sent placeholder values under the right short keys until
    it was changed to use the clue's own sample values first (`progress.md`, 2026-10-31 rows).
- **Crash Course (Binary RE) and Mixed Signals (Script): extra calls on reruns.** Binary RE's Time Trial reruns took
  3 calls because the alternate-reading groups were submitted as single-id calls before the base map; the base map
  was unchanged. Script's Time Trial reruns took 4 grouped calls instead of 1; the lane log does not say why.

## Tooling and process changes around this day

From `git log` and the librarian notes:

- `7d55a47f` (2026-10-26): the Day 26 challenge work and run folder.
- `e1c28cfe` (2026-10-26, evening): the Warmups record-26 fix in `codec_chain.py` and the ranked solver, with new parity
  cases (`tools/ranked/cases/warmups-d26.json`); the `stall()` escalation in `orchestrate.py`; a read-only `openapi`
  subcommand in `gm.py`; and the Host-header library entry that was later corrected.
- `9a19811e` (2026-10-26): the escalated submission cap (20 per ID+mode, 70 once a lane escalates), first used for Day
  27. Day 26's own first run used a flat cap of 20, which is why Warmups Mastery stopped at 19 of 20.
- `6a838a2d` (2026-10-29): Trust the Webhook attribution fixed and Time Trial and Mastery recorded; `27a80dee` the same day
  added the `manual_solve.json` marker that the webhook leaf uses and reworked handling of team-scored Single Solves.
- `021814b6`, `6af31628` (2026-10-31), `38668151`, `f0e20aa1` (2026-11-01): ranked reruns and gap-closure sweeps that
  added the later `ranked-N` folders, the Mobile subclass fix and the webhook dispatcher fix.
- **Librarian proposals.** Metadata proposed a `CLAUDE.md` note (read the Prefetch name at offset 16 as stored;
  decompress `MAM` first; never ship a one-field solver as a full-bundle candidate), and Mobile proposed one (`.apks`
  deliveries, never select by `.apk` suffix). Binary RE, Script and Serialized Offender proposed none. A grep of the
  current `CLAUDE.md` finds no Day 26, Prefetch or `.apks` lines, so neither proposal was adopted there; the
  compressed-Prefetch note does exist as a library entry. No librarian note exists for Warmups or Trust the Webhook.
