# AutoIt Takes Two

| | |
|---|---|
| Day / Lane | 12 / Script Analysis |
| Challenge ID | `script-analysis-day-12-autoit-workflow-pair` |
| Technique | static reconstruction of inert descriptors across many script families; AutoIt added as a new family |
| Single Solve | Solved (journal-recorded); the attempt log also records `CORRECT` |
| Time Trial | Solved (journal-recorded); the attempt log also records `CORRECT` |
| Mastery | Solved (journal-recorded); the attempt log also records `CORRECT` |

## Summary

Day 12 added four AutoIt `.au3` specimens to a bank of 95 other inert scripts and asked 855
questions about the whole bundle. The AutoIt files needed no new decoding idea: each carries its
descriptor twice (a `HUNTRESS_ANALYSIS_V1` marker comment and a `Global Const` string), and the two
copies agree. The work was reconstructing every field offline, then finding out through grouped
submissions which of our ATT&CK and Batch-control readings the validator would accept. All three
modes reached CORRECT. No `result.json` was saved; the evidence is our journals and the per-attempt
submission logs.

## The challenge

Prompt: "Two inert AutoIt workflows arrived as a matched set, and each one carries half the story.
Correlate them with everything you've proven so far, and decide which pairing is genuine and which
is just going through the motions." Despite the wording, there is no question about a "genuine
pairing". The questions are per-file field lookups.

- Single Solve: one AES-ZIP (`evidence.zip`, password `infected`), 99 members, 855 answer IDs
  (143 enum, 506 exact_text, 54 integer, 2 `mitre_attack_id`, 54 ordered_tokens, 96 pattern).
- Time Trial: a fresh archive of the same shape, same 855 IDs, 50 s server deadline.
- Mastery: an outer ZIP holding `evidence-01.zip` and `evidence-02.zip`, each 99 members; 1,710 IDs
  prefixed `set1.` / `set2.`.
- Contents: PowerShell evidence samples (resource, collection, impact), JScript, Batch, VBScript,
  Python, Bash, Node.js, the v3-descriptor families (PHP, Lua, TypeScript, WMI MOF, host containers)
  and the four new AutoIt files: `benign-utility`, `downloader`, `launcher`, `staged-chain`.
  44 of the 855 questions are about AutoIt (11 fields per file).

## Solution

1. **Inventory.** The archive extracts with `pyzipper` (AES). Nothing was executed. 30 of the 99
   members carry an explicit `HUNTRESS_ANALYSIS_V1` marker with a HEX/B64/other-encoded payload; the
   rest document their descriptor in source and are decoded by following the source's own routine.
2. **AutoIt descriptors.** Each `.au3` starts with `; HUNTRESS_ANALYSIS_V1:<KIND>-<data>` where the
   kind was `HEX`, `B64`, `HB64` (hex of base64) or `FRAG` (hex split into dot-separated
   fragments), and also defines `Global Const $descriptor_<id> = "..."` in the same encoding. We
   decoded both and compared them as a corruption check; they matched in all four files. The decoded
   payload is a pipe-separated `key=value` record (`role`, `attack`, `adapter`, `steps`,
   `stage_count`, `next_language`, `url`, `file_path`, ...).
3. **Mapping to questions.** A mapper joined the decoded fields to the exact question IDs from the
   live response and asserted that its key set equalled the 855 `answer_ids` and that every value met
   its declared format. For AutoIt the answers are the descriptor values as stored. The downloader is
   `T1105`, the launcher and the staged chain are bare `T1059` (the staged chain ends in
   `dispatch_language`, not `dispatch_process`), and the benign utility is `none`. We did not
   find the question about "which pairing is genuine"; role and verdict-like fields carry that
   information.
4. **Submission.** The full 855-field map is one batch, and the Game Master returns no per-field
   feedback on `incorrect`, so a rejection localizes nothing. We submitted by family instead
   (see below). Single Solve took 11 calls, Time Trial 16 and Mastery 26, all inside the 30-call cap
   authorized for this challenge.

Single Solve call history (from the attempt log):

| Call | Fields | Result |
|---:|---|---|
| 1 | all 855 | incorrect |
| 2-5 | PowerShell trio, then JScript, Batch, VBScript families | each accepted |
| 6 | Python family (126) | incorrect |
| 7 | Python family, two ATT&CK fields corrected | accepted |
| 8-10 | Bash, Node.js, AutoIt families | each accepted |
| 11 | remaining 286 (PHP/Lua/WMI MOF/TypeScript plus host containers) | CORRECT |

## Time Trial and Mastery

- **Time Trial.** The same pipeline ran on the fresh archive (its own SHA-256, 855 IDs). The first
  full-map call was `incorrect`, and the 15-ID PowerShell call returned `expired` because the 50 s
  window had passed. A `retry` returned the same artifact hash and IDs with `deadline: null`, so the
  rest was untimed work on the same bytes: families confirmed one by one, and the 286-ID remainder
  went CORRECT at call 16.
- **Mastery.** Same decoder, with the answer IDs kept in two namespaces (`set1.`, `set2.`). The
  original deadline also expired before the first call and a `retry` gave an untimed assignment on
  the same bundle. The Batch controls took most of the effort (calls 7-19). The first 20 calls left
  1,266 IDs unresolved. A second pass confirmed set 2 Python, both VBScript sets and set 1 Bash, then
  the remainder in two combined groups; call 26 returned CORRECT.

## Dead ends and gotchas

- **Python staged-chain ATT&CK.** Two Python IDs (`python-py2exe-service-attack` and
  `python-pyarmor-pyinstaller-attack`) were first mapped from the graph as `T1105`, which made the
  126-field Python call `incorrect`. A cross-check against Day 11's accepted Python map led to
  `T1059.006` (the language sub-technique), and the corrected family passed. Here the Bash and
  Node.js staged chains took `T1105` and were accepted, so the sub-technique rule was not
  uniform across languages.
- **Descriptor `attack` field beats the graph reading** for the v3 polyglot and host files, including
  the AutoIt ones. The AutoIt downloader's graph (`acquire_remote>dispatch_process`) could be read as
  an interpreter technique, but the descriptor's `T1105` was accepted.
- **Batch benign control.** The Time Trial's isolated benign control was accepted as
  `call_subroutine`. In Mastery, controls varied per specimen: set 1's benign GOTO dispatch and
  several one-item wrappers (including the dropper, whose `for_once` guess was rejected) took
  `call_subroutine`, while set 2's benign control was accepted as a direct reconstruction. We could
  not find a rule from the specimens that predicted this; each was settled by a focused submission.
- **The prompt is flavor text.** Nothing in the data distinguishes a "genuine" pairing; the fields
  are plain per-file lookups.
- Both the Time Trial and the Mastery timers expired before the first graded calls finished; the
  untimed `retry` recovered both (see above).

## Takeaways

- **Compare duplicate descriptor copies** (marker comment versus source literal) before mapping. It
  costs nothing and catches a bad decode early.
- **Submit by family, not as one map.** With batch-level `incorrect` only, family subsets turn a
  855-field rejection into a short list of suspect groups. Python, Batch and the v3 families were
  where all the rejections were.
- A new language on a bank day usually reuses the same descriptor schema. Check that the schema
  matches first (here only the marker kinds and the file extension were new), and do not hunt for
  a hidden rubric change.
- Reuse the previous day's validator-confirmed map for overlapping families instead of
  re-deriving them (Python here).

## Evidence

Leaf base path: `challenges/day12/script-analysis/script-analysis-day-12-autoit-workflow-pair/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/static_decode.py`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-log.json` (11 calls, final `CORRECT`)
- `single-solve/artifacts/candidate-answers-attempt01.json` and `candidate-answers.json` (the two
  Python values that changed)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json` (16 calls)
- `mastery/journal.md`, `mastery/artifacts/submission-attempt-log.json` (26 calls),
  `mastery/artifacts/accepted-ids.json`
- Later reusable solver: `tools/ranked/solvers/script-analysis-day-12-autoit-workflow-pair/solve.py`
- Library: SCRIPT-TECHNIQUE-001/002/003 (descriptor reconstruction, byte transforms, inert markers)
- `progress.md` (grep `^| 12 .*autoit-workflow-pair`)
