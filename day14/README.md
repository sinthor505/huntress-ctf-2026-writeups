# Day 14 overview

**At a glance:** 5 challenges (one per lane, no web lane yet), 15 modes. By provenance label: 12
Independent, 3 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: decoy prompts over easy
techniques, plus one very expensive Mastery. Each prompt (a "map of intent" of shortcuts, Objective-C
memory, seeded nonces, MSBuild property flow) promised more than the questions asked. The real work
was a Shell Link field, a cumulative recap of earlier binaries, a declared Caesar asset, a Caesar-7
warmup, and a decoded descriptor. The one place the day cost real effort was Script Analysis Mastery,
where 34 submissions went mostly to a few Batch control fields.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Seven Steps](warmups_seven-steps.md) | Caesar shift of 7; Mastery chains 14 numbered codecs | Journal-recorded | Independent | Independent |
| Metadata Forensics | [The Shortcut Home](metadata-forensics_the-shortcut-home.md) | Shell Link `.lnk` target path: LinkInfo `LocalBasePath` checked against the IDList | Independent | Independent | Independent |
| Mobile Android | [Predictable Crypto](mobile-android_predictable-crypto.md) | Declared Caesar codec in `assets/huntress_puzzle.json`; "seeded nonce" crypto is a decoy | Independent | Independent | Independent |
| Binary Reverse Engineering | [Selective Memory](binary-reverse-engineering_selective-memory.md) | Cumulative recap (Days 1-14) with CRC-validated `HCFG` config records; Objective-C is 10 of 140 fields | Journal-recorded | Independent | Independent |
| Script Analysis | [Trust the Process](script-analysis_trust-the-process.md) | Inert descriptor reconstruction (971 fields); new MSBuild `.proj` container | Independent | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Journal-recorded (Warmups and BRE Single Solve).** The journal records our own submission as CORRECT
  (Warmups with a timestamp, BRE with a per-attempt log we wrote). No response file from the Game Master
  was saved for either, and `progress.md` has a "Solved" row for each.
- **Independent, with a saved response (Metadata, Mobile and Script Single Solve and Time Trial; Metadata and
  Mobile Mastery).** The leaf holds the submission response reading CORRECT (`submission-01.json`, or
  `attempt-01-response.json` for Script).
- **Independent, backed by ranked reruns only (Warmups TT and Mastery, BRE TT and Mastery, Script Mastery).**
  The original attempts are journal-recorded. The label rests on saved `ranked-N/result.json` files that
  read CORRECT. Script Mastery's saved responses show accepted batches and its attempt log ends at CORRECT
  on submission 34, but the CORRECT response itself was not kept; the ranked rerun is what backs it.
- **Rerun counts.** Metadata, Mobile, Warmups and BRE each have `ranked-1` (2026-10-26) and `ranked-2`
  (2026-10-28, a later pass made under the team) for both Time Trial and Mastery, all CORRECT in one call.
  Script Analysis has only `ranked-1` for each, for the reasons under "Tooling and process" below.
- All 15 first solves were played on 2026-10-20 (UTC timestamps in the journals run from about 23:05 for the
  first Warmups and Script Single Solves). `progress.md` dates every row 2026-10-20.

## What the day had in common

- **Prompts were flavor, not instructions.** Metadata asked to "identify the impossible destination" but
  every question asked for one shortcut target (we could not determine what the phrase meant). BRE's
  Objective-C prompt governed 10 of 140 questions. Mobile's crypto prompt described logic that is not in
  the code. Script's "trusted toolchain doing untrusted work" was one inert `Message` task. Only Warmups
  ("Caesar took a week off") meant exactly what it said.
- **Recap and descriptor structure over new technique.** BRE re-asked Day 1-13 binaries through one bundle
  and Script was the usual descriptor bundle with four new `.proj` files out of 111 members. The tools
  built for earlier days did most of the work.
- **Short windows, so scripted solves.** Time Trial windows were 30 s (Warmups), 50 s (Metadata, Mobile,
  Script) and 80 s (BRE); Mastery windows were 85 s (Warmups), 80 s (Metadata, Script's first two-set
  assignment), 110 s (Mobile) and 170 s (Script five-set retries); the ranked BRE Mastery window was 320 s.
  Each lane was one fetch, solve, submit pass.
- **Time Trial and Mastery changed fixtures more than mechanism.** The first deliveries were tiny (Metadata
  1 and 2 shortcuts, Mobile 1 and 3 APKs), but the later ranked deliveries were larger: Metadata 19 and 95
  questions, Mobile 5 sets, BRE 5 sets (700 fields). Solvers had to read each set's own members and ids.
- **Mastery bundles arrive nested or in several parts.** Metadata Mastery was an outer ZIP of nested ZIPs;
  Mobile Mastery delivered several APKs fetched one by one; BRE and Script Mastery were multi-set bundles.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Script | One-item `FOR /L` Batch wrappers: the question text said `for_once`, the validator accepted `call_subroutine` (per-ID overrides; the answer varies by fixture). Bash and Node.js staged-chain `attack` was `T1105` in every mode this day. The `.proj` marker shares a line with its `-->` terminator, so a decoder that works for every other container fails on exactly the new one. |
| BRE | `hcfg_offset` points at the first Base64 character for Base64-carried records, not at the decoded record. Mastery set 2 delivered PE variants of Days 2, 4 and 10 where other sets had ELF, so classify by magic bytes. Literal `HCFG` strings in some files are decoys; only a CRC-valid record counts. Enum `format` fields are exact (`elf`), pattern fields accept equivalent spellings. |
| Mobile | The prompt describes a different challenge than the artifact; the answer is the decoded asset (`ANS-` plus 16 hex). Mastery deliveries beyond the first need the `delivery=N` fetch. |
| Metadata | Description and working directory are not the target; read LinkInfo and cross-check the IDList. Ranked fixtures turn this into one field of a 19-field bundle. |
| Warmups | The record number selects the codec and Day N has N records (14 here); the slug is what follows the last colon. |
| All multi-answer calls | `incorrect` on a batch carries no per-field feedback, so isolating one wrong field costs submissions (Script Mastery). A transient HTTP 503 (BRE Mastery) is not a verdict. |

CLAUDE.md carries the lasting versions: the Days 13-15 Batch-control lessons (classify by structure, one-item
`FOR /L ... CALL` handled per fixture) and the Bash/Node.js `T1105` note for Days 11-22. The ranked-pass
lessons for Days 11-15, including the cap-blocked Day 14 Mastery, are in `tools/ranked/PROCESS.md`.

## Mistakes in our own tooling or process

Collected here from the challenge pages, with the evidence for each.

- **Trust the Process (Script).**
  - The shared submission ledger lost this challenge's counts: at the final audit it showed one entry
    for Mastery against 34 recorded in the leaf's attempt log, and no key at all for any Day 13-15 id.
    The audit note in the Mastery journal blames a non-atomic read-modify-write in the client under
    concurrent runs (a likely cause, not a proven one). The per-leaf attempt logs were treated as authoritative and the ledger
    was rebuilt from them (its Day 14 entries carry a "reconciled 2026-10-20" tag; Mastery now reads 35,
    i.e. 34 plus the ranked rerun). CLAUDE.md records that the client's ledger writes are now lock-protected
    and atomic.
  - The first Mastery assignment (two sets, 1,942 fields, 80 s) was answered with the whole map in one call
    and came back `incorrect` with no per-field feedback. The later work partitioned by family and by Batch
    control signature, which is what a first-submission plan should have been.
  - Windows expired during diagnosis: a JScript request on the original assignment went out after its
    deadline (`expired`), and a retry's set-3 request also expired before evaluation. Acceptances on one
    retry assignment never carried to the next retry's different fixtures, so some calls were spent
    re-proving groups.
- **Seven Steps (Warmups).** The first Mastery submission arrived after its 85 s deadline and returned
  `expired`; the journal does not say why. A `retry` issued a new variant and the same mapper went CORRECT in
  one call. The Single Solve and Time Trial have no saved response file, only journal lines.
- **Selective Memory (BRE).** The first Mastery submission (420 fields) got HTTP 503 `admission_unavailable`.
  The attempt and deadline were still active (confirmed with `status`) and the identical map was resubmitted
  once. It cost a call but no answer. This was the service, not our error; we list it because it is a call
  spent without a verdict.
- **Metadata and Mobile (bookkeeping).** Their journals say the `submission-01.json` files hold the full
  Game Master response, but that payload also carries a `solved_by` block and, for Time Trial and Mastery,
  start-style fields (`unique_attempts_remaining` and similar). We read it as the submission response
  because the journals say so and it reads CORRECT; we could not independently distinguish it from a status
  read-out, so those rows rest on the journals as well as the file.
- **The Shortcut Home (Metadata).** The Single Solve solve relied on a private parser attribute
  (`_link_info`) because no Shell Link note existed yet. A stdlib reader was added to the library afterwards.

## Tooling and process changes around this day

The repository history begins on 2026-10-22, after the first solves (2026-10-20), so the original solving has
no commits of its own. What the history and files show:

- `8d6ce9d0` (2026-10-22): first commit of the Day 14 leaves (message "Add Day 14 challenge work").
- `c9906b50` (2026-10-25): the Days 11-15 ranked pass. It added the per-challenge solvers under
  `tools/ranked/solvers/` for all five Day 14 challenges, their cases, the `ranked-1` results, and a
  reconciled ledger. The Metadata solver wraps the library Shell Link reader with an independent LinkInfo
  reader; the Mobile solver pairs each question to its APK by package, hash and delivery name and reads the
  codec from each asset. In that pass Day 14 script Mastery was **blocked by the 20-submission ledger cap before
  anything was issued** (ledger count 34, per `tools/ranked/PROCESS.md`).
- Cap raise, 2026-10-26 (per `ranked-1/result.json` deadline and `PROCESS.md`): with the player's explicit
  OK for `--cap 40`, Script Mastery was rerun once and went CORRECT in one call (4,855 answers; `PROCESS.md`
  records scoreboard rank 6 and 2,444 ms, while the leaf's own `t_total` is 4.4 s). This is why the ledger
  history for Mastery reads 35. The original 34 submissions had been made under the player's earlier
  authorization described in the Mastery journal: three phases of up to 10, then extra 10-request blocks
  "while each block made measurable progress".
- `0fdd8957` (2026-10-28): team-attributed reruns of Days 13-15 (19 of 20 modes). For Day 14 it produced
  `ranked-2` for Warmups, Metadata, Mobile and BRE, all CORRECT. Day 14 Script Time Trial returned
  `legacy_attempt_pinned`, and Script Mastery (ledger 35) was again over the cap, so neither was rerun.
- Library and notes touched by this day (named in the pages): `WARMUP-CAESAR-SHIFT-001` (signed-shift Caesar),
  `META-LNK-TARGET-PATH-B103` (Shell Link target), the Android asset-codec `caesar` entry, and the descriptor
  decoder's comment-terminator handling (`-->` for HTML/MSBuild). Batch controls later got a structural
  classifier (`batch_control_signature.py`), described in CLAUDE.md under "Batch controls are structural".
- The `submission-ledger.json` rebuild and the move to lock-protected writes are described in the Script
  Mastery journal and in CLAUDE.md; the commits for those client changes are not in this day's history.

## Effort

Submission calls on the original attempts, from the journals and attempt logs (Day 14 has no
`submission-attempts.md`):

| Lane | Single Solve | Time Trial | Mastery | Ranked reruns, all CORRECT |
|---|---|---|---|---|
| Warmups | 1 | 1 | 2 (first expired, then CORRECT) | 2 TT, 2 Mastery |
| Metadata | 1 | 1 | 1 | 2 TT, 2 Mastery |
| Mobile | 1 | 1 | 1 | 2 TT, 2 Mastery |
| BRE | 1 | 1 | 2 (503, then CORRECT) | 2 TT, 2 Mastery |
| Script | 1 | 1 | 34 (plus 1 ranked call under the raised cap) | 1 TT, 1 Mastery |

Ranked-rerun total times per pass (`t_total` in `result.json`): Warmups 2.0-2.6 s, Metadata 2.8-5.4 s,
Mobile 1.3-3.2 s, Script 2.2 s (TT) and 4.4 s (Mastery), BRE 9.3-9.7 s (TT) and 15.5-17.7 s (Mastery), both
dominated by the large archive download and the solve. These reruns were bragging-rights attempts on the speed
scoreboard, not the prize ranking. The journals' timestamps put the first Single Solve and Time Trial
submissions of Warmups, Metadata, Mobile and Script between about 23:05 and 23:14 UTC on 2026-10-20 (some
lanes ran in parallel). Script Mastery's retries started at 23:15 and 23:18, then an untimed retry from 23:21;
the files do not give an end time for the 34th submission.
