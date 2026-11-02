# Day 28 overview

**At a glance:** 7 challenges across 6 lanes (web exploitation shipped two), 21 graded modes. All
seven Single Solves had already been scored by a teammate before our release automation reached
them (`already_solved` on our own submission); all fourteen Time Trial and Mastery modes are our
own graded CORRECT. Four of the five non-web lanes were pattern-matches to a mechanism already in
the library (cumulative recap bundles, declared script descriptors, a new codec variant), so the
day's real cost was in the automation around the challenges rather than in the challenges
themselves: a release-pipeline bug that this day's web lane exposed and got fixed the same day, a
result file on Fail Open Friday that disagreed with itself, and ranked Time Trial/Mastery attempts
for both web challenges that did not actually land until three to four days later.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Back in Sixteen](warmups_back-in-sixteen.md) | Layered text codec chain (reverse, then hex); numbered Mastery records | Team | Independent | Independent |
| Metadata Forensics | [Where It All Began](metadata-forensics_where-it-all-began.md) | Raw disk image GPT/MBR partition-table parsing (new `disk` family) | Team | Independent | Independent |
| Mobile Android | [Protocol Reconstruction](mobile-android_protocol-reconstruction.md) | New framed-hex `packet` codec with an XOR checksum; v2 builds run through the Dalvik mini VM | Team | Independent | Independent |
| Binary RE | [Family Resemblance](binary-reverse-engineering_family-resemblance.md) | Cumulative recap bundle; role-named members (`family-a`/`family-b`/`false-friend`); HCFG config record | Team | Independent | Independent |
| Script Analysis | [Out of Order](script-analysis_out-of-order.md) | Terraform HCL added to the inert-descriptor bank; declared descriptor fields win | Team | Independent | Independent |
| Web Exploitation | [Fail Open Friday](web-exploitation_fail-open-friday.md) | Fail-open authorization on an undecodable remote policy decision | Team | Independent | Independent |
| Web Exploitation | [If a Breach Falls in a Log](web-exploitation_if-a-breach-falls-in-a-log.md) | Audit-log event-type gap (sensitive action logged under the wrong, lower-severity event) | Team | Independent | Independent |

"Team" means a teammate's attempt had already scored the Single Solve; `team_solved.json` sits in
every one of the seven Single Solve leaves and our own submission came back `already_solved`.
"Independent" means our own leaf holds a `result.json`/`accepted_plan.json` (or, for one mode, a
`manual_solve.json`) reading CORRECT. Provenance totals: 7 Team, 14 Independent, 0 unsolved.

## How solid the "Team" label actually is, mode by mode

All seven pages state the Single Solve label correctly — every leaf has `team_solved.json`, and no
page claims more than what its own files show. But the *diligence behind* that label is not even
across the day. Per `CLAUDE.md`'s rule ("Record `Team solve (not ours)` for SS only after
confirming this live for that exact ID, not by inference"), only **If a Breach Falls in a Log**
got the full treatment: its journal records a player session on 2026-10-28 that ran
`gm.py retry` and `gm.py practice` against that exact ID, got `405 operation_not_allowed` from
both with nothing consumed, and `progress.md` carries an explicit row for it. The other six
Single Solves — including its own same-day web sibling, Fail Open Friday — have **no
`progress.md` row at all** for Single Solve and no recorded `retry`/`practice` probe anywhere in
their journals; the "Team" conclusion there rests only on the automated workers seeing
`start_response.json` with no `targets[]` and a pre-existing `result: CORRECT`, which is correct
reasoning but not the live confirmation the rule calls for. This is a diligence gap, not a wrong
label — if a fresh Single Solve attempt is ever shown to be issuable for one of those six IDs, it
would be news, but nothing in the current evidence suggests that.

## Timeline (UTC, 2026-10-28 unless noted)

| Time | Event |
|---|---|
| 19:38:43 | Release orchestrator starts: 7 challenges across 6 lanes, cap 30 (70 once escalated) |
| 19:40:14–19:40:16 | Both web Single Solves recon as team-already-correct, no `targets[]` |
| 19:40:54–19:46:48 | Warmups, Metadata, Mobile, Script and Binary RE lanes each finish Single Solve (team) plus regular and ranked Time Trial/Mastery, all CORRECT |
| 19:42:43–19:47:50 | If a Breach Falls in a Log Single Solve stalls 4 rounds; round 2 escalates to Opus (cap 30→70); every round correctly finds no live target and refuses. The automated run for this challenge stops here without ever attempting Time Trial/Mastery |
| 19:43:05–19:44:11 | Fail Open Friday Single Solve stalls the same way, escalates to Opus at round 2 |
| 19:50:23–19:57:31 | Fail Open Friday Time Trial and Mastery: fresh `retry`s issued despite the team-solved `start`; Time Trial goes CORRECT, Mastery's automated pass resolves only 1 of 5 targets (`result: incorrect`, 6 calls) |
| 19:57:31 | Orchestrator reports "all lanes finished"; Breach's Time Trial/Mastery had not been attempted at all and Fail Open Friday's Mastery was still incomplete |
| 2026-10-28 (later, player session) | Rules-update review catches the stall; retry/practice confirmed refused for Breach's Single Solve (405/405); Breach's Time Trial and Mastery solved independently via `retry`; Fail Open Friday's remaining 4 Mastery targets recovered by hand and a 5th (stored-XSS) target completed after a follow-up pass — both web challenges' regular Time Trial/Mastery close out CORRECT this same day |
| 2026-10-31 23:09 | A ranked-rerun pass attempts ranked Time Trial/Mastery for both web challenges; all four calls fail with "no usable candidate for artifact" |
| 2026-11-01 04:30–06:45 | Further ranked-rerun passes land ranked Time Trial CORRECT for both web challenges; ranked Mastery for both needed additional manual solver fixes (below) before landing CORRECT, per `progress.md` |

Non-web lanes finished end-to-end (Single Solve team-scored, both Time Trial and Mastery regular
*and* ranked attempts CORRECT) inside the same ~8-minute window per lane; the web lane took the
rest of the day for its regular solves and ran into 2026-11-01 for its ranked ones.

## Automation mistake: "If a Breach Falls in a Log" stalled the whole challenge

This challenge is the one `CLAUDE.md` names as the trigger for the orchestrator fix dated
2026-10-28. Four Single Solve rounds (`challenges/day28/_run/lane-web-exploitation-day-25-if-a-breach-falls-in-a-log.log`)
each correctly found `start_response.json` carried no `targets[]`, just the team's pre-existing
`result: CORRECT`, and correctly refused to fabricate an answer — including after an Opus
escalation with the cap raised to 70. The release pipeline at the time treated that stall as a
reason to mark the whole challenge stuck and never attempt Time Trial or Mastery automatically,
even though both of those modes support an independent `retry` regardless of the team's prior
solve. A manual player session caught this the same day, confirmed the Single Solve limitation is
real (`retry`/`practice` both `405`), and then solved Time Trial and Mastery for real via `retry`.
`tools/release/orchestrate.py` was fixed so a lane whose first `start` already reads the team's
`CORRECT` status skips straight to team-solved bookkeeping and still proceeds to Time Trial and
Mastery, rather than halting.

## Automation mistake: a result file that disagreed with itself (Fail Open Friday Mastery)

The regular (untimed) Mastery attempt and its final outcome share one `attempt_id`
(`27f4ee5a-b101-4d79-bb0f-b6c04c1050c2`) but two different saved files disagree:
`mastery/regular/result.json` reads `"result": "incorrect"` with 0 accepted and all 5 proofs
listed as `bad` (6 calls), while the leaf's top-level `mastery/result.json` for that same
`attempt_id` reads `"result": "CORRECT"` with all 5 `accepted_ids`. Both are accurate snapshots of
the same attempt at different points: the automated worker pass only solved Target 5 (the
challenge's own fail-open class) and left the submission `incorrect`; a manual pass then recovered
Targets 1, 3 and 4 against known library classes, and a stored-XSS Target 2 after a follow-up,
closing the attempt CORRECT. The page's "Independent" label is correct — it is backed by the
terminal file and the matching `progress.md` row — but a reader who opens only
`mastery/regular/result.json` would see a false "incorrect," not the actual outcome.

## Ranked Time Trial/Mastery landed late for both web challenges

Both web challenges' *regular* (untimed) Time Trial and Mastery attempts were solved and graded
CORRECT the same day as the release (2026-10-28). Their *ranked* attempts did not: a ranked-rerun
pass on 2026-10-31 failed outright on all four ranked calls with "no usable candidate for
artifact," and `challenges/day28/_run/status.json` still records `"ranked_mastery_1": "solver_failed
Nones"` for both challenges even after they eventually went CORRECT. Fixing it took solver work
logged in `progress.md`: Fail Open Friday's `try_fail_open` had to be changed to read
`policy_format_route` live instead of guessing a parameter name, and its ranked Mastery's first
real attempt missed a SQL-injection target before a clean retry got all five. If a Breach Falls in
a Log's ranked Mastery bundle mixed a different fifth vulnerability class on nearly every retry
(an authorization-matrix gap, a password-reset IDOR, a forwarded-header spoof, a WebSocket-upgrade
bypass, a TOCTOU credit race) — five consecutive retries each resolved 4 of 5 targets with a
different class missing each time, each new class ported into the shared solver as it was found,
until a sixth retry went 5/5. That burned the ranked-attempt pool from 5 remaining down to 1. Since
ranked attempts are bragging-rights only (per `CLAUDE.md`, the prize ranking only needs one
regular CORRECT per mode), none of this affected either challenge's actual provenance — but it is
why both web challenges' entries in `status.json` look unresolved days after the leaves themselves
were already closed out.

## What the day had in common

- **Titles and prompts were flavor more often than not.** "Cluster them by behavior" (Family
  Resemblance) produced no behavioral question; "assemble the timeline... find the change that
  happened out of order" (Out of Order) produced no timeline question; "prove which image was
  rebuilt from an incomplete source" (Where It All Began) was never asked either. Warmups was the
  exception — its riddle named the actual codec.
- **Declared descriptor fields kept winning.** Out of Order's Terraform members were the fourth
  new scripting language in a row (after Elixir, Erlang and Dockerfile) where the specimen's own
  `role`/`attack`/`steps` fields were accepted over the printed graph rule.
- **Cumulative banks and recap blocks continued.** Family Resemblance re-asked 27 days of binary
  recap fields plus one new 14-field block; Where It All Began's Time Trial/Mastery re-asked every
  earlier metadata family alongside the one new disk field.
- **Web Mastery bundles kept mixing vulnerability classes.** Fail Open Friday's and If a Breach
  Falls in a Log's five-target Mastery bundles shared an identical four-class filler set
  (reflected-XSS conjunction gate, stored-XSS via a moderator bot, session fixation, a
  state-changing GET) and differed only in their fifth, challenge-specific target — the same
  pattern seen on multiple earlier web-lane days.

### Validator and format traps seen this day

| Lane | Trap |
|---|---|
| Binary RE | `d13_analysis_crystal_symbol_count` rejected `.symtab`+`.dynsym` (1255) and accepted `.symtab`-only (771); later days re-confirmed `.symtab`-only as the default. |
| Script | The Single Solve's ATT&CK question text was lenient (parent technique or `none` accepted); the Time Trial's text for the same field family was strict (exact declared technique only). The declared value satisfied both. |
| Web | Stripping the required `remote:<id>:` prefix instead of appending the undecodable suffix after it hits `remote_policy_required` and never reaches the fail-open path (Fail Open Friday); the accepted answer is the whole `proof_<hex>` string, not the bare hex. |
| Web | Swapping which of two fields is the "real" event and which is the "logged" event in an audit-log mismatch report returns a generic rejection with no directional hint (If a Breach Falls in a Log). |
| Metadata | GPT entry start LBA is the u64 at offset +32 of each entry; MBR start is the u32 at +8; a second, independent reader failed on the newer hex-tagged member naming (`roster-<hex>.csv`), not on the disk parsing itself. |
| Mobile | A checksummed frame (`HA` magic + length + XOR checksum) is self-validating: a wrong framing guess simply fails the checksum, so a pass is a correct decode. |

## Tooling and process changes around this day

- **2026-10-28, commit `2360033e`** ("Add Day 27 challenge work: independent TT/Mastery solves,
  local-evidence rule, Librarian pass") landed the "local evidence only" rule strengthening that
  the Day 28 web Single Solve confirmations (the retry/practice 405 checks) rely on.
- **2026-10-28, commit `4201ad3c`** ("Complete Day 28 web Mastery solves (breach + Fail Open
  Friday)") is the manual completion described above: Breach's Time Trial/Mastery via `retry`,
  and Fail Open Friday's remaining Mastery targets recovered by hand.
- **`tools/release/orchestrate.py`** was fixed the same day so a lane whose first `start` already
  reads the team's `CORRECT` status skips straight to team-solved bookkeeping instead of treating
  a stalled Single Solve as a reason to skip Time Trial/Mastery entirely (see above).
- New library entries from this day: `library/web-exploitation/audit-log-missing-event-report.md`
  (WEB-AUDITLOG-001), `library/metadata-forensics/disk-partition-lba/README.md`
  (META-DISK-PARTITION-LBA-001), and a `decode_packet` addition to
  `library/mobile-android/asset-codecs/asset_codecs.py`. Fail Open Friday's fail-open-authorization
  class was flagged as a gap candidate (`fail-open-authorization-decoder.md`) rather than written
  up, since the mechanism was adapted from neighboring entries and only confirmed once a live
  target existed on the Time Trial fix round.

## Effort

Submission calls per mode, from `progress.md` and the leaves' own `result.json` where stated
("team" = nothing graded for us):

| Challenge | Single Solve | Time Trial (regular) | Mastery (regular) |
|---|---|---|---|
| Back in Sixteen | team | 2 | 9 (7 of them a systematic rejection before a codec-table fix) |
| Where It All Began | team | 3 | 2 |
| Protocol Reconstruction | team | 2 | 2 |
| Family Resemblance | team | 5 (4 fields in bisection, 1 symbol-count variant) | 2 |
| Out of Order | team | 5 | 2 |
| Fail Open Friday | team | 1 | 6 (automated pass resolved 1 of 5 targets; remainder recovered by hand) |
| If a Breach Falls in a Log | team (confirmed via `retry`/`practice` 405) | not separately logged | 2 (4 targets in one call, 5th after a follow-up) |

Ranked attempts for the five non-web modes all landed CORRECT the same day in one call each
(3.89 s–20.31 s locally, per `progress.md`). Both web challenges' ranked Time Trial and Mastery
needed the multi-day recovery described above before landing CORRECT on 2026-11-01.
