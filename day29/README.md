# Day 29 overview

Day 29 released at 09:00 ET (13:00 UTC) on 2026-10-29: seven challenges across six lanes, 21 graded
modes (Single Solve, Time Trial, Mastery each). Two lanes' Single Solves were scored by a teammate
within minutes of unlock; everything else we solved ourselves.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Rotten Base](warmups_rotten-base.md) | ROT13 over base64; new Mastery record codec | Team | Ours | Ours |
| Metadata Forensics | [Trust Issues](metadata-forensics_trust-issues.md) | X.509 SHA-256 fingerprint; GPT partition LBAs | Ours | Ours | Ours |
| Script Analysis | [Make It Make Sense](script-analysis_make-it-make-sense.md) | Makefile specimens; declared descriptor wins | Team | Ours | Ours |
| Mobile Android | [White Box Key Recovery](mobile-android_white-box-key-recovery.md) | `table` codec from known prefix; v2 builds via Dalvik VM | Ours | Ours | Ours |
| Binary RE | [Fraternal Twins](binary-reverse-engineering_fraternal-twins.md) | O3/O0 ELF pair; cumulative recap bank | Ours | Ours | Ours |
| Web Exploitation | [Trust the Build](web-exploitation_trust-the-build.md) | Signature scope gap in a signed manifest | Ours | Ours | Ours |
| Web Exploitation | [Internal Affairs](web-exploitation_internal-affairs.md) | SSRF into an internal IDOR | Ours | Ours | Ours |

"Team" means a teammate's graded solve scored the Single Solve first, so our own submission came back
`already_solved` and nothing was graded for us (a fresh Single Solve attempt is not issuable once a
teammate has solved it). "Ours" means our own graded CORRECT. Details are on each challenge page.

## Timeline (UTC, 2026-10-29)

| Time | Event |
|---|---|
| 12:50 | Release watcher and supervisor start; preflight against the Day 28 list |
| 12:55 | Polling window opens (T-300 s) |
| 13:00 | Day 29 unlocks; recon and solver sessions start for all seven challenges |
| ~13:02-13:03 | Mobile Android, Script Analysis and Warmups lanes complete |
| ~13:06-13:07 | Metadata Forensics and Binary RE lanes complete |
| ~13:18 | Internal Affairs (web) completes |
| ~13:23 | Trust the Build (web) completes; the last lane |
| 13:23-14:19 | Librarian pass: one library note per challenge, library checker clean |
| 14:19 | Supervisor: 7 of 7 lanes ok, no lane needed the player |

Each lane's completion covers Single Solve, a regular Time Trial and Mastery rehearsal, then the
ranked Time Trial and Mastery attempts. All seven lanes were finished 23 minutes after unlock.

## Submission effort

Submission calls per challenge and mode, from the local ledger (regular rehearsal plus ranked
attempt combined). Single Solve is "team" where nothing was graded for us.

| Challenge | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Rotten Base | team | 2 | 3 |
| Trust Issues | 1 | 3 | 2 |
| Make It Make Sense | team | 5 | 2 |
| White Box Key Recovery | 1 | 2 | 2 |
| Fraternal Twins | 2 | 12 | 2 |
| Trust the Build | 4 | 2 | 12 |
| Internal Affairs | 1 | 2 | 10 |

Ranked-attempt times as measured locally (end to end, solver plus download plus submit):

| Challenge | Time Trial | Mastery |
|---|---|---|
| Rotten Base | 4.41 s | 13.2 s |
| Trust Issues | 3.73 s | 7.23 s |
| Make It Make Sense | 5.18 s | 10.94 s |
| White Box Key Recovery | 3.08 s | 8.77 s |
| Fraternal Twins | 6.63 s | 14.11 s |
| Trust the Build | 11.18 s | 23.7 s |
| Internal Affairs | 14.97 s | 24.15 s |

## What the day taught

- **A new day usually adds one new thing; the rest re-asks old questions.** Fraternal Twins had 8 new
  answer IDs out of 406. Trust Issues' Time Trial and Mastery re-asked the previous day's new field.
  Keeping a cumulative parser library, and promoting each new reader into it the day it first
  appears, paid off.
- **Single Solve and Mastery can disagree on what is already known.** Rotten Base passed Single Solve
  and Time Trial with the existing decoder, but its Mastery failed 143 of 143 until record 29 was
  added to the per-record codec table. A solver that special-cases the single blob misses this.
- **Declared metadata beats derived reasoning.** For Make It Make Sense, the validator accepted each
  specimen's own `role`/`attack` descriptor for the fourth new-language day in a row (Elixir,
  Erlang, Dockerfile, Makefile).
- **Mobile builds split by schema.** White Box Key Recovery's Single Solve was a v1 `table` codec;
  its Time Trial and Mastery were v2 APKs whose answers come from running the app's own code in the
  Dalvik interpreter. The solver must dispatch on the schema, not assume one codec.
- **Web Mastery bundles mix vulnerability classes.** Both web challenges shipped a Mastery bundle of
  five targets, only one of which matched the prompt (backup disclosure, an activity-feed IDOR,
  paren-wrapped SQL injection and PHP LFI made up the rest). Recon every target; do not assume order.

## Mistakes in our own automation

These cost submissions but are not challenge lessons.

- **Answer-group payload shape (Trust the Build).** The first solver keyed each answer variant by the
  literal placeholder `id` from the spec instead of the real answer ID (`proof-1`), so a correct
  proof was rejected as `incorrect` with `unaccepted_ids: ["id", "proof-1"]`.
- **Single-target assumption in Mastery (Trust the Build, Internal Affairs).** Early solvers ran one
  exploit against the first target and copied its proof to every answer. Both were rewritten as a
  per-question target resolver with a fingerprint-and-dispatch step, which fixed Mastery for both.
- **Toolkit gap (Trust Issues).** The first Time Trial rehearsal resolved 33 of 34 fields because
  the metadata toolkit had no disk reader; one was added before the next attempt.
- **Ambiguous symbol-count field (Fraternal Twins).** The first full Time Trial submission came back
  `incorrect` with no per-field feedback, so the rehearsal bisected it in chunks (11 calls) and found
  one field, `d13_analysis_crystal_symbol_count`, whose first reading (symtab plus dynsym) was wrong;
  the symtab-only reading was accepted. The ranked attempt then passed in a single call.

## Tooling and process changes around this day

Changes made to the playing workspace on 2026-10-29, from the commit history:

- **Release-day supervisor added.** A watchdog that starts with the release watcher, recovers a dead
  orchestrator, resumes orphaned lanes, and hands any lane lacking our own CORRECT to a stronger
  model whose only submission path has the attempt cap baked in. On Day 29 it ran as designed and
  intervened zero times.
- **Team-solved handling reworked.** A `manual_solve.json` marker for hand-solved leaves and
  de-duplicated team-solved documentation, so a teammate's solve can never be mistaken for ours in
  the local ledger. Both Day 29 team-solved Single Solves are recorded this way.
- **Local-evidence-only rule applied.** Progress rows are written from our own graded results, never
  from the Game Master's status, which reports the team's attempt.
- **Solve-times tab** added to the local leaderboard viewer, showing gaps to place 50 and first.
- **Librarian pass.** Each challenge produced a post-solve note and a proposed update to the
  playbook. Notable ones: the new signature-scope-gap technique (`WEB-SUPPLYCHAIN-001`), adding a
  new Mastery record number to the warmups codec table before submitting, promoting a new reader
  into the metadata toolkit on the day it first appears, and not assuming a fixed target count in
  web Mastery bundles.
