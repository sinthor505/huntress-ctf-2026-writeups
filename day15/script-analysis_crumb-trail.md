# Crumb Trail

| | |
|---|---|
| Day / Lane | 15 / Script Analysis |
| Challenge ID | `script-analysis-day-15-credential-store-clues` |
| Technique | static descriptor reconstruction over a 116-specimen script bank; Batch control-flow classification by structure |
| Single Solve | Independent (CORRECT on submission 31) |
| Time Trial | Independent (CORRECT on submission 25 of an untimed retry; a later ranked rerun was CORRECT in 1 call) |
| Mastery | Independent (CORRECT on submission 14 of an untimed attempt; two later ranked reruns were CORRECT in 1 call each) |

## Summary

"Crumb Trail" is a script-analysis bundle of 116 synthetic specimens in about fifteen languages,
with 1,026 answer fields per set. The themed prompt (browser credential stores) turned out to be
flavor: every answer comes from a descriptor that each specimen carries in encoded form, and the
work is reconstructing those descriptors statically and mapping them to the question schema. A
full-map submission was rejected as a batch, and nearly all the effort went into isolating which
fields the validator disagreed with. They were nine to 45 Batch `*-control` classifications, where
the validator followed the specimen's structure rather than the prompt's wording or any per-ID
rule. All three modes ended CORRECT.

## The challenge

Prompt: "Some credential-access-shaped code keeps rummaging through the browser's cupboards, and
synthetic store artifacts are sitting there waiting. Connect each code path to the artifact it
targets, and find the reference that points at a store that was never there."

- **Delivered:** one AES-encrypted ZIP (`evidence.zip`, password `infected`) with 116 flat script
  members: PowerShell, JScript, VBScript, Batch/CMD, Python, Bash, Node.js, PHP, Lua, WMI MOF,
  TypeScript, AutoIt, AppleScript, MSBuild XML and Perl. Single Solve and Time Trial deliver one
  such archive (about 108 KB); Mastery delivers an outer ZIP of five of them (about 535 KB).
- **Questions:** 1,026 per set. Format mix in Single Solve: 609 `exact_text`, 160 `enum`, 113
  `pattern`, 71 `integer`, 71 `ordered_tokens`, 2 `mitre_attack_id`. By ID prefix they cluster into
  `polyglot-*` (396), `bash-*`, `python-*`, `host-*`, `jscript-*`, `batch-*`, `vbscript-*`,
  `nodejs-*` and a handful of PowerShell scenario fields (`resource-*`, `collection-*`, `impact-*`).
- **Modes:** Single Solve is untimed. The Time Trial's original window was 50 seconds. Mastery is
  5 sets x 1,026 = 5,130 IDs prefixed `setN.`.

## Solution

1. **Extract and census.** The archive opens with the shared AES-ZIP helper (stdlib `zipfile` cannot
   do WinZip AES). The 116 members were treated as data and never executed. A census confirmed
   there is no `Login Data`, `Cookies`, `Web Data` or `Local State` file in the archive: the
   credential-store names appear only as strings inside `collection-sample.ps1`'s descriptor. The
   prompt's "reference to a store that was never there" has no matching question that we could find;
   we noted it and moved on.
2. **Decode descriptors statically.** Each specimen hides a small JSON descriptor behind a
   language-specific encoding, for example Base64 JSON in PowerShell, or Base64 plus a byte-reversal
   routine in JScript. An offline parser, carried over from the Day 14 bank, follows the declared
   codec chain and decoder order and ignores inert metadata markers. It reconstructed 116 of 116
   members with zero errors on every set in every mode.
3. **Map descriptors to answers.** A mapper keyed on the response's exact question IDs fills every
   field from the decoded descriptor plus structural facts from the source. The preflight checked
   that the key set equals `answer_ids`, every value is a string, every enum value is in the
   accepted list, and integers and ATT&CK IDs are well-formed. Rules that mattered:
   - Descriptor `role`/`adapter`/`attack` fields win over any graph-derived reading, especially for
     the polyglot and `host-*` families.
   - Bash and Node.js staged chains (`...acquire_remote, write_file, change_mode,
     dispatch_process`) took the ingress technique `T1105` in all three modes; the language
     sub-technique (`T1059.004`/`.007`) is the alternate reading.
4. **Submit by group, not all at once.** `incorrect` is batch-level only, so the Single Solve
   submissions went by family. Attempt 1 (all 1,026) was `incorrect`. Attempts 2-9 each accepted a
   family: core PowerShell fields, role enums, ATT&CK fields, indicators, adapter/language
   identity, profile/representation fields, and stage counts/ordered steps (921 IDs confirmed). The
   remaining 105 were syntax, control and transform fields. Splitting them further (Bash syntax
   primitives, Python native syntax, VBScript transforms and wrappers, Batch reconstruction, JScript
   styles) confirmed all but a few Batch `*-control` fields; those were isolated to singletons, with
   the last field (`batch-launcher-control` = `goto_dispatch`) returning `CORRECT` on submission 31.

## Time Trial and Mastery

**Time Trial.** Each `retry` issued a fresh archive (same 116 members, different generated
content), and the earlier answers did not carry over. The Single Solve decoder and mapper decoded
and mapped all 1,026 fields on every variant, so all non-control fields were accepted in one call
as soon as a variant was submitted inside its window. Several 50-second windows were still lost to
`expired` (submissions 15, 16 and 21 returned nothing gradable).

The nine `batch-*-control` fields were the problem. Attempts 13, 14, 18 and 20 all came back
`incorrect`. The cause was our mapper, which still carried the three per-ID Batch overrides from
Single Solve (`call_subroutine` for the dropper, impact and uploader controls). Overrides do not
transfer: the generator re-creates each structure per variant, and one of the freshly generated
specimens was a bare `set` that is truly `direct`. After removing every override, the mapper
reproduced all nine Single Solve control truths from source alone. On an untimed retry we
classified each control by structure and tested one group per class: `err`/`goto`/`if_same` (5
controls), then one-item `FOR /L ... DO CALL` (2), then plain `call :LABEL` (2). All three groups
were `accepted`, and the 1,017 non-control fields then returned `CORRECT` on submission 25. This
solve was on an untimed retry (`deadline: null`, not Hall-of-Fame eligible). A later scripted rerun
of the same mode (ranked attempt, 50 s window) was `CORRECT` in one call, 1.99 s wall time locally.

**Mastery.** The outer ZIP holds `evidence-01..05.zip`, each a full 116-member set. A small driver
extracted the five sets, ran the Time Trial decoder and mapper per set, stripped and re-added the
`setN.` prefix, and combined them: 5,130 of 5,130 IDs mapped. This time there were 45 Batch
controls (9 per set). The first group, 32 controls of the trusted classes, was accepted in one call.
The `direct` class then failed twice as a group (see below) and cost seven singleton submissions;
a final six-ID group cleared the rest. All 5,085 non-control fields went through in one call,
`CORRECT` on submission 14, again on an untimed attempt. Two later ranked reruns (170 s window) used
the finished solver and were `CORRECT` in one call each (about 4-5 s).

## Dead ends and gotchas

- **The prompt's Batch rule is wrong for the validator.** The question says to classify a one-item
  `FOR /L ... DO CALL` wrapper as `for_once`. The validator rejected `for_once` as singletons in
  Single Solve (dropper, impact and uploader controls) and accepted `call_subroutine`. In Time Trial
  and Mastery the same shape was `call_subroutine` too. We saw this on Days 13 and 14 as well; the
  library's Batch-control entry carries the rule.
- **Per-ID overrides leak.** The three Single Solve overrides were valid only for that attempt's
  bytes. Reusing them in the Time Trial wasted attempts 14 and 18-20.
- **A classifier regex miss poisons a whole class.** The Mastery signature regex matched
  `if defined X set ...` but not `if defined X call set ...`, so that one specimen landed in the
  nine-ID `direct` class. Both the grouped submissions (`direct` and `call_subroutine`) came back
  `incorrect`, forcing singletons. Corrected, that specimen is `if_guard`. Another bare
  `call set "v=%%a%%%%b%%"` is `direct`: it is CALL re-parsing, not a subroutine call.
- **`incorrect` reveals nothing per field.** This GM returns no partial `accepted_ids` on a failed
  composite call, so a 105-ID "syntax/control" group failing tells you only that at least one field
  is wrong. The efficient move was always to split into small, structurally homogeneous groups.
- **Timer vs. group testing.** A 50 s window leaves no room for the group search. Group testing
  needs an untimed attempt, which is what the CORRECT Time Trial and Mastery solves used. The
  signature table is what later made the 1-call ranked reruns possible.
- **A very long `--ids` list on Windows.** Submitting 1,017 IDs on the command line exceeded the
  Windows length limit before anything was sent. Our own tooling issue; see the day overview.

## Takeaways

- **Technique:** static descriptor reconstruction. Decode the per-specimen encoded descriptor
  offline, trust its declared `role`/`adapter`/`attack` fields over graph-derived readings, and
  validate the mapper's key set against `answer_ids` before the first submission.
- **Technique:** classify Batch `*-control` wrappers by structure signature and test one group per
  signature (`err`, `goto`, `if_same`, `if_init`, `F`, `C`, `direct`), not per ID and never from
  another mode's overrides. Library entry SCRIPT-TECHNIQUE-004.
- **Faster next time:** start from a mapper with no overrides; submit all non-control fields in one
  call first; test unconfirmed control classes one group each. A 105-field mixed batch is
  uninformative when it fails.
- **Unresolved:** we could not determine which question, if any, covers the prompt's "store that was
  never there", and the validator did not tell us.

## Evidence

Leaf base: `challenges/day15/script-analysis/script-analysis-day-15-credential-store-clues/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/static_decode.py`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/luna-batch-03-validator-overrides.json` (the three Batch control overrides)
- `time-trial/journal.md`, `time-trial/artifacts/ttrun/` (override-free decoder, mapper and
  class-group driver), `time-trial/artifacts/submission-attempt-log.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/prep_mastery.py`, `mastery/artifacts/run_mastery.py`,
  `mastery/artifacts/submission-attempt-log.json`, `mastery/ranked-1/result.json`,
  `mastery/ranked-2/result.json`
- `tools/ranked/solvers/script-analysis-day-15-credential-store-clues/solve.py` (final solver)
- `library/script-analysis/batch-control-classification.md` (SCRIPT-TECHNIQUE-004)
- `progress.md` (grep `^| 15 .*credential-store-clues`)
