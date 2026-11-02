# Fetch Quest

| | |
|---|---|
| Day / Lane | 17 / Script Analysis |
| Challenge ID | `script-analysis-day-17-download-graph` |
| Technique | descriptor-bank reuse: decode inert per-specimen metadata, map it to a large answer bank; structural Batch-control classification |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (original attempt journal-recorded; ranked rerun has a saved CORRECT `result.json`) |
| Mastery | Independent (original attempt journal-recorded; ranked rerun has a saved CORRECT `result.json`) |

(Not to be confused with the Day 24 web challenge that has the same display title.)

## Summary

The prompt promised a "download graph" built by reconciling source code with network evidence. No
network evidence existed anywhere in the delivery: the real task was the same large "descriptor bank"
as the previous script-analysis days, 1,139 answer fields over 127 specimens in 17 languages. The
decoder and mapper from Day 15 carried over without code changes, so every mode was a single
submission of a complete map, and every mode went CORRECT on call 1.

## The challenge

Prompt: "The source code says where it would fetch from, and the generated network evidence says where
the requests actually went. Reconcile both into one download graph, and find the indicator that shows
up on only one side of the story."

Delivered:

- **Single Solve:** one AES-encrypted ZIP (114,808 bytes, password `infected`) holding 127 specimen
  files (PowerShell, JScript, Batch/CMD, VBScript, Python, Bash, Node.js, PHP, Lua, TypeScript, WMI
  MOF, AutoIt, AppleScript, MSBuild, Perl, Ruby, AutoHotkey). 1,139 questions.
- **Time Trial:** the same layout and the same 1,139 question IDs, but a fresh variant: 325 of the
  1,139 answer values differed from the Single Solve fixture.
- **Mastery:** an outer ZIP of five inner AES archives (`evidence-01..05.zip`), 5 x 1,139 = 5,695
  questions, IDs prefixed `set1.` to `set5.`.

Answer formats were mostly `exact_text` (678), then `enum` (171), `pattern` (124), `integer` and
`ordered_tokens` (82 each), plus two `mitre_attack_id`. Typical questions: a specimen's role
(`downloader`, `stager`, `dropper`, `relay`, ...), its `attack` technique, its `adapter`, an embedded
URL or campaign id, the step list and step count, and (for Batch) the `control` wrapper.

## Solution

1. **Check the bank before the prose.** The start response listed no question about network
   evidence or a download graph. Diffing the ID set against Day 15 "Crumb Trail" showed Day 17 was a
   strict superset of it: 113 new IDs (Ruby and AutoHotkey polyglots, a Ruby Rack host container,
   and two more advanced-Bash features such as ANSI-C quoting and regex conditionals). Every new ID
   belonged to a family whose marker format was already parsed.
2. **Decode the inert descriptors.** Each specimen carries its metadata as inert data: either a
   language-specific encoded literal (base64, reversed base64, URL-safe base64, `\u00XX` escapes,
   percent-hex, JSON number arrays, with an optional named byte transform such as XOR or rolling XOR)
   or a `HUNTRESS_ANALYSIS_V1:` comment marker (hex, base64, hex-of-base64, or fragmented).
   Decoding yields a `key=value|key=value` record (`role`, `attack`, `adapter`, `steps`,
   `campaign_id`, URL, ...). We never executed a specimen; everything was static. The Day 15
   `static_decode.py` decoded all 127 members with 0 errors.
3. **Map descriptors to answer IDs.** The Day 15 Mastery `answer_map.py` (which had no per-ID
   overrides) turned the records into answers. For the new polyglot and host families, `attack` was
   the descriptor's own `attack` field, which is the rule that had held on earlier days.
4. **Classify Batch `control` fields structurally.** The one field that is not a plain descriptor
   lookup is the Batch `*-control` question: how the specimen's reconstruction statement is wrapped.
   The mapper's old logic was replaced by a signature classifier over the generated fixture layout
   (one signature per structure: `err`, `goto`, `if_same`, `if_init`, `F`, `C`, `direct`). The only
   control field in the Single Solve was a one-item `FOR /L ... DO CALL :label` wrapper:

   ```python
   # batch_control_signature.py
   if re.search(r'\bfor\s+/l\s+%%\w+\s+in\s*\(\s*1\s*,\s*1\s*,\s*1\s*\)\s+do\s+call\s+:', seg, re.I):
       return 'F'
   # answer_map.py
   SIG_ANSWER = {'err': 'errorlevel_guard', 'goto': 'goto_dispatch', 'if_same': 'if_guard',
                 'if_init': 'if_guard', 'F': 'call_subroutine', 'C': 'call_subroutine',
                 'direct': 'direct'}
   ```

   `F` was answered `call_subroutine`, the value the validator had accepted for this structure on
   earlier days (a one-item `FOR` wrapper is not always answered `for_once`; Day 11 differed).
5. **Submit once.** All 1,139 fields in one call: `CORRECT`.

## Time Trial and Mastery

- **Time Trial:** a fresh archive (114,131 bytes, hash matched the start response). The Single Solve
  `static_decode.py` and `answer_map.py` ran unchanged: 127 members decoded, 0 errors, 0 `UNKNOWN`
  values. The Batch controls were re-classified from the new bytes rather than copied from the Single
  Solve, because the fixture regenerates the structure per attempt. Complete map, one call, `CORRECT`.
- **Mastery:** outer ZIP (572,491 bytes) of five inner archives. A small prep script extracted the outer
  and inner archives, built one working directory per set that mimicked the Single Solve layout (set
  start response with the `setN.` prefix stripped), ran the same decoder and mapper per set, and merged
  the results back under `setN.` IDs. 5,695 of 5,695 IDs mapped with no missing, extra or `UNKNOWN`
  IDs. One call, `CORRECT`.
- **Ranked reruns.** Later, ranked attempts were run for both modes (50 s Time Trial window, 170 s
  Mastery window) with a thin solver wrapper around the shared bank decoder; each was one submission
  and `CORRECT`, taking 3.0 s and 4.3 s end to end. These were bragging-rights attempts only, not the
  scoring basis, and they are the saved graded evidence for those two modes.

## Dead ends and gotchas

- **The prompt describes something that is not in the bank.** "Reconcile source with generated network
  evidence" matched no file and no question. The same happened on several neighbouring days. Compare
  the question IDs and the artifact list first; the narrative is flavour.
- **Fresh variants change values, not structure.** About 325 of 1,139 values differed in the Time
  Trial (campaigns, URLs, adapters, paths, controls). The pipeline is reusable, the values are not, so
  re-decode every variant instead of carrying an answer table over.
- **Per-ID Batch overrides do not travel.** The Batch control structure regenerates per attempt;
  the classifier works from the specimen's own text and was accepted without any override.
- **Tooling trap:** `python -I` hides user-site packages, so the AES ZIP step could not import
  `pyzipper` until the script added the user site path itself (see the day overview).

No submission was rejected in any mode, so there were no validator-diagnosis rounds to report.

## Takeaways

- **Descriptor-bank reuse.** Script-analysis banks from Day 12 onward are cumulative: each day is the
  previous day's ID set plus a few new specimen families, with identical question text. The first
  move is an ID-set diff against the last confirmed bank, then running the confirmed decoder and
  mapper on the new bytes, and only then reading the prose. Before spending the submission, confirm 0
  decode errors, 0 `UNKNOWN` and 0 missing IDs.
- **Classify Batch wrappers by structure, one group per signature.** The `control` answer for a
  one-item `FOR /L ... DO CALL` is `call_subroutine`; do not infer it from the prompt wording.
- **Mastery = N copies of the Single Solve problem.** Build per-set directories that look like the
  single-solve layout and reuse the whole pipeline instead of writing a bundle-specific solver.

## Evidence

Leaf: `challenges/day17/script-analysis/script-analysis-day-17-download-graph/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/static_decode.py`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-log.json` (one call, `CORRECT`; no `result.json` was saved)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-1/pass.log`
- `mastery/journal.md`, `mastery/artifacts/prep_mastery.py`, `mastery/artifacts/prep-report.json`,
  `mastery/artifacts/submission-attempt-log.json`, `mastery/ranked-1/result.json`
- `tools/ranked/solvers/script-analysis-day-17-download-graph/solve.py` (ranked wrapper)
- Library: SCRIPT-TECHNIQUE-005 (descriptor-bank reuse), SCRIPT-TECHNIQUE-004 (structural Batch
  control classification), `library/script-analysis/batch_control_signature.py`
- `progress.md` (grep `^| 17 .*download-graph`)
