# All the World's a Stager

| | |
|---|---|
| Day / Lane | 9 / Script Analysis |
| Challenge ID | `script-analysis-day-09-stager-relationships` |
| Technique | Static reconstruction of encoded "inert descriptors" in 83 scripts across 9 languages, then rubric-driven classification (role, ATT&CK, wrapper style) |
| Single Solve | Independent (CORRECT on submission 43; 695 fields) |
| Time Trial | Independent (CORRECT on the untimed retry, request 10 of that mode; later ranked reruns also CORRECT) |
| Mastery | Independent (CORRECT in 1 call; 3,475 fields; ranked reruns also CORRECT) |

## Summary

The archive is a set of synthetic malware-style scripts. Each one hides a small key/value "descriptor" (role, ordered behavior steps, campaign ID, next language, ...) behind a per-file encoding, and the questions ask for those fields or for classifications derived from them. Decoding was mechanical. Nearly all of the 43 Single Solve submissions went on the classification rubric, where the validator disagreed with the prompt text in several places. Once that map was validated, Time Trial and Mastery were each solved with a reused decoder and mapper.

## The challenge

Prompt: "Every stager claims to know who came before it and who comes next. Build the relationship graph from final-byte evidence alone, and expose the stage whose claimed family tree is impossible."

Single Solve delivered one AES-encrypted `evidence.zip` (password `infected`) with 83 script specimens and 695 answer fields. The families:

- PowerShell samples (resource, collection, impact)
- JScript, Batch, VBScript, Python, Bash and Node.js specimens in role-named files such as `memory-stage`, `embedded-write`, `remote-write`, `outbound-transfer`, `local-dispatch`, `staged-handoff`, `relay-handoff`
- "Advanced" Bash files (`trap-handler.sh`, `recursive-walk.sh`, ...)
- PHP and Lua polyglots
- `host-*` containers (WSF, HTA, SCT, PHP webshell, OpenResty Lua)

Question kinds:

- exact text (URL, adapter, campaign ID, file path, wallet, mutex)
- integers (stage count)
- `>`-joined ordered tokens (the behavior graph)
- enums (role, Batch expansion/control wrapper, VBScript transform/control)
- ATT&CK technique IDs

No field asks for the "impossible family tree" directly.

## Solution

1. Extracted the 83 members with `pyzipper` (system `unzip` cannot read AES members) and treated them as data. Nothing was executed.
2. Found the descriptor in each file. There are two shapes. v1 is pipe-delimited `key=value` text (`steps=acquire_remote,dispatch_memory|adapter=...|next_language=...`). v3 is used by the PHP/Lua polyglots and `host-*` files and adds `role`, `verdict`, `attack`, `adapter`, `host_profile`, `obfuscation_profile`, `stage_profile`, `next_language` and `stage_count`.
3. Reversed each file's own encoding, following the decoder in the source rather than guessing. Representations seen: JSON number arrays, reversed or URL-safe or chunked Base64, `\u00XX` escapes, percent-hex, hex variants and decimal lists with various separators. Transform families on top: identity, single-byte XOR, repeating-key XOR, add/subtract mod 256, byte reversal, rolling XOR, reverse-then-XOR. `static_decode.py` produced `decoded-descriptors.json` (83 of 83 decoded, 0 errors) and `answer_map.py` mapped it to the 695 IDs.
4. Read exact-text, integer and ordered-token fields straight from the decoded descriptor. The classification fields were harder (see below).
5. Submitted in groups (by family, then by suffix, then singletons) because a mixed batch only ever returns `incorrect`. Accepted groups bank their fields, so the failing group shrinks. Cumulative accepted fields went 15 (call 2), 607 (call 10), 638 (call 19), 670 (call 30) and 695 (call 43).

On the "impossible family tree" clue, the decoded `polyglot-php-staged-chain.php` claims `stage_profile=cross_language_handoff` with `next_language=php`, a cross-language handoff to its own language. That is the strongest candidate, but we never confirmed it. The validator exposes no field for it, so treat it as our interpretation.

## Time Trial and Mastery

Time Trial (50 s window) re-issued the same 695-field shape with a fresh archive, and Mastery (170 s window) delivered an outer ZIP of five nested `evidence-0N.zip` sets (83 specimens each, IDs `setN.<id>`, 3,475 fields). Fixtures are regenerated on every attempt, so specimens and values differ from Single Solve.

- **Original Time Trial attempt.** The mapper produced all 695 values, but the submit came after the 50 s deadline. The request also carried the bare challenge ID rather than `<id>/time_trial`, so the service answered `already_solved` and scored nothing (see the day overview for this harness mistake).
- **Retry 2** was timed and expired. The first mapper pass left 6 fields unmapped because the parser had not followed source order for reversals (below).
- **Retry 3** was untimed. The first full map was `incorrect`. Grouped diagnostics accepted the 18 v3 embedded-`attack` rows, the 7 ordinary Python rows and the remaining 50 non-v3 non-Python pattern rows. The 7 package/runtime Python rows needed two interpreter-technique corrections. The full corrected map then returned CORRECT. That is 10 requests for the mode, the first 2 of them spent on the unscored and expired attempts.
- **Mastery** needed two parser extensions: following reversal order inside JavaScript decoders and recovery routines, and joining adjacent quoted Bash Base64 fragments in one assignment. With the Time Trial ATT&CK corrections applied, one 3,475-field submission went CORRECT before the deadline.
- **Ranked reruns.** `ranked-1` and `ranked-2` exist for both modes, each CORRECT in one call (Time Trial `t_total` 1.86 s and 3.54 s; Mastery 3.56 s and 6.68 s). They used the per-challenge solver in `tools/ranked/solvers/`, which re-implements the same logic on the library descriptor decoder.

## Dead ends and gotchas

- **v3 descriptor `attack` wins over the graph.** For PHP/Lua polyglots and `host-*` files, the descriptor's own `attack` field was the accepted answer, even when the steps pointed elsewhere. Two PHP files had identical decoded graphs (`decode_configuration,return_configuration`) but needed different answers: `polyglot-php-benign-utility` wanted `none`, `polyglot-php-uploader` wanted `T1041`. Singleton submissions (calls 40-41) found this. Calls 42-43 confirmed the rule for the rest, and all 14 remaining v3 rows were accepted in one call.
- **`dispatch_language` readings that failed.** For the five graphs ending in `dispatch_language` we tried the receiving language (`next_language`), the specimen's own language technique and the parent `T1059`. All were rejected in aggregate. Only the descriptor's own value passed.
- **Python chains ending in `dispatch_process`.** `T1105` was rejected for `py2exe-service` and `pyarmor-pyinstaller` (steps `profile_host,acquire_remote,decode_embedded,write_file,dispatch_process`) and `T1059.006` was accepted. The same ingress reading was fine for Bash and Node.js chains (`T1105` accepted) and for the Python `pyarmor-source` stager. The library's behavior-rubric notes record this split.
- **Batch controls.** Our first mapper only searched lines before the reconstruction assignment and answered `direct`. The four files with `if defined` on that statement were `if_guard` (call 19). A one-item `for /l ... do call :LABEL` wrapper was `call_subroutine`, although the question explicitly says to classify that case as `for_once`: `for_once` was rejected as a singleton for the dropper and the downloader, and `call_subroutine` was accepted. The Batch expansion forms (`percent`, `delayed`, `call_double`, `for_indirect`) were accepted once isolated (call 17); the failures were all in the control wrappers.
- **Ordered transforms do not commute.** A reversal in the enclosing recovery call, a reversal inside the decoder, and an XOR must be applied in source order. Getting this wrong left 6 fields unmapped in Time Trial retry 2.
- **Aggregate `incorrect` hides which field failed.** Batches of 34 and 124 fields were rejected without any hint, which is why the bisection took 43 calls.
- **Mode IDs.** Time Trial and Mastery submissions need `<id>/time_trial` or `<id>/mastery`. See the note above.

## Takeaways

- Descriptor reconstruction is reliable. Following each file's own decoder gave 83 of 83 correct descriptors, and the exact-text fields were never the problem. Spend the effort on the classification fields.
- When a rubric prompt and the validator disagree, trust the descriptor's declared field (v3 `attack`/`role`/`adapter`) and the structural shape of the wrapper over the prose, and bisect by family.
- Keep a validated answer map per mode and diff fixtures against it. Time Trial and Mastery then went through with small parser fixes, and later ranked reruns solved in under a second of compute.
- The challenge's reusable parts were generalized into the library's `material_codec_decode`, `behavior_rubric` and descriptor-decoder techniques. The leaf-specific answer-ID mapping stays in the leaf.
- Flag recorded by the Single Solve response: `ace90a13e93ec47d950a71daae836191`.

## Evidence

Leaf base: `challenges/day09/script-analysis/script-analysis-day-09-stager-relationships/`

- `single-solve/journal.md`, `single-solve/artifacts/static_decode.py`, `answer_map.py`, `decoded-descriptors.json`, `submission-NN-request/response.json` (43 pairs; `submission-43-response.json` reads CORRECT).
- `time-trial/journal.md`, `time-trial/artifacts/submission-01-*.json` (`already_solved`), `time-trial/retry-02/` (expired), `time-trial/retry-03/artifacts/submission-10-response.json` (CORRECT), `time-trial/ranked-1/result.json`, `ranked-2/result.json`, `open-1/result.json`.
- `mastery/journal.md`, `mastery/artifacts/bundle_extract_and_map.py`, `mastery/artifacts/submission-01-response.json` (CORRECT), `mastery/ranked-1/result.json`, `ranked-2/result.json`.
- Later per-challenge solver: `tools/ranked/solvers/script-analysis-day-09-stager-relationships/solve.py`.
- `progress.md` rows for Day 9 (all three modes `Solved`, 2026-10-20).
