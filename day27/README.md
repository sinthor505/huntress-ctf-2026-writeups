# Day 27 overview

**At a glance:** 8 challenges across six lanes (five non-web lanes plus three web challenges), 24 modes. By
provenance label: 19 Independent, 5 Team, 0 Journal-recorded, 0 Local mark only, 0 n/a. The five Team modes
are the Single Solves of every non-web lane: a teammate scored them before our first submission, so ours came
back `already_solved` and nothing was graded for us. This was also the day our release automation mishandled
that response, spent hours and a lot of submissions on it, and never reached Time Trial or Mastery on the
release day (see "What went wrong"). Time Trial and Mastery for those five lanes were solved on fresh `retry`
attempts about 17.6 hours after release. The three web lanes were solved on release day with no such problem.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Monkey Business](warmups_monkey-business.md) | Ook! (a Brainfuck dialect) through a Brainfuck VM; new record 27 in Mastery | Team | Independent | Independent |
| Metadata Forensics | [Packet Loss Prevention](metadata-forensics_packet-loss-prevention.md) | classic pcap record walk by `incl_len`, snaplen-cut records counted | Team | Independent | Independent |
| Script Analysis | [FROM Whence It Came](script-analysis_from-whence-it-came.md) | Dockerfile descriptor markers; declared `role`/`attack` beat the step graph | Team | Independent | Independent |
| Mobile Android | [Dependency Surface](mobile-android_dependency-surface.md) | plain v1 `base64` asset decode; the "dependency" theme was dead metadata | Team | Independent | Independent |
| Binary RE | [Compiler Says What](binary-reverse-engineering_compiler-says-what.md) | recap bundle; gcc/tcc role pair; literal HCFG trailer (XOR + CRC32) | Team | Independent | Independent |
| Web Exploitation | [Double Spent](web-exploitation_double-spent.md) | TOCTOU race / last-byte-synchronized double spend | Independent | Independent | Independent |
| Web Exploitation | [Socket to Me](web-exploitation_socket-to-me.md) | WebSocket-upgrade authorization bypass | Independent | Independent | Independent |
| Web Exploitation | [Forwarded Misgivings](web-exploitation_forwarded-misgivings.md) | trusted-proxy `X-Forwarded-For` IP spoofing | Independent | Independent | Independent |

How the labels were assigned, from local files only:

- **Team (five non-web Single Solves).** Each leaf's `single-solve/` holds a `team_solved.json`, and `progress.md`
  records `Team solve (not ours)` for each. The lane logs show every Single Solve submission as `already_solved`
  (BRE 36, Script 52, Metadata 8, Mobile 4, Warmups 4). Nothing was graded, so the offline answer maps for these
  five modes were never validated by their own mode. They were validated indirectly: the same readers went
  CORRECT in Time Trial and Mastery. A Single Solve cannot be reissued once a teammate has solved it (retry 405,
  practice 409, per `progress.md` and the shared gotchas note), so these cannot be redone.
- **Independent (all other modes).** Every Time Trial and Mastery leaf holds `result.json` CORRECT in both its
  `regular/` and `ranked-1/` folders. The three web Single Solves hold `single-solve/accepted_plan.json`, and the
  lane logs show each as `result=CORRECT` on its first submission call.

## Timeline (UTC, 2026-10-27 unless noted; from `watch.log`, the lane logs and `orchestrator.log`)

| Time | Event |
|---|---|
| 12:50 | Release watcher starts; polling opens at 12:55 |
| 13:00:03 | Day 27 detected after 211 polls (T+3.6 s); recon complete in every lane within 3 to 20 s |
| 13:02-13:10 | First solver candidates ready: Mobile 13:02, Warmups 13:02, Metadata 13:04, Script 13:04, BRE 13:10 |
| 13:03-13:12 | First Single Solve submissions in the five non-web lanes (Mobile 13:03, Warmups and Script 13:04, Metadata 13:06, BRE 13:12), every one `already_solved` |
| 13:11-13:14 | Web Single Solves CORRECT: Forwarded Misgivings 13:11, Double Spent 13:13, Socket to Me 13:14 |
| 13:03-13:21 | Fix rounds run in all five non-web lanes; each escalates to the stronger model in round 2 (13:14 to 13:21) and raises the cap from 20 to 70 (Script had already used 20 of 20 at 13:13) |
| 13:11-13:27 | Web Time Trial and Mastery (regular then ranked) CORRECT for Forwarded Misgivings (13:24) and Socket to Me (13:27); Double Spent Time Trial CORRECT, Mastery failing lint |
| 13:25 | Usage gate (5-hour window 86%, threshold 80%) stops new solver sessions: BRE lane from 13:25, Double Spent lane from 13:28 |
| 18:02-18:07 | Gate clears; BRE's last Single Solve round is again `already_solved`; "all lanes finished" at 18:07:56 |
| 18:08-18:31 | First Librarian pass, over seven lanes (Metadata skipped) |
| 10-28 06:34 | Run resumed; the first Time Trial/Mastery attempts for the five lanes fail (start returned the team readout, no artifact) |
| 10-28 06:38-06:40 | Patched orchestrator issues a fresh `retry`; regular and ranked Time Trial and Mastery CORRECT in all five lanes on call 1 (Script TT 4 calls, Metadata 2) |
| 10-28 06:50 | Double Spent Mastery CORRECT (regular 06:49, ranked 06:50) after the multi-class dispatcher |
| 10-29 02:32-02:51 | Second Librarian pass, over the lanes that now had our own results |
| 11-01 06:44-06:45 | Late gap-closure run over Mobile, Script and Warmups: more ranked CORRECTs |

Release to our first own CORRECT per mode (lane-log `result=CORRECT` times against detection at 13:00:03):

| Lane | Single Solve | Time Trial (regular) | Mastery (regular) |
|---|---|---|---|
| Forwarded Misgivings | 11 min | 12 min | 24 min |
| Double Spent | 14 min | 21 min | 17 h 50 min |
| Socket to Me | 14 min | 15 min | 26 min |
| Metadata, Mobile, Warmups, Script, BRE | Team | 17 h 34 min to 17 h 39 min | 17 h 38 min to 17 h 40 min |

The ranked (Hall-of-Fame-eligible) attempts followed a minute or so after each regular one. Ranked runs were
bragging rights, not the scoring, so they are not tabulated.

## Submission effort

Calls per mode, from `result.json` and the lane logs. Single Solve for the non-web lanes is the count of
`already_solved` responses, not graded calls.

| Lane | Single Solve | TT regular | TT ranked | Mastery regular | Mastery ranked |
|---|---|---|---|---|---|
| Warmups | 4 (Team) | 1 | 1 | 1 | 1 (+2 later reruns, 1 each) |
| Metadata | 8 (Team) | 2 | 1 | 2 | 1 |
| Script | 52 (Team) | 4 | 1 (+2 reruns, 4 calls each) | 1 | 1 (+1 rerun, 4 calls) |
| Mobile | 4 (Team) | 1 | 1 (+1 rerun) | 1 | 1 |
| BRE | 36 (Team) | 1 | 1 | 1 | 1 |
| Double Spent / Socket to Me / Forwarded Misgivings | 1 each | 1 each | 1 each | 1 each | 1 each |

The Script reruns include two on a second machine (the ranked folders named for it); all were CORRECT. The web
Mastery attempts needed fix rounds before the one graded call, but those rounds produced no graded submissions
(solver failures before reaching the Game Master).

## What the day had in common

- **Prompts promised more than the questions asked.** BRE talked about attributing a toolchain (no question
  touched a compiler), Metadata about rejecting malformed captures (no malformed capture was ever delivered),
  Mobile about a reachable dependency flaw (no dependency existed in any build we saw), Script about
  resemblance between specimens (the answers were the descriptors' declared fields). Warmups was the honest
  one: "Ook." named the codec.
- **Three non-web lanes added one small block on top of a recap bundle.** Metadata
  added `pcap.packet_count` to a 32-field recap, Script added four Dockerfile specimens to Day 26's bank, BRE
  added a 12-id block to 312 older ids. Diffing against the previous day's accepted plan made the old ids safe.
- **Web: three new classes, and the usual mixed Mastery bundles.** Race condition, WebSocket upgrade bypass
  and forwarded-header trust were all new to the lane. Each Mastery bundle mixed in unrelated earlier classes,
  and Double Spent's mix changed between grading attempts (recon every bundle fresh).

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Script | Declared descriptor `role`/`attack` beat the step graph for the Dockerfile family (variant 0 everywhere). Time Trial and Mastery reused Single Solve's named ids (no hash ids or `LYR` layers as on Days 23-26): dispatch on id shape, not the day. |
| Metadata | Snaplen-cut records still count; the plain record count was accepted in all graded runs. |
| Warmups | Record 27 (the Ook layer) was missing from the codec record table, the same trap as on Day 29; the solver registered it at import time. |
| Mobile | Mastery bundles are rebuilds of earlier days (codecs reverse, base64, hex, xor, caesar, native, reflection, dynamic, split), so classify per set, not per day. |
| BRE | The 312 older ids matched Day 26's accepted plan; the 12 new ids read cleanly (`elf`, `62` as a string). |
| Web | The proof is the verbatim `evidence` token; Double Spent's proof endpoint must be polled, not read once. |
| All | Every answer value is a string; `already_solved` is not a verdict. |

## What went wrong (our own automation)

These cost submissions and hours, and are not challenge lessons. The shared gotchas note and `CLAUDE.md` now
carry the short versions.

- **`already_solved` was read as "0 accepted".** All five non-web Single Solves were team-scored before our
  first submission. The log cannot say when the teammate solved them (`progress.md` says "within seconds" from
  a Game Master status read, which I did not re-verify); it only shows our earliest submissions, from T+3 min,
  already returned `already_solved`. The orchestrator treated the replies as a systematic rejection,
  bisected, and opened fix rounds with unchanged solvers: BRE 36 calls over four rounds, Script 52 over four
  (hitting 20 of 20 before escalating), Warmups, Mobile and Metadata 4 to 8 each. All five escalated to the
  stronger model in round 2 and raised the cap to 70. The candidates were identical each round (the logs show
  the same answer and group counts), and no grading signal existed to change them.
- **The run ended without attempting Time Trial or Mastery.** At 18:07 the orchestrator reported "all lanes
  finished" with the five lanes' Time Trial and Mastery never started. Its lane logs say `progress.md` rows
  for all three modes were recorded, and the first Librarian pass ran over four of those lanes. The BRE
  journal's Librarian section says the d27 readings were then "observed, not validated" because everything returned
  `already_solved`. I could not determine from the files what those first rows said. The Time Trial and Mastery
  attempts only happened after the 10-28 06:34 restart.
- **A team readout is not a delivery.** On restart, Time Trial and Mastery `start` returned the team's CORRECT
  status object, which has no `artifacts[]`. Solvers failed on it (Time Trial in BRE, Mobile, Script and Warmups; Metadata
  Mastery); those four lanes opened two more fix rounds and Metadata one, each with another cap raise, before the
  orchestrator was patched to issue a fresh `retry`; after that every lane went CORRECT on call 1 (Script and Metadata needed 2 to 4
  calls for group-by-group acceptance).
- **Usage gate idle time.** From 13:25 to about 18:02 the BRE lane (and from 13:28 to 17:58 the Double Spent
  lane) slept in 30-minute steps at 86% to 95% of the 5-hour window (threshold 80%). The logs do not say what
  consumed the window; the five lanes' repeated escalated rounds ran just before it.
- **Web Mastery solvers assumed one class.** The first Socket to Me and Forwarded Misgivings Mastery solvers
  only handled their own class and returned one of five proofs; the release harness discards a candidate with
  any missing answer id, so the attempt was `solver_failed` rather than partial. Both were fixed on release day
  by per-target dispatch. Double Spent needed four rounds across the usage gate and the restart.
- **Bookkeeping.** `progress.md` still reads "no ranked CORRECT recorded" for Double Spent Time Trial although
  `ranked-1/result.json` is CORRECT (11.98 s). Its auto-recorded rows also carry "Game Master solve time"
  values that are negative (down to about -17 hours), so they are not solve durations.

## Supervisor, escalations and the Librarian

- **Supervisor:** first armed on Day 29. For Day 27 only a dry-run log exists (2026-10-29 02:39, all 8 lanes
  "ok"); there is no supervisor report.
- **Escalations to the stronger model:** five non-web lanes in round 2 (13:14 to 13:21), all triggered by
  `already_solved`; Double Spent Mastery at 13:24 (a genuine multi-class problem); and the cap raises logged again
  at the 10-28 restart. Only the Double Spent escalation addressed a real failure.
- **Librarian proposals:** BRE, Mobile and Warmups: a start returning the team's CORRECT payload has no
  artifacts, so a fresh `retry` is needed, and `already_solved` should end a submission loop; Mobile: check for a
  `Release` class before reaching for the Dalvik VM; Script: dispatch on id shape (named ids on Day 27); Metadata:
  new `pcap.packet_count` entry; Warmups: add `ook` to the codec chain and record map; Double Spent: race-condition
  entry (poll after the burst; Mastery mixes classes). Socket to Me and Forwarded Misgivings proposed no
  `CLAUDE.md` change.

## Tooling and process changes around this day

From the commit history:

- `9a19811e` (2026-10-26): escalated submission cap in release automation (20, or 70 once a lane escalates).
  Day 27 was the first run to use it; Script reached 20 of 20 at 13:13 and escalated.
- `2360033e` (2026-10-28): the Day 27 work and the fixes it prompted: "solved" means local evidence only
  (journal, `progress.md`, `result.json` or `accepted_plan.json`), `closeout.is_correct` reads local evidence,
  `submit_plan` stops on `already_solved`, the orchestrator retries when `start` returns the team's readout and
  records `team_solved.json`, and the Librarian gate handles a team-scored Single Solve.
- `27a80dee` (2026-10-29): team-scored handling consolidated, `manual_solve.json` marker for hand-solved leaves,
  and the Day 27 retelling condensed into the shared gotchas note.
- `1a0fa8a1` (2026-10-29): release-day supervisor, after Day 27.
- `38668151` and `f0e20aa1` (2026-11-01): ranked gap-closure sweeps, which added the later Day 27 reruns, including
  the second-machine ones.
