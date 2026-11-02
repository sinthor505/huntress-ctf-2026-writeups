# Day 13 overview

**At a glance:** 5 challenges (one per lane, no web lane in this release), 15 modes. By provenance label: 13
Independent, 2 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: four of the five lanes were easy
to read and the trouble came from scale and spelling. Script Analysis was the exception: a 913-field bundle
whose validator disagreed with its own prompt on two Batch fields, which cost 48 Single Solve calls and 91
Time Trial calls before Mastery went through in one.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Opposite Day](warmups_opposite-day.md) | Atbash decode; Mastery chains 13 codecs selected by record number | Independent | Independent | Independent |
| Metadata Forensics | [Product Placement](metadata-forensics_product-placement.md) | MSI `Property` table `ProductCode`, braces kept | Journal-recorded | Independent | Independent |
| Mobile Android | [Keystore Misuse](mobile-android_keystore-misuse.md) | Declared `base64` codec in `assets/huntress_puzzle.json`; Keystore theme is flavor | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Crystal Clear](binary-reverse-engineering_crystal-clear.md) | Cumulative d01-d13 recap; XOR+CRC32 HCFG record inside a Base64 run | Independent | Independent | Independent |
| Script Analysis | [Zip Code](script-analysis_zip-code.md) | Static decoding of inert descriptors in 105 scripts (913 fields); Batch control override | Independent | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Journal-recorded (Metadata and Mobile Single Solves).** The journal records our own CORRECT (Metadata on
  its second submission, Mobile on its first), but no response file was saved. `progress.md` has a "Solved"
  row for each.
- **Independent, backed by a saved response.** Warmups (all three modes, saved `submission-attempt` or
  retry response files), BRE (saved `attempt-NNN-response.json` files), and Script Analysis (the call-48,
  call-91 and call-1 responses, one per mode).
- **Independent, backed by ranked reruns only.** Metadata and Mobile Time Trial and Mastery: the original
  attempts (progress rows dated 2026-10-20) are journal-recorded, and the label rests on the `ranked-N/result.json`
  files, all CORRECT in one call. Ranked reruns were bragging-rights attempts, not the scoring path.
- **Release.** The day was played on 2026-10-20 (progress rows; start responses run from 22:58 UTC that day to
  04:02 UTC on 2026-10-21 for the Script Mastery). There is no `_run/` folder for Day 13, so there is no
  librarian note, lane log or timeline to draw on. The Day 13 Web challenge named `web-exploitation-day-13-posted-too-much`
  is a curriculum-day ID released later; it is not part of this folder.

## What the day had in common

- **Titles and prompts were flavor.** "Keystore Misuse" has no Keystore code in any build; "Crystal Clear"
  is a recap of every earlier binary day with only a loosely related Crystal-built ELF; "Product Placement"
  talks about a collision across installers but asks for one field. The Warmups clue ("A met Z") was the one
  prompt that was literal.
- **Cumulative or declared structure did most of the work.** Warmups Mastery chained 13 text codecs by
  record number, BRE reused every earlier day's parser, Mobile's asset named its own codec. The new piece
  each lane had to add was small.
- **Scale jumped between modes.** Metadata Time Trial and Mastery in the ranked reruns were 18 and 90 field
  recaps against 1 and 2 fields originally; Mobile Mastery went from 3 sets to 5; BRE Mastery from 3 sets
  (390 fields) to 5 (650); Script Mastery was 5 x 913 = 4,565 fields. Solvers that read the manifest and the live
  answer IDs survived this; per-delivery scripts would not have.
- **Windows were short on the original attempts.** Warmups TT 30 s, Metadata and Mobile TT 50 s,
  Warmups and Metadata Mastery 80 s, BRE TT 80 s, Mobile Mastery 110 s, BRE Mastery 200 s (start-response
  deadlines minus start times). Script Time Trial was 50 s and its Mastery ran with no clock. Warmups,
  Metadata, Mobile and BRE were each one scripted fetch, solve, submit pass.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | The prompt advertises a brace-free canonical GUID; the validator wanted the stored `{...}` upper-case form. The brace-free try was `incorrect`. |
| BRE | Every answer value must be a JSON string. Native arrays and integers returned HTTP 400 `invalid_answers`. `hcfg_offset` for a Base64 carrier is the offset of the Base64 run, not of `HCFG` in the decoded bytes. Mastery set 2 delivered d02/d04/d10 as PE32+ while sets 1 and 3 were ELF, and the first 390-field call was `incorrect` until the d10 labels were fixed; the exact label that failed is not recorded. |
| Script | Batch one-item `FOR /L ... DO CALL :LABEL` wrappers: the prompt and source said `for_once`, the validator accepted `call_subroutine` (benign and relay in Single Solve, benign and chain in Time Trial). `incorrect` on a whole-map call carries no per-field feedback, so two wrong fields hid 911 good ones. |
| Warmups | Time Trial and Mastery must be submitted under `<id>/time_trial` or `<id>/mastery`; the bare ID returns `already_solved` and scores nothing. Atbash is not reversal (Mastery record 12 vs 13). |
| Mobile | None. The decoded value already matched `ANS-[0-9A-F]{16}`, `exact`. |

CLAUDE.md carries the lasting forms of these lessons: the Day 13 Zip Code bullet (Batch one-item `FOR`
controls were `call_subroutine` in every case; once a mapper is validator-confirmed, submit the remainder
in one call), the string-only answers rule, the `<id>/<mode>` rule, the MSI ProductCode braces note, and the
note that Mastery d02/d04/d10 sets mix PE and ELF. We could not date when each bullet was added: the file's
first commit is `6d3ea597` (2026-10-22), already after Day 13 was solved.

## Mistakes in our own tooling or process

Collected here from the challenge pages, with the evidence for each.

- **Zip Code (Script).**
  - Single Solve and Time Trial each began with a full-map submission that was `incorrect` and said nothing
    about which field was wrong. Two Batch controls were the cause, and a structural alternate
    (`call_subroutine`) queued from the start would have found them in a handful of calls rather than 48.
  - Time Trial's first two timed 50-second attempts were wasted: the first full map was rejected (consistent
    with the same `for_once` problem, though the journal does not state the cause) and the retry's submission
    returned `expired` at the deadline boundary. It finished on an untimed practice attempt after 91 logged
    calls on the mode (calls 1 and 2 were the two timed attempts). After 90 calls, a re-check showed the mapper
    byte-identical to the Single Solve one and the remaining 495 values went in as one call. The journal itself
    says calls 11-90 were avoidable; the Mastery single call is the model. The journal also records that calls
    11-20 were handed to a second agent.
  - **Cap bookkeeping.** The Time Trial journal logs the final call as "attempt 91 of 110" but also says "20
    authorized" and that no explicit player authorization for calls 21-91 is written there ("a documentation
    gap only"). `tools/ranked/PROCESS.md` separately records that the later ranked rerun of this mode (ledger
    91) ran with a cap of 101 after the player authorized 10 more submissions. We could not reconcile 110 vs
    101 from the files. The ledger now reads 48 / 92 / 2 for Script Single Solve / Time Trial / Mastery
    (92 = the 91 plus the ranked rerun).
- **Crystal Clear (BRE).**
  - Single Solve's first call was HTTP 400 `invalid_answers` (native JSON arrays and integers). Nothing was
    graded, but it was counted. The string-only rule is in CLAUDE.md as a Days 7 and 8 lesson, so this repeated a
    known mistake.
  - Mastery's first 390-field call was `incorrect` because set 2's d10 specimen was a PE32+ and the mapper
    had assumed ELF. The journals never recorded which exact label the validator rejected.
- **Opposite Day (Warmups).** Two early Time Trial submissions went to the bare `warmups-day-13` ID and came back
  `already_solved`; the first timed attempt also expired during recon. Both are our process, not the puzzle.
- **Product Placement (Metadata).** Attempt 1 sent the brace-free GUID that the prompt describes. This is
  mostly a validator trap, but a rule of "try the stored form first" would have saved the call.
- **Keystore Misuse (Mobile).** No mistakes found. The Single Solve download was a ZIP-named APK, which cost
  nothing.
- **Ranked reruns.** Script Mastery could not be re-issued on 2026-10-28 (`legacy_attempt_pinned`), and Script
  Time Trial (ledger 92) was over the cap, so Script has only its first-sweep ranked files.

## Tooling and process changes around this day

The repository history starts on 2026-10-22, two days after Day 13 was played (progress rows are dated
2026-10-20), so the original solving has no commits of its own. What the history does show:

- `5ed03473` (2026-10-22): first commit of the Day 13 work, i.e. the journals, mappers, the Script submission
  logs and diagnostic payloads, and the BRE and Warmups response files.
- `3bcba4e6` (2026-10-22): the technique library, including the Batch control classification notes, the MSI
  product-identity note and the BRE cumulative-recap-bundles note that the pages cite. These were written after
  the Day 13 solves, not used during them.
- `c9906b50` (2026-10-25): the Day 11-15 ranked pass, which added the per-challenge Day 13 solvers under
  `tools/ranked/solvers/` and the first `ranked-1` results for Day 13. Its message records that
  three script-analysis modes were blocked by the 20-submission cap before anything was issued (Day 13 Time Trial
  count 91, Day 14 Mastery 34, Day 15 Time Trial 25); `PROCESS.md` records that the player then authorized further
  submissions and Day 13 Script Time Trial went CORRECT in one call with `--cap 101`. Day 13 Script Mastery's
  ranked rerun is also from this pass.
- `0fdd8957` (2026-10-28): team-attributed Day 13-15 reruns. For Day 13 it added `ranked-2` for Metadata, Mobile
  and BRE (Time Trial and Mastery). Warmups has no `ranked-2` and we could not determine why from the files. The
  message says 19 of 20 attempted modes ran CORRECT and that Day 13 Script Mastery returned
  `legacy_attempt_pinned` and Day 13 Script Time Trial was over the cap, so neither was re-run.
- `38668151` (2026-11-01): the top-30 gap-closure sweep added a third Mobile pair (`ranked-3` for Time Trial
  and Mastery, both CORRECT in one call).

## Effort

Submission counts, from the journals, response files and the ledger (Script counts match the ledger; for the
other lanes they are from the journals and saved responses):

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Ranked reruns that went CORRECT |
|---|---|---|---|---|
| Warmups | 1 | 1 on the retried attempt (plus 2 mis-routed `already_solved`) | 1 | 1 TT, 1 Mastery |
| Metadata | 2 (first `incorrect`) | 1 | 1 | 2 TT, 2 Mastery |
| Mobile | 1 | 1 | 1 | 3 TT, 3 Mastery |
| BRE | 2 (first HTTP 400) | 1 | 2 (first `incorrect`) | 2 TT, 2 Mastery |
| Script | 48 | 91 (untimed practice attempt) | 1 (untimed regular attempt) | 1 TT, 1 Mastery |

Ranked-rerun total times (`t_total` in `result.json`): Warmups 1.6-2.2 s, Metadata 1.7-2.9 s, Mobile 1.5-4.3 s,
Script 2.1 s (TT) and 5.6 s (Mastery), BRE 9.9-18.3 s (dominated by the archive download and solve). The server-side
rank is not recorded in the leaves.
