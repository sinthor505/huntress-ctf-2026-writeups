# The Butler Did It

| | |
|---|---|
| Day / Lane | 20 / Script Analysis |
| Challenge ID | `script-analysis-day-20-jenkins-pipeline-intrusion` |
| Technique | descriptor-bank reconstruction (inert marker decoding) plus structural Batch control classification |
| Single Solve | Independent (2 submission calls) |
| Time Trial | Independent (2 calls untimed; also a timed rerun, 1 call) |
| Mastery | Independent (2 calls untimed, re-solved on a fresh variant; also a timed rerun, 1 call) |

## Summary

"The Butler Did It" is a 1,329-question script-analysis bank built around a Jenkins Pipeline
(Groovy) host plus the families from earlier days. Nothing has to be executed or reverse engineered
in the usual sense: every specimen carries an inert analysis descriptor, and the job is to decode it
and map it onto the answer vocabulary. The bank was cumulative, so the earlier day's decoder and
mapper worked after two decoder patches. Every field except the Batch `*-control` ones was accepted
in one call, and the controls, classified by structure, closed the mode in a second call. Time Trial
and Mastery used the same recipe.

## The challenge

Prompt: "A Jenkins Pipeline splits its Groovy stages between controller and agent, and not every
step runs where it appears to. Recover the pipeline's intent, separate controller semantics from
agent semantics, and trace the modeled build into its prior-language stages."

The prompt is flavor text. The delivery was a password-protected AES ZIP (password `infected`) of
145 specimen files (130,248 bytes for Single Solve, SHA-256 matched the API). Each
question names its file as `evidence.zip -> <member>`. There were 1,329 answer IDs: 796
`exact_text`, 189 `enum`, 142 `pattern`, 100 `integer`, 100 `ordered_tokens` and 2 `mitre_attack_id`.
Per specimen the questions ask for fields like `role`, `attack`, `adapter`, `campaign`, `url`,
`steps`, `obfuscation`, `language` and, for Batch files, a `control` structure.

The ID set was a strict superset of the Days 10-19 banks. The 80 IDs new relative to Day 19 were four
Groovy polyglots, `host-classic-asp-vbscript`, `host-jenkins-pipeline-groovy` (11 IDs: role, attack,
adapter, campaign, url, stage count, steps, host profile, obfuscation, stage profile, next language)
and two Bash features, `mapfile-process` and `coprocess-block`.

## Solution

1. **Recon.** Extracted the archive, checked the SHA-256 against the start response, and diffed the
   answer IDs against the previous accepted bank (see the library's bank-reuse workflow). Only the
   80 new IDs needed attention; the rest could reuse the existing decoder and mapper.
2. **Decode the descriptors.** Each specimen hides a `v=3|key=value|...` descriptor in an inert
   `HUNTRESS_ANALYSIS_V1:` marker (or a language-native encoded literal). Marker payloads come in
   `HEX`, `B64`, `HB64` (hex-wrapped base64) and `FRAG` (dot-separated hex fragments) forms. The
   Groovy/Jenkins files used these same forms (`final String descriptor_<id> = "<hex>"` in the
   Groovy polyglots, a `pipeline { agent none; ... }` host), so they needed no new decoder. Two other
   new files did:
   - **Google Apps Script** uses `HUNTRESS_ANALYSIS_V1:SPLIT`: the real payload is the first const
     string array, joined in order.
   - **The inline ASPX/C# host** puts the marker inside a C# string literal followed by a literal
     `\n` escape, which had to be cut off before decoding.

   ```python
   if m.group(1).strip()=='SPLIT':
       arr=re.search(r'HUNTRESS_ANALYSIS_V1_SPLIT_\w+\s*=\s*\[(.*?)\]\s*\.join\(',s,re.S)
       joined=''.join(re.findall(r'"([^"]*)"',arr.group(1)))
       m=re.match(r'(.*)',joined)
   marker=marker.split('\\n',1)[0]   # C# string-literal hosts: drop the trailing \n escape
   ```

   After the patches all 145 members decoded with 0 errors.
3. **Map descriptors to answers.** The mapper takes the descriptor's own `role`, `attack` and
   `adapter` when present (this held for the Groovy polyglots and the Jenkins and classic-ASP hosts,
   including a polyglot whose declared role differed from what its step list would suggest), and
   falls back to a graph rubric over the `steps` list for families without those fields.
4. **Batch controls by structure.** Day 15's per-ID Batch overrides (`call_subroutine` for the
   dropper, impact and uploader controls) were removed: on Day 20 those fixtures were `if_same`,
   `direct` and `if_same`, so the overrides would have been wrong. Instead each Batch file was
   classified from its own source with `batch_control_signature.py`:
   `if_same` / `if_init` -> `if_guard`, `goto` -> `goto_dispatch`, `err` -> `errorlevel_guard`,
   `F` / `C` wrappers -> `call_subroutine`, `direct` -> `direct`.
5. **Submission.** Two calls: first every field except the nine Batch `*-control` IDs (1,320, all
   accepted, 9 remaining), then the nine controls (`CORRECT`; the Game Master returned the flag
   `b4bcde72e18c779a4e5111e60a5f69b2`).

## Time Trial and Mastery

- **Time Trial.** Same 1,329 IDs, a fresh archive (131,065 bytes, 145 members). The mapper's Batch
  `for_once` branch was changed to `call_subroutine`, since one-item `FOR /L ... DO CALL` wrappers
  were `call_subroutine` here. Same two calls (1,320 accepted, then the nine controls `CORRECT`).
  This attempt was untimed (`deadline: null`, not Hall-of-Fame eligible).
- **Mastery.** Five sets of 1,329 IDs, `setN.`-prefixed (6,645 total), delivered as an outer ZIP of
  `evidence-01..05.zip`, 145 members each. An offline prep script unpacked both layers and ran the
  Time Trial decoder per set, mapping 6,645 of 6,645 with 0 decode errors. 90 of the IDs are
  `-control` fields: 45 Batch ones, held back for the second call, and 45 from other families, which
  go in the first. The first call (6,600 fields) was accepted; the 45 Batch controls then closed it.
- **Re-solve on a fresh variant.** The original Mastery's second call has no recorded result, and a
  later untimed re-start left an open ungraded attempt. We took a fresh variant by `retry`, repeated
  the two-call recipe, and the 45 controls were `CORRECT`. A separate structural classifier agreed
  with the mapper on all 45 (0 mismatches).
- **Timed reruns (bragging rights only).** On a later ranked pass, a general solver for this bank
  (the shared Days 9-10 descriptor bank with an `ingress` staged-chain attack style) went `CORRECT`
  in one call on both modes with all answers in one submission: Time Trial 1,329 answers in 2.46 s
  against a 50 s window, Mastery 6,645 in 4.49 s against a 170 s window.

## Dead ends and gotchas

- **Carried-over Batch overrides were stale.** Described above: a mapper copied from the previous
  day carried per-ID control overrides that were wrong for the new fixtures. Delete per-ID overrides
  when carrying a mapper forward and classify from the file's own structure.
- **The Batch `*-control` fixture style varies by mode.** Single Solve had no `F`/`C` (for/call)
  wrappers, Time Trial one `F`, Mastery both; the structural table covered all of them. The
  one-item `FOR /L ... DO CALL` wrapper is `call_subroutine`; the Time Trial mapper first carried a
  `for_once` branch from another day.
- **Mastery bundle accounting.** Mastery's `-control` IDs are not all Batch (45 of 90), so "all
  controls last" would have been wrong; only the Batch ones needed the structural second call.
- The Time Trial and Mastery regular attempts were untimed, so no timer pressure shaped these solves.

## Takeaways

- **Cumulative bank reuse.** Diff the new answer IDs against the last accepted bank first; the
  previous mapper plus decoder patches for the new marker containers is usually the whole solve.
- **Declared descriptor fields beat graph reasoning** for `role`, `attack` and `adapter` where the
  specimen declares them; use the graph rubric only for families that do not.
- **Submit in two steps**: everything except Batch controls, then Batch controls grouped by
  structural signature. A multi-answer `accepted` count tells you which half failed.
- Decoder hazards for marker containers: `SPLIT` arrays (Google Apps Script) and markers embedded in
  string literals with a trailing `\n` escape (C# script hosts).

## Evidence

Leaf base path: `challenges/day20/script-analysis/script-analysis-day-20-jenkins-pipeline-intrusion/`
(Day 20's `_run/` folder holds no librarian note or lane log for this challenge).

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/static_decode.py`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/prep_mastery.py`,
  `mastery/artifacts/control_by_signature.py`, `mastery/artifacts/submission-attempt-log.json`,
  `mastery/ranked-1/result.json`
- `tools/ranked/solvers/script-analysis-day-20-jenkins-pipeline-intrusion/solve.py` (timed-rerun solver)
- Library: SCRIPT-TECHNIQUE-D20-GROOVY (`library/script-analysis/d20-jenkins-groovy-bank.md`), descriptor
  decoder, descriptor schema and rubrics, batch control classification
- `progress.md` (grep `^| 20 .*jenkins-pipeline-intrusion`)
