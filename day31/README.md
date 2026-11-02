# Day 31 overview

Day 31 was the event's finale: a 24-hour window released 2026-10-31 13:00 UTC, with the event
closing the following morning. Seven challenges across six lanes (Web Exploitation carried two:
a carryover from the prior day's curriculum numbering, plus the day's own new challenge), 21
graded modes. Every lane's capstone challenge was an explicit "everything you've learned" recap on
top of one new addition. The team's final event standing was **41st of 1594** — stated here as an
established project record (the commit message for the Day 31 run states it directly; neither
`_run/closeout.md` nor `_run/supervisor-report.md` carries a final standing, so this page does not
cite either as its source).

**Provenance check:** all seven challenge pages' labels match `progress.md` and the leaf
`result.json`/`accepted_plan.json`/`team_solved.json` evidence exactly — **no mismatches found**.
15 of 21 modes are Independent (every Time Trial and every Mastery, plus one Single Solve); 6 are
Team solve (not ours) (every other Single Solve, scored by a teammate within seconds to minutes of
unlock, before our own submission landed).

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [One More Thing](warmups_one-more-thing.md) | Layered text codec (base32 -> gzip -> base64); new per-record-31 codec table entry | Team solve (not ours) | Independent | Independent |
| Metadata Forensics | [No Artifact Left Behind](metadata-forensics_no-artifact-left-behind.md) | Execution-chain graph reconstruction across mixed-format artifacts, on a full Day 1-30 recap | Team solve (not ours) | Independent | Independent |
| Script Analysis | [The Full Transform](script-analysis_the-full-transform.md) | Descriptor-driven analysis; one new language (XSLT) folded into a 31-language finale recap | Team solve (not ours) | Independent | Independent |
| Mobile Android | [Mobile Arena Gauntlet](mobile-android_mobile-arena-gauntlet.md) | Running each APK's own bytecode in the lane's Dalvik mini-VM (`puzzle.v2`/`analysis` codec, every delivery) | Team solve (not ours) | Independent | Independent |
| Binary Reverse Engineering | [Final Boss](binary-reverse-engineering_final-boss.md) | Static container classification plus a new HCFG-layout trailer record (`HGAU`), on a 30-day cumulative recap | Team solve (not ours) | Independent | Independent |
| Web Exploitation | [Framework Roulette](web-exploitation_framework-roulette.md) | Reverse-proxy trusted-header spoofing, two-claim variant; Mastery mixed in 5 other classes | Team solve (not ours) | Independent | Independent |
| Web Exploitation | [Full Stack Incident](web-exploitation_full-stack-incident.md) | Unsigned fetch-to-worker replay via a method-gap signature bypass; Mastery mixed in 7 other classes | Independent | Independent | Independent |

## Timeline (UTC, 2026-10-31 unless noted)

| Time | Event |
|---|---|
| 12:50:01 | Release watcher and usage daemon start; preflight against the Day 30 list; lead time 300 s |
| 12:55:00 | Polling window opens (T-300 s) |
| 13:00:03.854 | Day 31 detected after 212 polls (T+3.85 s): 21 modes across 7 challenges / 6 lanes; `release_list.json` saved |
| 13:00:04 | Orchestrator launches all lanes, authorized cap 30 (70 once a lane escalates) |
| ~13:01:43 | Mobile Android Time Trial and Mastery (regular attempts) both CORRECT — the fastest lane |
| ~13:02:53 | Warmups Time Trial and Mastery CORRECT |
| 13:05:45.541 | Metadata-Forensics Time Trial genuinely goes CORRECT on attempt `ea1cbcb3`, but a parallel-submission race in the release tooling discards it as `already_solved` (see "Mistakes in our own automation") |
| 13:08:11 | Metadata-Forensics lane escalates to a stronger model (fix round 2), chasing the now-closed attempt |
| 13:08:38 | Five of six lanes report no more automation needed; only Script Analysis still active |
| 13:08:54 | Script Analysis lane finishes; librarian pass starts for it |
| 13:09:36 | A fresh Metadata-Forensics Time Trial attempt (`d97cb5cc`) is opened elsewhere on the team while the stale attempt is still being chased; 28 submission calls are spent against the closed/superseded attempt before the race is recognized |
| ~13:16:32 | Binary Reverse Engineering Time Trial and Mastery CORRECT |
| 13:23:33-13:25:08 | Two resume passes for Metadata-Forensics; the first hits `cap_reached` (34/30 — the resumed lane's escalated-cap flag was not restored); the second, after both bugs are fixed, drives Time Trial CORRECT (2 calls) and Mastery CORRECT (1 call) |
| 13:25:22 | Metadata-Forensics lane finishes; librarian pass runs |
| 13:27:07 | Web Exploitation Full Stack Incident Mastery genuinely goes CORRECT on our own attempt (`3c4e4212...`, opened 13:12:00) — also mis-recorded as `already_solved` by the same race bug, not yet recognized |
| 14:01:38-14:14:19 | Remaining librarian passes run in a batch: Binary RE, Mobile Android, Warmups, Full Stack Incident |
| 14:25:35 | Supervisor report: 5 lanes **ok**, Metadata-Forensics and Script Analysis **RESUMED**, Full Stack Incident **RESOLVED** (resolver reconstructs the true CORRECT and writes `manual_solve.json`), Framework Roulette **BLOCKED** (needs the player — gate-2 has no non-fuzzing channel) |
| later 10-31 | Framework Roulette's gate-2 channel is found outside the automated pipeline (a fresh read of the `Vary` response header); Time Trial and Mastery are solved interactively, CORRECT on the first submission call each; Single Solve stays team-scored and un-reissuable |
| into 2026-11-01 | Ranked/speed passes continue: Full Stack Incident Mastery's ledger cap is raised 70 -> 110 to recover a stuck open attempt and finish a ranked CORRECT; Framework Roulette's ranked Time Trial and Mastery are re-run from a lower-latency machine for scoreboard rank |

## Submission effort

Regular (untimed) attempts are what count toward the Day 30+ prize rule (time from release to our
own CORRECT on each of Single Solve, Time Trial and Mastery, once each); ranked attempts below are
bragging-rights only and did not affect scoring.

| Challenge | Single Solve | Time Trial (regular) | Mastery (regular) |
|---|---|---|---|
| One More Thing | team (0 calls) | 1 | 1 |
| No Artifact Left Behind | team (0 calls) | 36 (28 spent chasing a closed/superseded attempt) | 1 |
| The Full Transform | team (6 calls, 0 accepted) | 7 | 7 |
| Mobile Arena Gauntlet | team (1 call) | 1 | 1 |
| Final Boss | team (1 call) | 7 | 1 |
| Framework Roulette | team (0 calls, ledger 0/70) | 1 (interactive) | 1 (interactive, 6/6) |
| Full Stack Incident | 1 | 1 | 18 |

Ranked-attempt local totals (`t_total`, where recorded) and server deadlines:

| Challenge | Time Trial ranked | Mastery ranked |
|---|---|---|
| One More Thing | 3.62 s / 30 s window | 7.23 s / 170 s window |
| No Artifact Left Behind | 3.43 s / 50 s window | 5.41 s / 480 s window |
| The Full Transform | 4.68 s / 50 s window | 14.19 s / 610 s window |
| Mobile Arena Gauntlet | 3.62 s / 50 s window | 8.18 s / 470 s window |
| Final Boss | 6.46 s / 80 s window | 37.71 s / 715 s window |
| Framework Roulette | 896 ms, rank 1/34 on the public speed scoreboard | 3,394 ms, rank 6/21 |
| Full Stack Incident | 11.28 s | 19.55 s (final attempt, after the cap raise and solver fixes) |

Framework Roulette's two ranked rows are the only ones where a scoreboard rank is recorded in
`progress.md`; both were achieved by re-running the same exploit from a lower-latency machine
(progression on the Time Trial alone: 12,946 ms -> 1,571 ms -> 896 ms).

## What the day had in common

- **Every lane's capstone was a recap wearing a dramatic title.** "Final Boss" (Binary), "The Full
  Transform" (Script, 31 languages), "Mobile Arena Gauntlet" (15 Mastery sets rebuilding Days 1-31),
  "No Artifact Left Behind" (Metadata, every prior format reader) and "One More Thing" (Warmups, the
  per-record codec table reaching 1-31) all re-asked hundreds to tens-of-thousands of previously
  solved fields and added one small new block each (Binary: 14 of 434 fields; Script: 44 of 1,846;
  Warmups: one new record number). The existing cumulative libraries resolved the recap fields with
  no code change in every case; only the new block needed work.
- **Declared-descriptor-wins held for a sixth straight new-language day** (Script Analysis's XSLT,
  after Elixir, Erlang, Dockerfile, Makefile and BeanShell): the validator took each specimen's own
  `role`/`attack` field over a plausible derived alternative (`T1220` for XSL script processing was
  never accepted).
- **Web Exploitation Mastery bundles kept mixing vulnerability classes under one challenge's name** —
  Framework Roulette's 6 targets and Full Stack Incident's 8 targets each matched the challenge's own
  prompt on only one target; the rest were already-catalogued classes (session fixation, CSRF-via-GET,
  upload content-type confusion, SSTI, stored-XSS bot exfiltration, activity-feed IDOR, paren-wrapped
  SQLi, PHP LFI, bare BAC, reflected-XSS conjunction gate) plus, for the first time this event, a
  Gin/Go path-traversal class found live during Full Stack Incident's ranked speed work.
- **No new artifact format this day** — both new mechanics (Binary's `HGAU` trailer record, Metadata's
  `graph.*` execution-chain field) were generalizations of a byte layout or reasoning pattern the
  library already had (the HCFG-family record reader; per-format "next hop" readers), not a new file
  type.

## Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Binary | The `HGAU` trailer record reuses the exact 12-byte `HCFG`/`HGRF` layout under a new magic; a mutex string with one literal backslash was accepted at variant 0 again, and the Mastery ranked bundle delivered set 1 with no `setN.` prefix (an already-known pattern, not an error). |
| Metadata | PowerShell launchers with a genuine quote-parse error are still part of the accepted execution chain in a third of the Mastery sets — dropping "malformed" edges disagreed with the validator. A bare-name lookup for the nested `case-001.zip` silently grabbed set 1's case for every set unless resolved through the question's own `artifact` chain. |
| Script | The new XSLT family's descriptor marker appears twice (an XML comment and a bare `<descriptor>` element); only the comment copy was ever exercised. The MITRE-plausible `T1220` alternate was never accepted over the declared value. |
| Mobile | The Single Solve itself shipped the `v2`/`analysis` codec this time (every Days 28-30 Single Solve had shipped a new `v1` stored codec instead) — breaking the pattern a solver might have assumed. |
| Warmups | The prompt names its layers in encode order ("thirty-two ... then sixty-four"), the opposite of decode order; record 31 was unmapped at delivery time and relied on the tool's generic auto-detect fallback, validated beforehand with a synthetic rehearsal built from Day 30's data. |
| Web (Framework Roulette) | Gate 2 was genuinely oracle-free — every wrong header name or value returned a byte-identical 403. The channel's header *name* was sitting unnoticed in the gate's own `Vary` response header; the missing *value* was the unabbreviated word `administrator`, not any `admin`/`staff` abbreviation. |
| Web (Full Stack Incident) | A signature check that visibly exists (`POST` and a bare `GET` with no id both reject) was wired to only one HTTP method; a `GET` with the logged event's own id bypassed it entirely. The win condition was a schema-shaped detector/repair submission that self-validates against a 5-check regression block, not a bare proof string. |

## Mistakes in our own automation

These cost submissions or player time but are not challenge or validator lessons.

- **Parallel-submission race discarding a genuine CORRECT (`tools/release/rl.py`).** `submit_plan`
  sent a round's group calls in parallel and collected results with `list(ex.map(...))`. When one
  group call came back CORRECT and, a fraction of a second later, a sibling group's call on the
  same now-closed attempt came back `already_solved`, the sibling's exception was re-raised and the
  already-captured CORRECT was thrown away — the round logged as `already_solved`, which read as
  "nothing graded." This hit two lanes on Day 31:
  - **Metadata-Forensics Time Trial** (13:05:45): the orchestrator treated the discarded CORRECT as a
    failure, escalated to a stronger model, and kept resubmitting against the now-closed attempt even
    after a fresh attempt (`d97cb5cc`) had been issued elsewhere on the team — 28 submission calls
    spent chasing an attempt that was either already solved or already superseded.
  - **Full Stack Incident Mastery** (13:27:07): the same race discarded our own genuine CORRECT on
    attempt `3c4e4212...`; two further retries against the same attempt naturally returned
    `already_solved` again and were misread as a team solve, triggering an unneeded stall escalation.
    A supervisor resolver pass later reconstructed the true outcome from the lane log's submit
    sequence and a read-only status check on that exact attempt id, and recorded it via
    `mastery/manual_solve.json` since the bug had kept it out of `result.json`.
  - Fixed the same day: `rl.py submit_plan` now collects every parallel future and lets a CORRECT win
    over a sibling's `already_solved`/cap/deadline exception.
- **Escalated-cap flag not restored on resume (`tools/release/orchestrate.py`).** `Lane.existing_cands`
  restored the fix-round counter on resume but not the lane's escalated-to-a-stronger-model flag, so a
  resumed lane fell back to the un-escalated submission cap (30) with no fix round left to re-trigger
  escalation to 70. Metadata-Forensics' first resume attempt hit `cap_reached` at 34/30 as a direct
  result. Fixed the same day: resume now re-marks a lane escalated whenever its feedback-round count is
  at or past the escalation threshold, and an `already_solved` regular result issues one fresh attempt
  instead of spawning more fix-round workers.
- **Full Stack Incident's Mastery dispatcher needed several live fixes** before its 18 regular-attempt
  submission calls converged: a solver that only recognized its own challenge class plus a generic
  `proof_<hex>` text scan solved just 2 of 8 targets at first; a one-capture-group regex unpacked as
  two values silently emptied a field via a swallowed exception; and a transient miss on the
  reflected-XSS gate probe let the dispatcher fall through to the generic scan and report a decoy
  `proof_<hex>` string baked into that target's static page as the answer. All three are detailed on
  the challenge page; noted here only because they drove the high call count.
- **Framework Roulette's gate 2 absorbed a one-time, player-authorized bounded brute force** (roughly
  2,500 header-name/value combinations) as a deliberate, scoped exception to the web lane's normal
  no-fuzzing boundary — it came back clean, confirming the channel was outside any guessable
  vocabulary and that the real fix was reading the gate's own `Vary` header, not guessing harder. The
  Single Solve pipeline's resolver pass came back BLOCKED (no automatically-submittable candidate ever
  existed for that mode), which is why no `champion-*.json` file exists for that challenge — not
  because the challenge went unsolved overall.

## Multi-stream setup (sanitized)

For the finale, the player ran four machines in parallel, with a strict division of roles:

- **The main workstation** was the sole submitter to the Game Master. It ran the release automation
  (watcher, orchestrator, supervisor), held the authorization token, and was the only stream that ever
  called a submission endpoint or executed a downloaded sample.
- **A second machine running a general-purpose agent** worked candidate-generation only: it derived
  candidate answers and logged them locally, with no submission path and no execution of downloaded
  samples, as a second opinion across lanes.
- **A reversing VM running a general-purpose agent** focused on crackme-style and binary
  reverse-engineering work, again candidate-only — deriving answers statically without submitting or
  running untrusted samples.
- **A smaller, lighter-weight machine running a lighter model** gave a second opinion specifically on
  the Warmups and Metadata-Forensics lanes, also candidate-only.

All three secondary streams fed their candidates back for the main workstation to compare against its
own results; none of them ever had a path to submit to the Game Master or to execute a sample it
downloaded. This arrangement is recorded in the repository as a strict "no-submit, no-execute" brief
for the secondary streams, committed the night before release.

## Tooling and process changes around this day

From the commit history around 2026-10-31/11-01:

- A dedicated brief for the three candidate-only secondary streams was added the night before release,
  fixing the no-submit/no-execute boundary and giving each stream a per-box focus lane.
- The main Day 31 run commit carries the `rl.py`/`orchestrate.py` fixes above (parallel-submit race,
  escalated-cap restoration on resume) and the day's librarian promotions: a new technique doc for the
  unsigned-fetch-to-worker incident class, a new technique doc for execution-chain graph reconstruction,
  `codec_chain.py`'s record-31 entry, `question_facts.py`'s `HGAU`/tagged-record generalization, and
  `descriptor_decode.py`'s XSLT family support.
- A follow-up commit landed the Framework Roulette header-spoof break (Time Trial and Mastery) once the
  `Vary`-header channel was found outside the automated pipeline.
- Later commits (into 2026-11-01) cover the Full Stack Incident Mastery solver fixes and its recovered
  ranked CORRECT, and Framework Roulette's ranked speed optimization (the lower-latency-machine reruns
  that produced the rank 1/34 and rank 6/21 scoreboard results), as part of a broader top-30 gap-closure
  sweep across many earlier days run at the same time.

## Takeaways

- A "final boss" title does not mean a harder puzzle — on every lane this day, checking the new-field
  count first (against the previous day's bank) correctly predicted that almost all of the work was
  re-running already-built readers, and only a small, well-scoped addition needed new code.
- The two automation bugs this day share one root cause: code that assumes a submission round's result
  is settled as soon as a client-side exception appears, without checking whether a sibling call in the
  same round, or a later call against the same attempt id, had already produced the real answer. Any
  future parallel-submission or resume path should keep a CORRECT it has already observed rather than
  let a later `already_solved`/`cap_reached` overwrite it.
- When a gate gives no oracle at all (every wrong guess looks identical), check whether the gate's own
  response headers are naming the channel before concluding the axis is unguessable — and once a header
  name is confirmed, sweep past abbreviations to the full semantic word.
- Running the fastest-latency machine available for a live web ranked attempt, not just the most capable
  one, was the deciding factor for scoreboard rank on Framework Roulette (12,946 ms -> 1,571 ms -> 896 ms
  across three different runners for the same exploit).

## Evidence

- `challenges/day31/_run/release_list.json`, `orchestrator.log`, `watch.log`, `status.json`,
  `timing_meta.json`, `closeout.md`, `supervisor-report.md`, `supervisor/*.r1.md`
  (metadata-forensics, script-analysis, web-exploitation-day-30-framework-roulette,
  web-exploitation-day-31-full-stack-incident)
- `challenges/day31/_run/resume-metadata-forensics-20261031T062333.log` /
  `...T062508.log`, `challenges/day31/_run/resume-script-analysis-20261031T060837.log`
- `progress.md` (grep `^| 31 `)
- `git log` (2026-10-31/11-01): the "Add remote candidate-only solver brief for Day 31 secondary boxes",
  "Day 31: full run — solved lanes, librarian promotions, progress rows, orchestrate/rl fixes",
  "Day 31 Framework Roulette: Blazor two-gate header-spoof + Mastery 6/6", "Day 31 Full Stack Incident:
  fix solver bugs, ranked Mastery CORRECT" and "Day 31 Framework Roulette: ranked speed optimization"
  commits
