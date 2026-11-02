# Orphan Process

| | |
|---|---|
| Day / Lane | 18 / Script Analysis |
| Challenge ID | `script-analysis-day-18-process-story` |
| Technique | inert-marker descriptor decoding; reuse of a cumulative question bank; new `SPLIT` marker for Google Apps Script |
| Single Solve | Independent (CORRECT saved in `submission-attempt-02-output.txt` and the journal; no `result.json`) |
| Time Trial | Independent (journal-recorded untimed CORRECT; ranked rerun `ranked-1/result.json` CORRECT) |
| Mastery | Independent (journal-recorded untimed CORRECT; ranked rerun `ranked-1/result.json` CORRECT) |

## Summary

The prompt talks about reconstructing a process tree and catching a child that claims a parent it
never had, but the 1,194 questions are the usual Script Analysis descriptor bank: each specimen
carries an inert, encoded descriptor comment, and the answers are fields of that descriptor. Day 18
added exactly one new thing, Google Apps Script (`.gs`) specimens, whose marker is split across a
JavaScript array instead of sitting on one comment line. We added a decoder branch for it, ran the
previous days' validator-confirmed mapper unchanged, and all three modes went CORRECT.

## The challenge

Prompt: "Source code describes which process should spawn which, and native telemetry keeps its own
record of events. Reconstruct the static process tree, and catch the child claiming a parent it
never had." Nothing in the questions is about process trees in the OS sense.

- **Single Solve:** one AES-encrypted ZIP (password `infected`, 119,408 bytes, SHA-256 matched the
  start response) with 132 flat members, 1,194 questions. Untimed.
- **Time Trial:** a fresh archive of the same shape, the same 1,194 question IDs.
- **Mastery:** an outer AES ZIP holding five nested AES evidence ZIPs, 5 x 1,194 = 5,970 answer IDs
  prefixed `setN.`.

Question kinds in Single Solve: 713 `exact_text`, 176 `enum`, 129 `pattern`, 87 `integer`, 87
`ordered_tokens` and 2 `mitre_attack_id`. By suffix the bulk is `attack`, `role`, `campaign`,
`profile`, `adapter`, `url`, `steps` and step `count`.

## Solution

1. **Recon.** Extracted the 132 members with the shared AES-ZIP helper after checking the SHA-256.
2. **Scout the bank.** Compared the new `start_response.json` with earlier days. The bank is
   cumulative: all 1,026 Day 15 IDs are present with identical wording, Day 17's IDs are a subset,
   and Day 18 adds exactly 55 fields. Those 55 are 5 Google Apps Script specimens (four polyglots
   plus one `host-google-docs-sidebar`) with 11 fields each. Everything else is the Day 15 bank,
   regenerated.
3. **The one new decoding problem.** Every other family keeps the descriptor in a
   `HUNTRESS_ANALYSIS_V1:<KIND>-<payload>` comment. The Apps Script specimens only say
   `// HUNTRESS_ANALYSIS_V1:SPLIT` and put the payload in a constant:

   ```javascript
   // HUNTRESS_ANALYSIS_V1:SPLIT
   const HUNTRESS_ANALYSIS_V1_SPLIT_1d4fcda2 = [
       "FRAG-763d337c6c616e6775616765.3d676f6f676c655f61707073.5f7363726",
       ...
   ].join("");
   function buildAnalysisPlan_1d4fcda2() { const encoded_1d4fcda2 = [ ... ].join("") ...
   ```

   The first fragment names the kind (`FRAG`, `HEX`, `B64` or `HB64`, one each across the five
   files), the `.` characters inside fragments are decoration, and `buildAnalysisPlan_<id>` repeats
   the same payload without the kind prefix. We joined the array, split the kind off, stripped the
   dots, decoded by kind, and used the plan-function copy as a free cross-check (it must decode to
   the same bytes; a mismatch raises). All 132 members decoded with zero errors.
4. **Descriptor to answers.** The decoded text is the usual `v=3|key=value|...` descriptor, for
   example `language=google_apps_script|role=staged_chain|attack=T1059|adapter=operation_graph|...`.
   `role`, `attack` and `adapter` were answered from the descriptor's own fields, as for the other
   polyglot and host families, even where the step list would suggest something else (a
   `benign_utility` whose steps are `decode_embedded,write_file,dispatch_process`; a `launcher`
   whose steps are only `decode_configuration,return_configuration`). The mapper itself was the
   Day 15 Mastery mapper, unchanged and with no per-ID overrides; 1,194 of 1,194 IDs mapped, enum
   values all valid.
5. **Submission.** First call: the 1,130 fields that were not Apps Script and not Batch `*-control`
   (accepted; the response reported `remaining: 64`). Second call: the full 1,194-ID map,
   CORRECT, flag `cb77dd4e174785b06750df3932a5d69a`. That second call was meant to be only the 55
   Apps Script fields (see gotchas); it turned out to be the whole map, which was the right answer
   anyway.

## Time Trial and Mastery

Both were first run as untimed regular attempts, using the Single Solve decoder and mapper as-is.

- **Time Trial:** 132/132 decoded, 1,194/1,194 mapped, full map in one call, CORRECT on call 1.
- **Mastery:** a small wrapper extracted the outer ZIP, then each nested set, ran the same
  decoder and mapper per set (132/132 decoded, 1,194/1,194 mapped per set) and merged with the
  `setN.` prefix. The full 5,970-ID map was CORRECT on call 1.
- **Batch controls** were classified from the specimen's structure (one signature per ID) rather
  than from the prompt or from another mode's answers. Time Trial had direct x3, C x2, F x2, err
  and goto; Mastery had goto 11, err 7, direct 7, if_same 7, if_init 2, C 5 and F 6. The one-item
  `FOR /L ... DO CALL` wrapper (class F) was mapped to `call_subroutine`, and `if_guard` for the
  `if_same`/`if_init` signatures; the prompt's alternate reading (`for_once`) was not needed.

A later ranked-sweep rerun used the shared descriptor-bank solver with the `T1105` staged-chain
attack style. Both ranked attempts were CORRECT on the first call: Time Trial in 2.06 s total
inside a 50 s window, Mastery in 4.11 s inside a 170 s window. These ranked runs were
bragging-rights only and are not what the two Independent labels above rest on.

## Dead ends and gotchas

- **The prompt is flavor.** "Process tree" and "spawn" language suggested a different kind of
  analysis. The questions are descriptor fields only; the descriptor's `steps` field is the closest
  thing to a "process story", and even it is answered verbatim from the field.
- **Descriptor beats graph reading.** For the Apps Script specimens the declared `role`/`attack`
  values were accepted even where the step list pointed elsewhere (the odd `benign_utility` and
  `launcher` cases above). The validator confirmed this in the very first full-map call.
- **Batch `FOR /L ... CALL` controls** are the place where the prompt's wording and the accepted
  answer disagree; structural classification, not the prompt, decided them.
- **Accidental full-map submission.** In Single Solve, `gm.py submit --ids ""` selects every
  answer. The ID list was meant to come from a temp file that was never written, so the second call
  carried all 1,194 answers instead of the 55 Apps Script fields. It happened to be CORRECT; it is
  a tooling slip, noted in the day overview, not a property of the challenge.

## Takeaways

- Technique name: **inert-marker descriptor decoding** with the **cumulative question bank**
  shortcut. Before writing anything new, diff the new day's question IDs and wording against the
  previous day's start response; if only one language is new, patch the decoder for it and run the
  previous validator-confirmed mapper unchanged. Here that reduced a 1,194-field day to one new
  marker format.
- When a marker names a *variable* (`SPLIT`) instead of carrying data, look for the array it points
  to and for a second copy of the payload in the same file; the second copy gives a free integrity
  check on the decoder.
- Submit the trusted subset first, then the new family, then the ambiguous controls; but once the
  mapper has survived one full validation, a single full-map call per mode is enough (Time Trial
  and Mastery were one call each).
- Apps Script `SPLIT` decoding is now folded into the consolidated descriptor decoder (library
  entries SCRIPT-TECHNIQUE-D18-SPLIT and SCRIPT-TECHNIQUE-C101).

## Evidence

Leaf base: `challenges/day18/script-analysis/script-analysis-day-18-process-story/`

- `single-solve/journal.md`, `single-solve/start_response.json`
- `single-solve/artifacts/static_decode.py` (Day 15 decoder plus the `SPLIT` branch),
  `single-solve/artifacts/answer_map.py` (the unchanged Day 15 mapper)
- `single-solve/artifacts/submission-attempt-log.json`, `single-solve/artifacts/submission-attempt-01-output.txt`,
  `single-solve/artifacts/submission-attempt-02-output.txt`
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-01-output.txt`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/artifacts/prep_mastery.py`, `mastery/artifacts/control_check.py`,
  `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- `tools/ranked/solvers/script-analysis-day-18-process-story/solve.py` (ranked solver)
- `library/script-analysis/split-marker-apps-script.md` (SCRIPT-TECHNIQUE-D18-SPLIT)
- `progress.md` (grep `^| 18 .*process-story`)
