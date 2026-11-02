# Host With the Most

| | |
|---|---|
| Day / Lane | 2 / Script Analysis |
| Challenge ID | `script-analysis-day-02-encoded-intent` |
| Technique | Static descriptor recovery from obfuscated JScript (WSH) and PowerShell specimens; source-shape classification (function / invocation / property style) |
| Single Solve | Solved (journal-recorded): CORRECT on submission 10 of 20 per the journal; no result file saved in the leaf |
| Time Trial | Independent (CORRECT on submission 1; also a ranked rerun, CORRECT in 1 call) |
| Mastery | Independent (CORRECT on submission 1; also a ranked rerun, CORRECT in 1 call) |

## Summary

Each specimen carries a small encoded "descriptor" (an ordered behavior graph plus a few indicators).
Decoding the descriptors statically and applying the prompt's role and ATT&CK rubric answers most of
the 87 fields. The rest are questions about the *syntax* of the script itself, and two of those
classifiers were wrong on the first try. Once fixed, the same mapper solved Time Trial and Mastery in
one submission each, inside a window of a few dozen seconds.

## The challenge

The prompt: "Windows Script Host has JScript plans for execution, download, upload, and impact. Carry
your PowerShell findings forward, tell a function call from an invocation or a property access, and
work out which script hands off to which."

The delivery is an AES-encrypted ZIP (password `infected`) with 12 specimens:

- 3 PowerShell files (`resource-sample.ps1`, `collection-sample.ps1`, `impact-sample.ps1`).
- 9 JScript files (`host-context.js`, `memory-stage.js`, `embedded-write.js`, `remote-write.js`,
  `local-dispatch.js`, `staged-handoff.js`, `outbound-transfer.js`, `impact-plan.js`,
  `relay-handoff.js`).

There are 87 answer fields, all strings:

- PowerShell: scenario (enum), ATT&CK technique ID, and three exact indicators each
  (URL, wallet, mutex, filename, user-agent, extension, recovery-note name). 15 fields.
- JScript, per specimen: `role` (enum), `attack`, `adapter`, `campaign`, one specimen-specific value
  (URL, path, host, next-stage filename, or extension and note name), plus `function-style`,
  `invocation-style` and `property-style`. 72 fields.

The prompt prints the role and ATT&CK rules in full (for example "benign_utility for
decode_configuration then return_configuration", "dropper for decode_embedded then write_file",
"staged_chain ... or a graph ending in handoff_script").

## Solution

1. **PowerShell descriptors.** Each sample builds a byte array from
   `[Convert]::FromBase64String('...')`, passes it through an identity "reconstruct" block and pipes
   the UTF-8 text to `ConvertFrom-Json`. Nothing needs executing: base64-decode the literal and parse
   the JSON. The keys (`a`, `b`, `c`, ...) hold the URL, wallet, mutex and similar indicators. Scenario
   and ATT&CK ID are fixed per sample (`resource_hijacking`/`T1496`, `synthetic_collection`/`T1555.003`,
   `synthetic_impact`/`T1486`).

2. **JScript descriptors.** Every file has the same shape: a `decode_*` function (XOR with a key array,
   a rolling or repeating-key XOR, or add/subtract mod 256), a `recover_*` function that assembles the
   payload (`firstPart.concat(secondPart)`, a dash/colon-hex/percent-octet `packedText` string, or a
   plain array, sometimes reversed) and an `inspect_*` function that `eval`s the recovered JSON. We
   rebuilt the payload bytes with regexes, replayed the decoder in Python and parsed the resulting JSON
   (`steps`, `adapter`, `values`, `stageCount`). The scripts were never run.

3. **Role and ATT&CK from the step list.** `role_for_steps` and `attack_for_steps` implement the
   rubric text as ordered rules. The ordering matters because the prompt says the terminal step wins:

   ```python
   if steps and steps[-1] == "handoff_script":
       return "staged_chain"
   if steps and steps[-1] == "dispatch_next_stage":
       return "relay"
   ...
   def attack_for_steps(steps, language):
       subtechnique = "T1059.007" if language == "jscript" else "T1059.003"
       if steps and steps[-1] in {"handoff_script", "dispatch_next_stage"}:
           return subtechnique
       if steps[:2] == ["decode_configuration", "return_configuration"]:
           return "none"
       if "acquire_remote" in steps:
           return "T1105"
       ...
   ```

   The accepted JScript attack values were `none` (benign), `T1105` (stager, downloader), `T1140`
   (dropper), `T1059.007` (launcher, chain, relay), `T1041` (uploader) and `T1486` (impact).

4. **"Which script hands off to which."** The chain specimen's `next_stage` value names a PowerShell file
   (`resource-sample.ps1`) and the relay specimen's names a JScript file (`local-dispatch.js`). The
   answers are those filenames, read from the descriptor. In the saved Single Solve map the
   `relay-handoff.js` specimen took role `staged_chain` (its graph ends in `handoff_script`), not
   `relay`; the role rules apply to the graph, not the file name.

5. **Style questions are about source shape.** Three questions per JScript file, answered from the
   text of `inspect_*`:
   - `function-style`: `declaration`, `anonymous_expression` or `named_expression`.
   - `invocation-style`: how `recover_*` is invoked inside `inspect_*` (`direct`, `call`, `apply`).
   - `property-style`: `dot`, `bracket_literal` or `mixed`, counting only the accesses to
     `steps`/`adapter`/`values`/`stageCount` made on the variable assigned from `eval(recover_*())`.

6. The mapper `answer_map.py` prints one JSON object keyed by the live `answer_ids` and exits with an
   error if the key set differs from the start response. The values were submitted by family, as
   described below.

## Time Trial and Mastery

The bank is the same (3 PowerShell + 9 JScript files); only the fixtures (names, keys, indicators,
style mix) change per attempt.

- **Time Trial:** 87 fields, server window 50 s. Even a plain `start` without `run_for_time` returned a
  deadline. The unchanged Single Solve mapper ran on the fresh bytes (0 missing, 0 extra IDs) and one
  scripted pass of download, map and submit returned CORRECT on the first call.
- **Mastery:** 174 fields, window 80 s. The outer AES ZIP holds `evidence-01.zip` and
  `evidence-02.zip`, one full bank each, and the IDs are prefixed `set1.` / `set2.`. The Mastery
  `answer_map.py` is a thin wrapper that applies the Single Solve rules to each inner ZIP. A dry run on
  the Single Solve ZIP before starting the timer reproduced the accepted Single Solve map exactly,
  and the live pass finished about 2 s after the start. One submission, CORRECT.
- **Ranked reruns (2026-10-26):** one further scripted pass per mode went CORRECT in one call each, in
  about 1.6 s (Time Trial) and 1.8 s (Mastery) end to end, which the local logs record.

## Dead ends and gotchas

Single Solve took ten submissions. The Game Master returns `incorrect` only for the whole batch, so we
split the map into families to find the bad ones:

| Call | Subset | Result |
|---|---|---|
| 1 | full map (87) | incorrect |
| 2 | PowerShell (15) | accepted |
| 3 | JScript facts: adapter, campaign, specimen value (27) | accepted |
| 4 | roles (9) | accepted |
| 5 | all three style families (27) | incorrect |
| 6 | attacks (9) | accepted |
| 7 | `function-style` (9) | incorrect |
| 8 | `invocation-style` (9) | accepted |
| 9 | `property-style` (9) | accepted |
| 10 | corrected `function-style` (9), whole map | CORRECT (87/87) |

The two traps, both in the style classifiers:

- **Named function expressions.** `remote-write.js` uses `var inspect_x = function inspect_x_body() {`.
  A regex that searched for `function inspect_...` first read it as `declaration`. The correct answer
  is `named_expression`. The check for the `var inspect_x = function` forms has to come before the
  declaration check. This was the one rejected field in the `function-style` family (the downloader).
- **`property-style` must be scoped.** The prompt says to ignore `.call`/`.apply` and properties of
  other objects. The `inspect_*` body also contains a helper `table` object and a returned object literal
  with its own `steps:`/`values:` keys. Counting every `.steps` or `["adapter"]` in the function gave
  wrong results; counting only accesses on the `eval` result variable gave the accepted values.

  The journal records that the first batch failed and that these two fixes were needed. It does not
  say which specimens' `property-style` values the first version got wrong, so we do not list them.

Other gotchas:

- The older `bundle.zip` copy in the leaf did not match the API artifact's SHA-256. The solve used the
  `evidence.zip` that matched.
- The ZIP is WinZip AES, so `pyzipper` is needed. Under `python -I` the user site is hidden, so the
  solver appends the path itself.
- Time Trial and Mastery are timed even when an untimed start is requested, so the pass is scripted
  and tested offline beforehand.

## Takeaways

- Treat the encoded descriptor as data: recover it statically (concat/reverse/XOR replay) and let the
  prompt's rubric run as an ordered rule list. The terminal step wins; do not classify by file name.
- Answer syntax-style questions from a parse of the exact function the prompt names, and scope each
  regex to that function body and to the specific variable.
- Submit by family. On a batch-level `incorrect` this isolated the two bad classifiers in a handful of
  calls instead of guessing.
- Dry-run the mapper on a saved ZIP before starting a timed attempt. With that done, a 50 or 80 s
  window was not a constraint. The Day 2 solver was also the base for the Day 3 script challenge,
  which adds Batch specimens (library techniques: descriptor bank reuse, variant corpus regression).

## Evidence

Leaf base: `challenges/day02/script-analysis/script-analysis-day-02-encoded-intent/`

- `single-solve/journal.md`, `single-solve/start_response.json`
- `single-solve/artifacts/static_decode.py` (decoders, `role_for_steps`, `attack_for_steps`, `js_styles`),
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/candidates.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`
- `time-trial/ranked-1/result.json`, `time-trial/ranked-1/pass.log`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`
- `mastery/ranked-1/result.json`
- Progress rows: `progress.md`, Day 2 rows for this challenge. They record Single Solve as solved on
  2026-10-20 after 10 submissions and Time Trial and Mastery as solved in 1 submission each. An
  earlier "Player-directed local mark" basis was superseded by the 2026-10-20 re-solve.
- No `result.json` was saved for the three regular attempts; those results are recorded in the
  journals and `progress.md`. The Single Solve flag was not recorded.
