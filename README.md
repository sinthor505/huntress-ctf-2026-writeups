# Huntress CTF 2026: writeups and progression

Writeups for a team entry in the 2026 Huntress CTF, written after the event closed. The event released one
set of challenges per day (09:00 ET) across Warmups, Metadata Forensics, Script Analysis, Mobile Android and
Binary Reverse Engineering, with a Web Exploitation lane added from Day 19 and two Welcome challenges on
Day 1. Each challenge has up to three modes: Single Solve, Time Trial and Mastery. The event ran for 31
days, closing 2026-11-01; the team's final standing was 41st of 1,594.

**This set is complete: Days 1-31.** Days 1-10 were played by hand, before this repository's own history
begins. Days 11-20 kept that by-hand shape while the technique library absorbed most of the day-to-day
cost. Release automation (a watcher, an orchestrator, headless solver sessions, a submission ledger) went
live on Day 21, the Web Exploitation lane (debuted Day 19) joined that automation on Day 22, and a
release-day supervisor was added for Day 29. Day 30 added a one-off reversing challenge we could derive
offline but never grade; Day 31 was the event's 24-hour finale.

## How to read this repo

- `dayNN/README.md` is the day overview: the day's challenges, what they had in common, validator traps,
  our own tooling mistakes, process changes, and a timeline where files support one.
- `dayNN/<lane>_<challenge>.md` is one challenge: technique, solution, what changed in Time Trial and
  Mastery, dead ends, takeaways, and the evidence files it rests on.
- Flags and answers appear in places. They were issued per team and are worthless now.
- **Provenance labels.** Every mode carries one, judged only from this workspace's own graded result files,
  never from the Game Master's team-wide status. **Independent**: our own CORRECT file is in the leaf.
  **Journal-recorded**: the journal records our own CORRECT but no result file survives. **Local mark only**
  (or "graded result not preserved"): only a progress-ledger mark exists. **Team solve (not ours)**: a
  teammate's graded solve scored the mode and ours returned `already_solved`. A mode that is not Independent
  is described as what it is, not claimed.
- "Independent" for an old Time Trial or Mastery often rests on a later scripted rerun (see "Evidence
  quality" below). Each page says which attempt backs which label.

## Daily Summary

| Day | Challenges | Modes | Independent | Journal-recorded | Local mark only | Team | Absent |
|---|---|---|---|---|---|---|---|
| [Day 1](day01/README.md) | 7 | 17 | 9 | 2 | 6 | 0 | 4 |
| [Day 2](day02/README.md) | 5 | 15 | 9 | 5 | 1 | 0 | 0 |
| [Day 3](day03/README.md) | 5 | 15 | 11 | 2 | 2 | 0 | 0 |
| [Day 4](day04/README.md) | 5 | 15 | 11 | 4 | 0 | 0 | 0 |
| [Day 5](day05/README.md) | 5 | 15 | 11 | 1 | 3 | 0 | 0 |
| [Day 6](day06/README.md) | 5 | 15 | 12 | 3 | 0 | 0 | 0 |
| [Day 7](day07/README.md) | 5 | 15 | 11 | 4 | 0 | 0 | 0 |
| [Day 8](day08/README.md) | 5 | 15 | 15 | 0 | 0 | 0 | 0 |
| [Day 9](day09/README.md) | 5 | 15 | 14 | 1 | 0 | 0 | 0 |
| [Day 10](day10/README.md) | 5 | 15 | 10 | 5 | 0 | 0 | 0 |
| [Day 11](day11/README.md) | 5 | 15 | 8 | 7 | 0 | 0 | 0 |
| [Day 12](day12/README.md) | 5 | 15 | 3 | 12 | 0 | 0 | 0 |
| [Day 13](day13/README.md) | 5 | 15 | 13 | 2 | 0 | 0 | 0 |
| [Day 14](day14/README.md) | 5 | 15 | 12 | 3 | 0 | 0 | 0 |
| [Day 15](day15/README.md) | 5 | 15 | 13 | 2 | 0 | 0 | 0 |
| [Day 16](day16/README.md) | 5 | 15 | 15 | 0 | 0 | 0 | 0 |
| [Day 17](day17/README.md) | 5 | 15 | 10 | 5 | 0 | 0 | 0 |
| [Day 18](day18/README.md) | 5 | 15 | 7 | 8 | 0 | 0 | 0 |
| [Day 19](day19/README.md) | 8 | 24 | 16 | 8 | 0 | 0 | 0 |
| [Day 20](day20/README.md) | 7 | 21 | 14 | 7 | 0 | 0 | 0 |
| [Day 21](day21/README.md) | 8 | 24 | 22 | 0 | 0 | 2 | 0 |
| [Day 22](day22/README.md) | 7 | 21 | 20 | 1 | 0 | 0 | 0 |
| [Day 23](day23/README.md) | 8 | 24 | 16 | 7 | 1 | 0 | 0 |
| [Day 24](day24/README.md) | 7 | 21 | 21 | 0 | 0 | 0 | 0 |
| [Day 25](day25/README.md) | 8 | 24 | 22 | 2 | 0 | 0 | 0 |
| [Day 26](day26/README.md) | 7 | 21 | 20 | 0 | 0 | 1 | 0 |
| [Day 27](day27/README.md) | 8 | 24 | 19 | 0 | 0 | 5 | 0 |
| [Day 28](day28/README.md) | 7 | 21 | 14 | 0 | 0 | 7 | 0 |
| [Day 29](day29/README.md) | 7 | 21 | 19 | 0 | 0 | 2 | 0 |
| [Day 30](day30/README.md) | 8 | 22 | 17 | 0 | 0 | 5 | 2 |
| [Day 31](day31/README.md) | 7 | 21 | 15 | 0 | 0 | 6 | 0 |
| **Total, Days 1-31** | **189** | **561** | **429** | **91** | **13** | **28** | **6** |

"Absent" is a mode that structurally doesn't exist for that challenge, not a failed attempt: Day 1's
four are the two Welcome challenges (Single Solve only, no Time Trial or Mastery); Day 30's two are
SHA1-Hulud, which has no Time Trial or Mastery at all. Modes already exclude Absent ones, so the five
provenance columns (Independent through Team) sum to Modes on every row. Counts for Days 1-28 are
taken from each day overview's own "at a glance" line; Days 29-31 (which don't carry that line) are
tallied directly from each day's challenge table.

## Progression, Days 1-10

### The shape of a day

Every day from 2 to 10 was five challenges, one per lane, with all three modes: 15 modes a day. Each lane's
Single Solve was small (often one field), and Time Trial and Mastery re-asked the same thing at larger
scale, with fresh bytes on every `retry`. Almost every title and prompt was flavor text about something the
questions never tested; the answer was a literal field the question list named. Reading `answer_ids`
before the story was the single most useful habit, and it was relearned on nearly every day.

### Phase 1: primers (Days 1-5)

Each lane's early challenges were the simplest members of families that recur for the whole event: Base64,
ROT13 and hex warmups; hashes, magic bytes, BOMs and image headers in ZIPs; inert PowerShell, JScript and
Batch descriptors; ELF/PE headers and an XOR-plus-CRC32 `HCFG` config record; and an APK whose answer sits in
`assets/huntress_puzzle.json` under a decoy `client_secret`. The analysis was easy. The costs were elsewhere:

- **Short server clocks.** Time Trial windows were 20-50 s and Mastery 35-110 s, and on these days a plain
  `retry` was already the ranked attempt, so there was no free rehearsal. Every Time Trial and Mastery that
  went CORRECT did so from one script: retry, download, verify hash, solve, submit.
- **Mastery is not one file.** Bundles arrived as outer ZIPs of inner ZIPs, several separate deliveries, or
  five APKs. The Game Master client's `artifact` command returned only the first, and answer IDs had to be
  mapped to deliveries from the question text, not file order.
- **Day 4 Script Analysis took seven Time Trial windows**, each ending on a different one-bug difference in a
  freshly randomized archive. That produced the rule "test the decoder on every saved variant before the
  timer".
- **Rotation exhaustion** first bit on Day 5 (Rust Belt Mastery): a pool with no unique attempts left, then
  recovered through untimed `practice`.

### Phase 2: recap days and validator archaeology (Days 6-10)

From Day 6 the answer sets became cumulative. The BRE challenge asked 40-90 fields covering Days 1 through N
(Day 6: 48; Day 7: 56; Day 8: 80; Day 9: 90), Warmups Mastery chained one codec per record number (Day N has N
records), and Script Analysis banks grew to 400-750 fields. A new day usually added one new thing; the rest
was carried forward. That favoured a cumulative parser library, and it is also why wrong carry-over was a
common mistake (BRE fixtures for earlier days are only sometimes byte-identical, so compare hashes first).

The Script Analysis lane is where the validator was learned, and it was expensive. A multi-answer `incorrect`
carries no per-field feedback, so cause isolation costs submissions. The costliest Single Solves:

| Challenge | Single Solve calls | What was wrong |
|---|---|---|
| [Shell We Begin](day06/script-analysis_shell-we-begin.md) (Day 6) | 68 | Batch/VBScript controls and expansions; Time Trial never reached CORRECT in the original play |
| [Lost in Translation](day08/script-analysis_lost-in-translation.md) (Day 8) | 60 | coarse full-map submissions, then single-field retries |
| [All the World's a Stager](day09/script-analysis_all-the-worlds-a-stager.md) (Day 9) | 43 | Batch-control mapper searched only lines before the reconstruction assignment |
| [Node by Node](day07/script-analysis_node-by-node.md) (Day 7) | 40 | one uploader control, found only with per-field isolation |
| [Bind Over Matter](day10/script-analysis_bind-over-matter.md) (Day 10) | 30+ | descriptor `attack` field vs. printed graph rubric |
| [Pool Party](day06/binary-reverse-engineering_pool-party.md) (Day 6, BRE) | 24 | `d01_analysis_symbols`: every non-empty `.symtab` name, including `FILE` entries and duplicates |

The rules that came out of this stretch, and that later days reused, were:

- **The descriptor's own field wins** over the printed graph rule and over the plan text (`attack`,
  `adapter`, `role`).
- **Batch `*-control` follows the specimen's structure, not the prompt.** A one-item `FOR /L ... CALL`
  wrapper was `call_subroutine` where the prompt said `for_once`; an `if defined` guard was `if_guard`.
- **Python chains ending in `dispatch_process` took the language sub-technique** (`T1059.006`) where the
  graph says `T1105`, but not every day or language.
- **Every answer value is a JSON string.** Native ints, booleans and arrays returned `invalid_answers`,
  graded nothing and still counted (Days 3, 5, 7, 8, 10).
- **Spelling is per field and per day.** `little` vs `little endian` vs `little-endian`; `PE32_CLR` vs
  `PE32_CLI`; padded vs unpadded hex for a PE entry RVA; `x86-64` for ELF vs `AMD64` for PE.
- **Submit by group, never as one big map**: family, then suffix, then single ID.

By Day 9 and 10 the expensive lanes were cheap again where the library already held the answer: Nimble and
Zig Zag, the BRE challenges, needed one or two Single Solve calls.

### What stayed constant across all ten days

- **Prompt as decoy.** A WebView bridge that is never called; Kotlin coroutines that no question touches;
  a mismatched-ZIP "census" in which the counts always matched; a Swift-only prompt over eight days of
  recap fields.
- **Decoys sit next to the real value.** The mobile `client_secret` beside `proof=`; a CRC-invalid literal
  `HCFG`; a "wrong" magic string in a header.
- **Fixtures change per attempt.** A `retry` can issue new bytes, keys and slugs, so no answer carries
  between attempts or modes. Anything keyed on file name or order broke; anything keyed on the start
  response and on magic bytes survived.
- **Mastery bundles mix container types** (an ELF specimen replaced by a PE32+ in some sets), so classify by
  magic bytes.

### Evidence quality: how we played, and what the labels mean

We started playing around 2026-10-18, after Days 1-10 had been released, and the early work kept little.
The honest picture, from the overviews:

- **Single Solves from Days 1-5 are weakly evidenced.** Days 1-5 have 12 "Local mark only" modes, mostly
  Single Solves where the leaf holds the raw delivery and no journal or response. Day 1's five lane Single
  Solves, for example, survive only as a progress mark. The pages say so rather than claim them.
- **A 2026-10-20 audit re-solved the Time Trial and Mastery marks** from fresh, hash-verified bytes, because
  early rows had been marked Solved on the player's word without a graded result. Those are the origin of
  the progress ledger's "local evidence only" rule, which later made the Game Master's team-wide status
  inadmissible as evidence.
- **Most Time Trial and Mastery "Independent" labels rest on the ranked sweep** (2026-10-25/26): a scripted
  one-attempt-per-mode rerun of Days 1-10 with a per-challenge solver. The original attempts are often only
  journal-recorded. Days 8 and 9 are the strongest: all 15 and 14 modes hold our own graded files.
- **Saved artifacts were corrupted by line-ending translation** (LF to CRLF on binary ZIPs and APKs), which
  made several Day 1-5 deliveries unreadable until re-downloaded and hash-checked. The fix was a rule: verify
  the SHA-256 and the `PK` magic before analysis.
- **Some attempts were lost to infrastructure or our own scripts**, for example a ranked rerun that failed
  because an isolated interpreter hid the package needed to open an encrypted ZIP, after the attempt had
  already been issued and spent. Those are in the relevant day overviews, not the challenge pages.

## Tooling and process, in order

| When | Change | Where it shows |
|---|---|---|
| 2026-10-18 | First play; "local mark" entries for early Single Solves, no repo history yet | [Day 1](day01/README.md), [Day 4](day04/README.md) |
| 2026-10-20 | Retroactive audit and fresh re-solves; library curation sweep (codec chain, BRE reader, APK asset codecs, variant-corpus regression) | [Day 2](day02/README.md), [Day 3](day03/README.md) |
| 2026-10-20/23 | Web Exploitation lane debuts (Day 19), solved by hand: live `targets[]` instead of an artifact, a hidden header clock on Time Trial, Mastery bundles that mix vulnerability classes across targets | [Day 19](day19/README.md) |
| 2026-10-21 | Release automation (watcher + orchestrator) runs live for the first time, non-web lanes only; the web lane's worker/solver integration does not exist yet | [Day 21](day21/README.md) |
| 2026-10-22 | Repository history begins (bulk import of Days 1-10 leaves, technique library); Web Exploitation wired into the unattended pipeline, first full six-lane unattended run | [Day 22](day22/README.md), any day overview |
| 2026-10-23 | `gm.py artifact` gains `--capability`/`--name` for multi-delivery Mastery; `practice` for exhausted pools | [Day 5](day05/README.md), [Day 8](day08/README.md) |
| 2026-10-24 | Stall escalation to a stronger model (Opus) adopted as a standard procedure, after a genuine multi-round stall | [Day 23](day23/README.md), [Day 24](day24/README.md) |
| 2026-10-25/26 | Ranked sweep tooling: one-pass issue-download-solve-submit script, per-challenge solvers, cross-machine parity checks; escalated submission cap added (20 per ID+mode, 70 once a lane escalates) | [Day 2](day02/README.md), [Day 10](day10/README.md), [Day 26](day26/README.md) |
| 2026-10-27/28 | A full release run misreads `already_solved` as a systematic rejection, escalates needlessly, and never reaches Time Trial or Mastery for five lanes; fixed by treating local evidence (never Game Master status) as the only proof of our own solve, adding a `team_solved.json` marker, and reissuing a fresh `retry` when `start` returns the team's readout | [Day 27](day27/README.md), [Day 28](day28/README.md) |
| 2026-10-28 | Team-attributed reruns for modes that ranked only under a solo identity; `finish_open.py` for stale open attempts; the orchestrator stops halting a whole challenge just because its Single Solve is team-solved | [Day 9](day09/README.md), [Day 28](day28/README.md) |
| 2026-10-29 | Release-day supervisor (watchdog, orphaned-lane resume, an Opus resolver via `resume_lane.py`) and the `manual_solve.json` marker for hand-solved leaves | [Day 29](day29/README.md) |
| 2026-10-31 | Day 30's SHA1-Hulud flag-checker solved offline: isolating self-decrypting VEH walls and collapsing mixed-boolean-arithmetic chains to clean-run constants reproduces the flag, with no gradable submission behind it | [Day 30](day30/README.md) |
| 2026-10-31 to 11-01 | Gap-closure reruns toward the speed scoreboard; a parallel-submission race and an escalated-cap-on-resume bug found and fixed on the event's final day | [Day 8](day08/README.md), [Day 10](day10/README.md), [Day 31](day31/README.md) |

Release automation and the Web Exploitation lane are covered in detail in the Days 11-31 progression below.

## Progression, Days 11-31

### Days 11-20: the library absorbs the load

Days 11-20 kept the shape Days 6-10 had set: one challenge per lane, three modes each, and a cumulative
answer bank in Binary Reverse Engineering and Script Analysis that grew by one new block a day (Script's
descriptor bank passed 1,000 fields around Day 14 and kept climbing). Four rules from the Days 6-10
validator work carried the whole stretch: the descriptor's own field beats the printed graph rule, Batch
`*-control` answers are structural and have to be read from the specimen (never copied from another day or
mode), every answer value is a JSON string, and a Python staged chain ending in `dispatch_process` takes
the language sub-technique where Bash and Node.js chains of the same shape often take `T1105` (the exact
split flipped by day and by id schema more than once; see [Day 11](day11/README.md) and
[Day 23](day23/README.md)). Provenance on this stretch is mixed: several days (11, 12, 17, 18, 20) have
more Journal-recorded modes than Independent ones, because the original 2026-10-20 solves kept no saved
response and the Independent label rests on a later ranked rerun. [Day 16](day16/README.md) is the
cleanest day in the stretch (all 15 modes Independent on the first submission); [Day 12](day12/README.md)
the thinnest (3 of 15).

Day 19 added the Web Exploitation lane: no artifact, live `targets[]`, a Time Trial clock that lived in the
live target's own response headers rather than the Game Master's `start` payload, and Mastery bundles that
mixed the challenge's own vulnerability class with earlier-day classes (usually a `robots.txt` to
`.json.bak` backup disclosure) rather than three copies of one app. Every later web day repeated that last
pattern. The three Day 19 web challenges were solved interactively, since release automation did not exist
yet.

### Days 21-24: release automation takes over

[Day 21](day21/README.md) is the first day with a `challenges/dayNN/_run/` folder: a watcher polling for
the 09:00 ET unlock, an orchestrator that recons every lane and drives headless solver sessions, and a
submission ledger with a per-ID+mode cap. It covered the five non-web lanes only; Web Exploitation's
worker/solver integration did not exist yet, so that lane was still solved by hand.
[Day 22](day22/README.md) wired the web lane in and ran the first fully unattended six-lane day: the
non-web lanes finished in minutes, and the web lane needed two tooling fixes (a submission-merge bug that
could discard an already-correct answer, and solvers that assumed a Mastery bundle had only one target)
before its ranked Mastery went through. [Day 23](day23/README.md) and [Day 24](day24/README.md) added the
stall-escalation-to-Opus procedure (documented 2026-10-24, after a genuine multi-round stall on a web SSRF
Mastery): when a lane's solvers stall for two rounds, a stronger model gets one fresh-context attempt at a
written brief, and the lane's submission cap rises from 20 to 70. Both days were otherwise more of the
same: one new field or language per lane on top of a cumulative bank, decoy prompts, and Mastery bundles
mixing web vulnerability classes.

### Days 25-28: the automation's own failure modes

Four days in a row produced a genuine process bug, not just a validator surprise. [Day 26](day26/README.md)'s
Trust the Webhook was scored by a teammate about two hours after unlock, and chasing a wrong Host-header
theory for it needed several Opus passes before an interactive session found the real vector (the clue
endpoint simply named the wrong field keys). [Day 27](day27/README.md) is the low point: a teammate had
already scored all five non-web Single Solves within minutes of unlock, the orchestrator read every
`already_solved` reply as a systematic rejection, escalated all five lanes to Opus against unchanged
solvers, and reported "all lanes finished" at the end of the day with Time Trial and Mastery for those five
lanes never attempted. They were solved on a fresh `retry` about 17.6 hours later, once a player session
and a same-day fix taught the orchestrator that local evidence — never Game Master status — is the only
proof of our own solve, and that a `start` returning the team's readout should trigger a fresh `retry`, not
a halt. [Day 28](day28/README.md) exposed the same bug's twin: "If a Breach Falls in a Log" stalled on a
team-solved Single Solve and the pipeline marked the *whole challenge* stuck without ever trying Time Trial
or Mastery, even though both support an independent `retry`; a same-day fix made the orchestrator skip
straight to team-solved bookkeeping and keep going. Both fixes are why later Team-solved Single Solves
(Days 26-31) still had their Time Trial and Mastery attempted and won independently wherever those modes
exist — the one exception is Day 30's SHA1-Hulud, which has no Time Trial or Mastery to retry at all.

### Day 29: a steady state

By Day 29 the pattern had stabilized: a seven-challenge, six-lane day with a release-day supervisor
watching the orchestrator, finished in 23 minutes with no lane needing the player. Two Single Solves were
team-scored; every other mode — the remaining five Single Solves and all fourteen Time Trial and Mastery
modes — was graded on our own submissions. See the [Day 29 overview](day29/README.md) for the detail; it
sits here as the stretch's waypoint, the day the automation finally ran the way it was meant to.

### Day 30: SHA1-Hulud and an ungradable solve

[Day 30](day30/README.md) added a one-off extra Binary Reverse Engineering challenge alongside the usual
six-lane day: SHA1-Hulud, a flag-checking crackme with no Time Trial or Mastery at all. A teammate solved
its Single Solve about three hours after unlock, confirmed live as un-reissuable (`retry` returns 405,
`practice` returns 409), which makes it Team solve (not ours) and nothing more. We still reverse-engineered
it independently afterward: isolating each self-decrypting VEH wall in an emulation harness, reimplementing
the walls in Python, and collapsing the mixed-boolean-arithmetic chains that gate the flag to closed-form
constants, which reproduced the flag byte-for-byte. That work is real and ours, but the Game Master never
graded it and there is no mode left to submit it to, so the honest record is Team solve (not ours) for
Single Solve and Absent for Time Trial and Mastery, not a disguised Independent. Four of the day's other
seven Single Solves were also team-scored; all fourteen Time Trial and Mastery modes across the day's eight
challenges were our own graded CORRECT.

### Day 31: the finale

Day 31 was a 24-hour window (2026-10-31 13:00 UTC through the event's close the next morning), seven
challenges across six lanes, each one an explicit "everything you've learned" recap plus one small new
addition (a trailer-record format reused under a new magic byte, an execution-chain graph question, one
more scripting language). Six of the seven Single Solves were scored by a teammate within seconds to
minutes of unlock; every other mode — all seven Time Trial, all seven Mastery, and the one remaining Single
Solve — was graded on our own submissions, including recovering from two same-day automation bugs: a
parallel-submission race that could discard a just-earned CORRECT when a sibling call on the same
now-closed attempt came back `already_solved`, and a resume path that dropped a lane's escalated-cap
status. The team's final event standing was 41st of 1,594, per the project's own record of the run.

For the finale the player ran four machines in parallel with a strict division of roles, to get a second
opinion on every lane without widening who could act against the Game Master. The main workstation was the
only machine that ever submitted an answer or executed a downloaded sample, and it ran the release
automation end to end. A second machine, a reversing VM, and a smaller, lighter-weight machine each ran an
agent in a candidate-only capacity — deriving answers (the reversing VM on crackme-style binary work, the
lighter machine on a second opinion for Warmups and Metadata Forensics) and feeding them back for
comparison, with no path of their own to submit or to run untrusted code. That boundary was written down
the night before release and held for the whole window.

## Index

### Day overviews

[Day 1](day01/README.md) · [Day 2](day02/README.md) · [Day 3](day03/README.md) · [Day 4](day04/README.md) ·
[Day 5](day05/README.md) · [Day 6](day06/README.md) · [Day 7](day07/README.md) · [Day 8](day08/README.md) ·
[Day 9](day09/README.md) · [Day 10](day10/README.md) · [Day 11](day11/README.md) · [Day 12](day12/README.md) ·
[Day 13](day13/README.md) · [Day 14](day14/README.md) · [Day 15](day15/README.md) · [Day 16](day16/README.md) ·
[Day 17](day17/README.md) · [Day 18](day18/README.md) · [Day 19](day19/README.md) · [Day 20](day20/README.md) ·
[Day 21](day21/README.md) · [Day 22](day22/README.md) · [Day 23](day23/README.md) · [Day 24](day24/README.md) ·
[Day 25](day25/README.md) · [Day 26](day26/README.md) · [Day 27](day27/README.md) · [Day 28](day28/README.md) ·
[Day 29](day29/README.md) · [Day 30](day30/README.md) · [Day 31](day31/README.md)

### Challenge pages

| Day | Warmups | Metadata Forensics | Script Analysis | Mobile Android | Binary RE | Other |
|---|---|---|---|---|---|---|
| 1 | [A Little Based](day01/warmups_a-little-based.md) | [You Must Be This Hash to Ride](day01/metadata-forensics_you-must-be-this-hash-to-ride.md) | [Power Play](day01/script-analysis_power-play.md) | [Exported Trust](day01/mobile-android_exported-trust.md) | [ELF Aware](day01/binary-reverse-engineering_elf-aware.md) | Welcome: [Welcome Robots](day01/welcome_welcome-robots.md), [Read the Rules](day01/welcome_read-the-rules.md) |
| 2 | [Half a Turn](day02/warmups_half-a-turn.md) | [Looks Can Be Deceiving](day02/metadata-forensics_looks-can-be-deceiving.md) | [Host With the Most](day02/script-analysis_host-with-the-most.md) | [Deep Link Drift](day02/mobile-android_deep-link-drift.md) | [Mangled](day02/binary-reverse-engineering_mangled.md) | |
| 3 | [Hex Appeal](day03/warmups_hex-appeal.md) | [BOM Voyage](day03/metadata-forensics_bom-voyage.md) | [Expansion Pack](day03/script-analysis_expansion-pack.md) | [Intent Injection](day03/mobile-android_intent-injection.md) | [Table Manners](day03/binary-reverse-engineering_table-manners.md) | |
| 4 | [Two of a Kind](day04/warmups_two-of-a-kind.md) | [Picture This](day04/metadata-forensics_picture-this.md) | [Dim and Dimmer](day04/script-analysis_dim-and-dimmer.md) | [Pending Intent Relay](day04/mobile-android_pending-intent-relay.md) | [Go Fish](day04/binary-reverse-engineering_go-fish.md) | |
| 5 | [By the Numbers](day05/warmups_by-the-numbers.md) | [Chunk Norris](day05/metadata-forensics_chunk-norris.md) | [Hatch Careful](day05/script-analysis_hatch-careful.md) | [Backup Spill](day05/mobile-android_backup-spill.md) | [Rust Belt](day05/binary-reverse-engineering_rust-belt.md) | |
| 6 | [Eight Is Enough](day06/warmups_eight-is-enough.md) | [Portable Deception](day06/metadata-forensics_portable-deception.md) | [Shell We Begin](day06/script-analysis_shell-we-begin.md) | [Provider Paths](day06/mobile-android_provider-paths.md) | [Pool Party](day06/binary-reverse-engineering_pool-party.md) | |
| 7 | [Percent Chance](day07/warmups_percent-chance.md) | [Seven-Zip, No Relation](day07/metadata-forensics_seven-zip-no-relation.md) | [Node by Node](day07/script-analysis_node-by-node.md) | [WebView Bridge](day07/mobile-android_webview-bridge.md) | [Routine Check](day07/binary-reverse-engineering_routine-check.md) | |
| 8 | [Entity Relations](day08/warmups_entity-relations.md) | [What's in the Box?](day08/metadata-forensics_whats-in-the-box.md) | [Lost in Translation](day08/script-analysis_lost-in-translation.md) | [Cleartext Session](day08/mobile-android_cleartext-session.md) | [Swift Justice](day08/binary-reverse-engineering_swift-justice.md) | |
| 9 | [Escape Room](day09/warmups_escape-room.md) | [Built Different](day09/metadata-forensics_built-different.md) | [All the World's a Stager](day09/script-analysis_all-the-worlds-a-stager.md) | [Trust Everyone](day09/mobile-android_trust-everyone.md) | [Nimble](day09/binary-reverse-engineering_nimble.md) | |
| 10 | [Thirty-Two Below](day10/warmups_thirty-two-below.md) | [Where Do We Begin?](day10/metadata-forensics_where-do-we-begin.md) | [Bind Over Matter](day10/script-analysis_bind-over-matter.md) | [Network Config Split](day10/mobile-android_network-config-split.md) | [Zig Zag](day10/binary-reverse-engineering_zig-zag.md) | |
| 11 | [Safe Travels](day11/warmups_safe-travels.md) | [What's My Name Again?](day11/metadata-forensics_whats-my-name-again.md) | [By Any Other Name](day11/script-analysis_by-any-other-name.md) | [Local Secrets](day11/mobile-android_local-secrets.md) | [D-Day](day11/binary-reverse-engineering_d-day.md) | |
| 12 | [Back at You](day12/warmups_back-at-you.md) | [Managed Expectations](day12/metadata-forensics_managed-expectations.md) | [AutoIt Takes Two](day12/script-analysis_autoit-takes-two.md) | [SQLite Assumptions](day12/mobile-android_sqlite-assumptions.md) | [Pascal's Wager](day12/binary-reverse-engineering_pascals-wager.md) | |
| 13 | [Opposite Day](day13/warmups_opposite-day.md) | [Product Placement](day13/metadata-forensics_product-placement.md) | [Zip Code](day13/script-analysis_zip-code.md) | [Keystore Misuse](day13/mobile-android_keystore-misuse.md) | [Crystal Clear](day13/binary-reverse-engineering_crystal-clear.md) | |
| 14 | [Seven Steps](day14/warmups_seven-steps.md) | [The Shortcut Home](day14/metadata-forensics_the-shortcut-home.md) | [Trust the Process](day14/script-analysis_trust-the-process.md) | [Predictable Crypto](day14/mobile-android_predictable-crypto.md) | [Selective Memory](day14/binary-reverse-engineering_selective-memory.md) | |
| 15 | [Room Forty-Seven](day15/warmups_room-forty-seven.md) | [A Word to the Wise](day15/metadata-forensics_a-word-to-the-wise.md) | [Crumb Trail](day15/script-analysis_crumb-trail.md) | [Native Library Secrets](day15/mobile-android_native-library-secrets.md) | [Stripped](day15/binary-reverse-engineering_stripped.md) | |
| 16 | [Dot Com](day16/warmups_dot-com.md) | [Return to Sender](day16/metadata-forensics_return-to-sender.md) | [Same Same, But Different](day16/script-analysis_same-same-but-different.md) | [JNI Boundary](day16/mobile-android_jni-boundary.md) | [Wasm Up](day16/binary-reverse-engineering_wasm-up.md) | |
| 17 | [Bacon Bits](day17/warmups_bacon-bits.md) | [Hear Me Out](day17/metadata-forensics_hear-me-out.md) | [Fetch Quest](day17/script-analysis_fetch-quest.md) | [Name Collapse](day17/mobile-android_name-collapse.md) | [Decoder Ring](day17/binary-reverse-engineering_decoder-ring.md) | |
| 18 | [Off the Rails](day18/warmups_off-the-rails.md) | [Frame of Reference](day18/metadata-forensics_frame-of-reference.md) | [Orphan Process](day18/script-analysis_orphan-process.md) | [Encrypted Strings](day18/mobile-android_encrypted-strings.md) | [Flow State](day18/binary-reverse-engineering_flow-state.md) | |
| 19 | [Little Things](day19/warmups_little-things.md) | [Little Endian on the Prairie](day19/metadata-forensics_little-endian-on-the-prairie.md) | [Sharp Practice](day19/script-analysis_sharp-practice.md) | [Reflection Maze](day19/mobile-android_reflection-maze.md) | [Jack in the Box](day19/binary-reverse-engineering_jack-in-the-box.md) | Web: [Back Up a Minute](day19/web-exploitation_back-up-a-minute.md), [Permission to Slip](day19/web-exploitation_permission-to-slip.md), [Query Me This](day19/web-exploitation_query-me-this.md) |
| 20 | [Big Entrance](day20/warmups_big-entrance.md) | [Universal Appeal](day20/metadata-forensics_universal-appeal.md) | [The Butler Did It](day20/script-analysis_the-butler-did-it.md) | [Loaded Later](day20/mobile-android_loaded-later.md) | [Resourceful](day20/binary-reverse-engineering_resourceful.md) | Web: [Stay in Your Directory](day20/web-exploitation_stay-in-your-directory.md), [Administratively Speaking](day20/web-exploitation_administratively-speaking.md) |
| 21 | [Under Pressure](day21/warmups_under-pressure.md) | [Package Deal](day21/metadata-forensics_package-deal.md) | [Actions Speak Louder](day21/script-analysis_actions-speak-louder.md) | [Debug Me If You Can](day21/mobile-android_debug-me-if-you-can.md) | [Shared Custody](day21/binary-reverse-engineering_shared-custody.md) | Web: [Session Impression](day21/web-exploitation_session-impression.md), [Stored for Later](day21/web-exploitation_stored-for-later.md), [Reflect Upon This](day21/web-exploitation_reflect-upon-this.md) |
| 22 | [Deflated Ego](day22/warmups_deflated-ego.md) | [Main Character Energy](day22/metadata-forensics_main-character-energy.md) | [Next, Next, Finish](day22/script-analysis_next-next-finish.md) | [Root of Trust](day22/mobile-android_root-of-trust.md) | [Bridge Troll](day22/binary-reverse-engineering_bridge-troll.md) | Web: [Forged With Care](day22/web-exploitation_forged-with-care.md), [Uploading Expectations](day22/web-exploitation_uploading-expectations.md) |
| 23 | [Eighty-Five Alive](day23/warmups_eighty-five-alive.md) | [Page Me Maybe](day23/metadata-forensics_page-me-maybe.md) | [Press Start to Decode](day23/script-analysis_press-start-to-decode.md) | [Hook Line and Signal](day23/mobile-android_hook-line-and-signal.md) | [Trust Issues](day23/binary-reverse-engineering_trust-issues.md) | Web: [Template Temper](day23/web-exploitation_template-temper.md), [Ask Again Later](day23/web-exploitation_ask-again-later.md), [Posted Too Much](day23/web-exploitation_posted-too-much.md) |
| 24 | [String Theory](day24/warmups_string-theory.md) | [Event Horizon](day24/metadata-forensics_event-horizon.md) | [Web of Lies](day24/script-analysis_web-of-lies.md) | [Emulator Alibi](day24/mobile-android_emulator-alibi.md) | [Patch Diffing](day24/binary-reverse-engineering_patch-diffing.md) | Web: [The API Has Opinions](day24/web-exploitation_the-api-has-opinions.md), [Fetch Quest](day24/web-exploitation_fetch-quest.md) |
| 25 | [You Have a Point](day25/warmups_you-have-a-point.md) | [Hive Mind](day25/metadata-forensics_hive-mind.md) | [Ashes to Assets](day25/script-analysis_ashes-to-assets.md) | [Integrity Signals](day25/mobile-android_integrity-signals.md) | [Out of Bounds](day25/binary-reverse-engineering_out-of-bounds.md) | Web: [Shell Company](day25/web-exploitation_shell-company.md), [Reset Expectations](day25/web-exploitation_reset-expectations.md), [Resolver Revolt](day25/web-exploitation_resolver-revolt.md) |
| 26 | [Food for Thought](day26/warmups_food-for-thought.md) | [Before You Go](day26/metadata-forensics_before-you-go.md) | [Mixed Signals](day26/script-analysis_mixed-signals.md) | [Split Decision](day26/mobile-android_split-decision.md) | [Crash Course](day26/binary-reverse-engineering_crash-course.md) | Web: [Serialized Offender](day26/web-exploitation_serialized-offender.md), [Trust the Webhook](day26/web-exploitation_trust-the-webhook.md) |
| 27 | [Monkey Business](day27/warmups_monkey-business.md) | [Packet Loss Prevention](day27/metadata-forensics_packet-loss-prevention.md) | [FROM Whence It Came](day27/script-analysis_from-whence-it-came.md) | [Dependency Surface](day27/mobile-android_dependency-surface.md) | [Compiler Says What](day27/binary-reverse-engineering_compiler-says-what.md) | Web: [Double Spent](day27/web-exploitation_double-spent.md), [Socket to Me](day27/web-exploitation_socket-to-me.md), [Forwarded Misgivings](day27/web-exploitation_forwarded-misgivings.md) |
| 28 | [Back in Sixteen](day28/warmups_back-in-sixteen.md) | [Where It All Began](day28/metadata-forensics_where-it-all-began.md) | [Out of Order](day28/script-analysis_out-of-order.md) | [Protocol Reconstruction](day28/mobile-android_protocol-reconstruction.md) | [Family Resemblance](day28/binary-reverse-engineering_family-resemblance.md) | Web: [Fail Open Friday](day28/web-exploitation_fail-open-friday.md), [If a Breach Falls in a Log](day28/web-exploitation_if-a-breach-falls-in-a-log.md) |
| 29 | [Rotten Base](day29/warmups_rotten-base.md) | [Trust Issues](day29/metadata-forensics_trust-issues.md) | [Make It Make Sense](day29/script-analysis_make-it-make-sense.md) | [White Box Key Recovery](day29/mobile-android_white-box-key-recovery.md) | [Fraternal Twins](day29/binary-reverse-engineering_fraternal-twins.md) | Web: [Trust the Build](day29/web-exploitation_trust-the-build.md), [Internal Affairs](day29/web-exploitation_internal-affairs.md) |
| 30 | [Percent Percent](day30/warmups_percent-percent.md) | [Seeing Double](day30/metadata-forensics_seeing-double.md) | [Spill the Beans](day30/script-analysis_spill-the-beans.md) | [Layered Protections](day30/mobile-android_layered-protections.md) | [Campaign Trail](day30/binary-reverse-engineering_campaign-trail.md), [SHA1-Hulud](day30/binary-reverse-engineering_sha1-hulud.md) | Web: [Pass the Identity](day30/web-exploitation_pass-the-identity.md), [Queue the Confusion](day30/web-exploitation_queue-the-confusion.md) |
| 31 | [One More Thing](day31/warmups_one-more-thing.md) | [No Artifact Left Behind](day31/metadata-forensics_no-artifact-left-behind.md) | [The Full Transform](day31/script-analysis_the-full-transform.md) | [Mobile Arena Gauntlet](day31/mobile-android_mobile-arena-gauntlet.md) | [Final Boss](day31/binary-reverse-engineering_final-boss.md) | Web: [Framework Roulette](day31/web-exploitation_framework-roulette.md), [Full Stack Incident](day31/web-exploitation_full-stack-incident.md) |

This set now covers the full event, Days 1-31.
