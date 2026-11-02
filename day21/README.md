# Day 21 overview

**At a glance:** 8 challenges (one per lane in Warmups, Metadata Forensics, Script Analysis,
Binary Reverse Engineering and Mobile Android, plus the web-exploitation lane's first 3-challenge
day), 24 modes. By provenance label: 22 Independent, 0 Journal-recorded, 0 Local mark only, 2
Team solve (not ours), 0 n/a. Day 21 is the first day with a `challenges/day21/_run/` folder: the
release watcher and `tools/release/orchestrate.py` ran for the five non-web lanes, but the web lane
was not yet wired into that pipeline (it joined starting Day 22) and its three challenges were
solved interactively instead.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Under Pressure](warmups_under-pressure.md) | Layered base64-then-gzip text decoding, per-record codec table | Independent | Independent | Independent |
| Metadata Forensics | [Package Deal](metadata-forensics_package-deal.md) | Binary AXML parse for an APK's `package` attribute, wrapped in a 26-field recap | Independent | Independent | Independent |
| Script Analysis | [Actions Speak Louder](script-analysis_actions-speak-louder.md) | New GitHub Actions `.yml` polyglot family, declared-descriptor fields | Independent | Independent | Independent |
| Binary Reverse Engineering | [Shared Custody](binary-reverse-engineering_shared-custody.md) | Cumulative recap bundle + ELF `e_machine`/HCFG config record in a shared-object plugin | Independent | Independent | Independent |
| Mobile Android | [Debug Me If You Can](mobile-android_debug-me-if-you-can.md) | `huntress_puzzle.json` asset decode (`reverse` codec); anti-debug theme is decoration | Independent | Independent | Independent |
| Web Exploitation | [Session Impression](web-exploitation_session-impression.md) | Session fixation, cookie name differs from the planting parameter | Team solve (not ours) | Independent | Independent |
| Web Exploitation | [Stored for Later](web-exploitation_stored-for-later.md) | Second-order stored XSS exfiltrated via a moderator review bot | Team solve (not ours) | Independent | Independent |
| Web Exploitation | [Reflect Upon This](web-exploitation_reflect-upon-this.md) | Reflected XSS behind a nonce + `Sec-Fetch-*` conjunction gate | Independent | Independent | Independent |

## Provenance reconciliation

- **Session Impression and Stored for Later Single Solves — confirmed Team solve (not ours).**
  Both leaves hold a `team_solved.json` (`{"reason": "start_already_correct"}`, written
  2026-10-31) and no `result.json`/`accepted_plan.json`/`manual_solve.json`. Each journal narrates
  our own hands-on exploit reaching `CORRECT` on 2026-10-22, but no graded-result file from that
  original attempt was preserved, so the strict "local evidence only" rule means the label cannot
  be upgraded past Team solve (not ours) — the pages apply that correctly.
- **Reflect Upon This Single Solve — confirmed Independent, despite a later `team_solved.json`.**
  The leaf holds both files: `single-solve/accepted_result.json` (`"result": "CORRECT"`, filesystem
  timestamp 2026-10-23 17:37 local) and `single-solve/team_solved.json` (`{"reason":
  "start_already_correct"}`, timestamp 2026-10-31 22:07 local — eight days later). The
  `accepted_result.json` predates the `team_solved.json` by over a week, so it is evidence of our
  own graded solve on 2026-10-23, consistent with `progress.md`'s 2026-10-23 row for this mode and
  with the detailed multi-session investigation in `artifacts/blocker-browser-callback-required.md`.
  The `team_solved.json` was written by a later gap-closure automation resume on 2026-10-31 that
  re-started the mode, read the Game Master's standing `CORRECT` status (by then also matched by a
  teammate), and logged that readout — it does not retroactively undo the earlier `accepted_result.json`.
  The page's Independent label is correct; no edit was needed.

## Release timeline

From `challenges/day21/_run/watch.log`, `orchestrator.log` and `status.json`:

- **Unlock detected** 2026-10-21T13:00:07Z (poll #142, T+7.21s after the scheduled 13:00:00Z
  release), which saved `release_list.json` and handed off to `orchestrate.py --day 21
  --authorized --cap 20`.
- **First orchestrator run** logged at 2026-10-21T15:11:24Z — about 2 hours 11 minutes after the
  detected unlock. The files do not record what filled that gap (no entries exist between the
  13:00:07Z handoff and the 15:11:24Z orchestrator start). That run covered the five non-web lanes
  only; its own log line reads "not automated: ['web-exploitation']", so the web lane was excluded
  by design, not by failure — Web Exploitation's worker/solver integration into
  `tools/release/orchestrate.py` was not added until Day 22 (per `CLAUDE.md`).
- **Non-web lanes finished fast.** Metadata, Script, BRE and Mobile Single Solve, Time Trial and
  Mastery were all CORRECT by 15:15:55Z, about 4.5 minutes after the run started.
- **Warmups lane needed three extra passes** (below), finishing at 15:56:54Z — about 45 minutes
  after the run started, and roughly 2 hours 57 minutes after unlock.
- **Web lane was solved by hand**, not by this run: Session Impression and Stored for Later across
  2026-10-22 (per their journals and file timestamps, all three modes each), Reflect Upon This on
  2026-10-23 after a multi-session investigation of its callback gate.
- **Later gap-closure/ranked-sweep passes** (2026-10-31 and 2026-11-01, visible as repeated
  "resume"/"RANK" runs in `orchestrator.log` and the lane logs) re-ran Time Trial and Mastery for
  ranked-scoreboard attempts on all eight challenges. Some of these reruns ran from a second
  machine (`ranked-vm-1`/`ranked-vm-2` result folders appear in the BRE, Metadata and Mobile
  leaves) rather than the main workstation.
- **No escalation to a stronger model is recorded for Day 21.** `challenges/day21/_run/` contains
  no mention of "opus" anywhere (checked across the lane logs, worker logs, `orchestrator.log` and
  the champion/status files). Warmups needed three worker fix rounds, which is enough to trigger
  the stall-escalation mechanism on later days, but that mechanism (`--escalate-model opus
  --escalate-from 2`) is dated 2026-10-24 in `CLAUDE.md`, after Day 21; the `lane-warmups-day-21.log`
  line "lane escalated in an earlier run (3 fix rounds); cap 70 per id+mode" refers only to the
  submission-cap escalation, not a model change — all three fix-round worker logs show the same
  `adapter`/`rubric` worker pair.

## What the day had in common

- **Cumulative recap bundles continued.** Both Binary Reverse Engineering (Shared Custody, 240 of
  252 Single Solve IDs reused from Day 20) and Metadata Forensics (Package Deal's Time Trial and
  Mastery, 25 of 26 fields reused) were mostly a port of the previous day's accepted map plus one
  small new reader (an HCFG record in a shared-object plugin; an APK's binary-XML `package`
  attribute).
- **Prompts described a theme the questions didn't ask about.** Mobile's "neutralize a clean-room
  anti-debug state machine" had no debugger or state machine in the question; Metadata's "rebuilds
  versus clones" and "borrowed product name" never became fields; Script's "trust boundary"/"keeps
  coming back" framing was answered from a static descriptor, not workflow semantics; BRE's "mind
  the boundaries" prompt had no question about exports or imports at all. Reading the prompt for
  flavor and the start response for the actual schema was the correct move on every lane this day.
- **Declared-descriptor-wins carried over from Day 20 unchanged** for Script Analysis's new GitHub
  Actions family — no new rubric logic was needed, only verification that the existing marker
  decoder handled the new specimen type.
- **The web lane's Mastery bundles mixed vulnerability classes across targets**, as on every other
  web day: of the three Day 21 web challenges' Mastery bundles, only one target per bundle actually
  matched that challenge's own prompt; the others were earlier-day classes (robots.txt → `.json.bak`
  backup disclosure, an activity-feed IDOR).

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Warmups | The slug is the text after the last colon on the last non-empty decoded line; the per-record codec table must include the current day's new record before Mastery can decode at all. |
| Metadata | `e_machine`-style hex fields use padded forms; the APK `package_name` and other fields are taken verbatim with no alternate spelling rejected. |
| Script Analysis | `role`/`attack`/`adapter` for `polyglot-*` and `host-*` IDs take the descriptor's own declared value, including surprising ones (`none` for benign-utility, bare parent `T1059` for launcher/staged-chain) that a graph reading would not have predicted. |
| Binary Reverse Engineering | `e_machine` is the bare integer, not a label, unlike earlier days' `machine` fields; the loader carries no scored HCFG record (decoy), only the plugin does. |
| Mobile Android | The DEX's `SECRET-`/`proof=` strings are decoys; the answer is always in `assets/huntress_puzzle.json` under that day's fixed codec. |
| Web Exploitation | Live-target route slugs, hashed paths and nonces are re-randomized on every fresh `start`/`retry`, so `/api/clues` must be re-read each attempt rather than reusing a prior mode's values; Mastery bundles are not three copies of the same app. |

## Mistakes in our own tooling or process

Collected here, not in the challenge pages, per the writeup brief.

- **Under Pressure (Warmups) Mastery — missing codec record.** `library/warmups/codec-chain/codec_chain.py`
  had no entry for record 21 when the first Mastery attempt ran, so every one of the 103 blobs failed
  to decode ("no slug decoded"). Fix round 1 added `21: "base64-gzip"` to the table; this is the
  normal per-day cost the warmups lane always carries and was caught before any submission.
- **Under Pressure Mastery — lexical-sort member/set mismatch, the day's costliest mistake.** The
  per-set runner handed set *N* the *N*-th archive member in plain lexical sort order, and
  `blob-100.txt`..`blob-103.txt` sort ahead of `blob-11.txt`, so sets 11–103 were handed the wrong
  blob's slug (sets 1–10 were unaffected). This produced a long run of `incorrect`/bisection submit
  calls — well over 100 calls visible in `lane-warmups-day-21.log` between the first failed 103-answer
  submission (15:13:34Z) and the eventual 103-answer `CORRECT` (15:56:40Z) — crashed one lane run
  with `PermissionError(13, 'Access is denied')`, and burned through the mode's submission cap
  (`cap_reached`, calls:0), which needed the player to authorize raising it from 100 to 111 before a
  fix (resolving members by the question's own `artifact` field, never by sort position) could be
  confirmed and submitted. The decoder itself was correct throughout; only the blob-to-set pairing
  was wrong.
- **The web lane was out of scope for Day 21's automated run by design**, not by a bug —
  `orchestrate.py`'s own log line lists `web-exploitation` under "not automated" because its
  worker/solver integration did not exist yet (added Day 22). All three web challenges were solved
  by hand on 2026-10-22/23, including a multi-session investigation on Reflect Upon This caused by
  testing a two-condition gate one variable at a time (see that page's "Dead ends and gotchas").
- **Later gap-closure resumes tried to push the three web leaves' Time Trial through the
  now-generic release-automation solver and failed three times.** `lane-web-exploitation-day-06`,
  `-07` and `-08.log` (2026-10-31) each show "solver failed: no usable candidate for artifact" on a
  Time Trial retry, with no `submit` call logged — nothing was consumed by these failures. The
  actual ranked CORRECT results for all three came later from separate, purpose-built scripts
  (`solve.py`, `master_web_solve.py`) run outside that generic pipeline, as described in each page.
- **Several non-web ranked reruns on 2026-10-31/2026-11-01 split an already-one-call submission
  into multiple small group calls** (Metadata Time Trial's GUID/id/order/hex groups, BRE Time
  Trial's two hash-order groups, Script Time Trial's attack/batch-control groups) even though the
  original 2026-10-21 submission for the same answers went through whole in one call. Every group's
  first variant was accepted every time, so no answer was ever actually in doubt — the files do not
  explain why the later runs chose to submit by group rather than reusing the single-call shape, and
  each affected page notes this without further diagnosis.

## Effort

Submission calls to first CORRECT, from the lane logs and `progress.md`:

| Lane / Challenge | Single Solve | Time Trial (original) | Mastery (original) |
|---|---|---|---|
| Warmups / Under Pressure | 1 | 1 | ~108 (bisection after the sort-order bug; 1 once fixed) |
| Metadata / Package Deal | 1 | 1 | 1 |
| Script / Actions Speak Louder | 1 | 1 | 1 |
| BRE / Shared Custody | 1 | 1 | 1 |
| Mobile / Debug Me If You Can | 1 | 1 | 1 |
| Web / Session Impression | team-scored | 1 | 1 |
| Web / Stored for Later | team-scored | 1 | 1 |
| Web / Reflect Upon This | 1 | 1 | 1 |

Ranked (bragging-rights-only) reruns on 2026-10-31/2026-11-01 are not counted here; several are
detailed in the individual pages and above.

## Links

See the Challenges table for each page. `challenges/day21/_run/` holds the supporting timeline and
lane logs cited above; Day 21 predates automatic `closeout.py`/Librarian runs (added Day 25) and the
supervisor (added Day 29), so no `closeout.md`, librarian notes or `supervisor-report.md` exist for
this day — this overview was assembled by hand from `status.json`, `orchestrator.log`, `watch.log`
and the per-challenge lane logs instead.
