# Sharp Practice

| | |
|---|---|
| Day / Lane | 19 / Script Analysis |
| Challenge ID | `script-analysis-day-19-managed-assembly-bridge` |
| Technique | static decoding of inert `HUNTRESS_ANALYSIS_V1` descriptor markers in new placements (Google Apps Script SPLIT, string-embedded ASPX/C#), then rubric-based answer mapping |
| Single Solve | Independent (CORRECT on call 1) |
| Time Trial | Independent (CORRECT on call 1; a later ranked rerun was also CORRECT on call 1) |
| Mastery | Independent (CORRECT on call 1; a later ranked rerun was also CORRECT on call 1) |

## Summary

Sharp Practice is a large script-analysis corpus: 136 specimens across many scripting languages and 1,249 questions. Every specimen carries an inert, encoded descriptor, and every answer is a field of that descriptor or a function of it. The new work was two marker placements the earlier decoder did not handle: a Google Apps Script "SPLIT" array and a marker buried inside a C# string literal in an ASPX host. With those two parsers added, one offline decode-and-map pass was CORRECT on the first submission in all three modes.

## The challenge

Prompt: "A Roslyn C# script and an inline ASPX host act as a bridge that carries managed code back into the script corpus. Recover their layered configuration, resolve each static handoff, and work out where the bridge actually leads."

Delivered: an AES-encrypted ZIP (`evidence.zip`, about 124 KB, 137 members including one wrapper script) with no live target. The Single Solve start response listed 1,249 answer IDs (748 `exact_text`, 181 `enum`, 134 `pattern`, 92 `integer`, 92 `ordered_tokens`, 2 `mitre_attack_id`) across 136 members. Questions ask things like the scenario, the ATT&CK technique, the embedded URL, the adapter or the next language for each specimen. Every answer must be a JSON string.

## Solution

1. **Extract.** The archive is AES-encrypted (password `infected`), so stdlib `zipfile` is not enough: we used `pyzipper`. That package lives in user site, which `python -I` hides, so extraction ran without `-I` and everything that read artifacts ran isolated.
2. **Reuse the bank.** The corpus is the union of the Day 13-15 families (PowerShell, JS/CMD/VBS, recovered Python, Bash, Node.js, and the PHP, Lua, WMI, TypeScript, AutoIt, AppleScript, MSBuild and Perl polyglots and hosts) plus new Ruby, AutoHotkey, Google Apps Script, C# script and ASPX members. The Day 15 parser and mapper handled the old families unchanged.
3. **Handle the two new marker placements.**
   - *Google Apps Script SPLIT.* The first line is `// HUNTRESS_ANALYSIS_V1:SPLIT`, and the marker is a string array joined with `.join("")`. The joined text starts with a kind prefix (`HEX-`, `B64-`, `HB64-` for hex-wrapped Base64, or `FRAG-` for dotted hex fragments). A second `encoded_` array in the plan function looked like a copy, but it did not decode the same way in every kind, so we used only the marker array.
   - *ASPX inline C#.* The marker sits inside an escaped C# string literal (`"// HUNTRESS_ANALYSIS_V1:HEX-<hex>\n#nullable enable\n..."`). A line-wide regex swallowed the escaped `\n...` tail and broke `bytes.fromhex`. The fix was to match only the payload alphabet:

   ```python
   m = re.search(r'HUNTRESS_ANALYSIS_V1:((?:HEX|B64|HB64|FRAG)-[0-9A-Za-z+/=.\-_]+)', s)
   ```

   All 137 members decoded with 0 errors to the usual `v=3|language=...|role=...|attack=...|steps=...` descriptor.
4. **Map descriptor to answers.** The mapper is the Day 15 one with its per-ID overrides removed. For polyglot and host families it uses the descriptor's own `role`, `attack`, `adapter` and `next_language` fields, not a value derived from `steps`. In this bank those fields are independent (for example, role `staged_chain` with steps `decode_configuration,return_configuration`).
5. **Batch `*-control` answers** come from the structural classifier (one group per control-flow signature) rather than from the prompt's wording. The one-item `FOR /L ... DO CALL :LABEL` wrapper mapped to `call_subroutine`.
6. **Preflight and submit.** The map had 1,249 of 1,249 keys, with enum and ATT&CK syntax valid and no unknowns. One full-map call was CORRECT.

## Time Trial and Mastery

- **Time Trial** has the same schema and 1,249 IDs, but the fixtures were regenerated: 340 of the 1,249 answers differ from Single Solve (campaigns, URLs, adapters, paths, Batch controls, JS styles). The unchanged extractor and mapper, re-run on the new archive, gave 137/137 decoded and one CORRECT call. That first attempt was untimed (`deadline: null`).
- **Mastery** delivers a plain outer ZIP of five AES ZIPs (`evidence-01.zip` to `evidence-05.zip`) and 6,245 IDs, five sets of 1,249 prefixed `setN.`. We split it into per-set work directories, ran the same decoder and mapper on each set (all 137/137), merged the answers under `setN.<id>` keys and submitted once: CORRECT. This was also an untimed attempt.
- Later ranked reruns used a generalized solver on fresh fixtures and were CORRECT on call 1 (Time Trial 2.5 s total in a 50 s window; Mastery 4.1 s in a 170 s window). Those are bragging-rights runs, not the scoring basis.

## Dead ends and gotchas

- The ASPX marker lives inside a string literal. If you grab "the rest of the line" you get the escaped C# tail and `fromhex` fails. Match only the payload alphabet.
- The GAS `encoded_` plan array is not an independent check. It is a copy of the marker payload without its `KIND-` prefix, so comparing a Base64 or hex-wrapped Base64 copy as plain hex looks like a disagreement. A later library pass confirmed it agrees once decoded with the marker's own kind. We ignored it and submitted without trouble.
- `python -I` hides `pyzipper`, so a decode that works interactively fails under `-I`. `PYTHONUTF8` is also ignored under `-I`, so pass `-X utf8`.
- No rejected answers: all three modes were accepted on the first call, so there was no validator quirk to chase on this day.

## Takeaways

- **Inert-marker decoding.** These specimens never need to be executed. The answer lives in an encoded descriptor, and the work is finding where each language hides it and decoding it with the right codec (`HEX`, `B64`, `HB64`, `FRAG`).
- A new language usually means a new *placement* of the same marker, not a new puzzle. Check the placement first (comment, string literal, array join, XML comment).
- Answer from the descriptor field that the question names (`role`, `attack`, `adapter`, `next_language`). Do not derive a value from `steps`, and do not trust a graph rule printed in the prompt.
- Classify Batch controls by structure. Keep the mapper free of per-ID overrides, so that Time Trial and Mastery re-fixtures (340 changed answers, 5x the size) need nothing but a re-run.

## Evidence

Leaf base path: `challenges/day19/script-analysis/script-analysis-day-19-managed-assembly-bridge/`

- `single-solve/journal.md`, `single-solve/start_response.json`
- `single-solve/artifacts/static_decode.py` (SPLIT and embedded marker parser), `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/prep_sets.py`, `mastery/artifacts/merge_sets.py`, `mastery/artifacts/submission-attempt-log.json`, `mastery/ranked-1/result.json`
- Library entry SCRIPT-TECHNIQUE-D19-MARKERS (`library/script-analysis/d19-split-and-embedded-markers.md`)
- `progress.md` (grep `^| 19 .*managed-assembly-bridge`)
