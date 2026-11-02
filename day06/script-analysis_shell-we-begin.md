# Shell We Begin

| | |
|---|---|
| Day / Lane | 6 / Script Analysis |
| Challenge ID | `script-analysis-day-06-bash-behavior-graphs` |
| Technique | Static reconstruction of encoded behavior descriptors across six script languages; graph-to-role/ATT&CK mapping |
| Single Solve | Independent (399/399 fields, final `CORRECT` after 68 submissions) |
| Time Trial | Independent (`CORRECT` on 1 call in a later ranked pass; the first three timed attempts on the original play-through never reached `CORRECT`) |
| Mastery | Independent (`CORRECT` on 1 call, twice, in later ranked passes; no graded response from the original play-through) |

## Summary

The title and prompt promise "six Bash behavior graphs", but the 51-file encrypted bundle holds
PowerShell, JScript, Batch, VBScript, Python and Bash specimens, and all 399 answer fields have to be
filled. Each specimen hides a small descriptor (steps, adapter, campaign id, one indicator, and
language-specific style labels) behind a reversible encoding. We decoded every descriptor statically
and mapped it to the rubric. Most of the work was learning, by submission, where the validator's
accepted spelling differs from the prompt's wording (Batch one-item `FOR` controls and a few
ATT&CK ids).

## The challenge

Prompt: "Six Bash behavior graphs, none of which will ever run. Read them like a native: map the
adapters, count the stages, tag the ATT&CK techniques, and pull the exact indicators hiding in the
final bytes."

Single Solve delivered `evidence.zip` (AES-encrypted ZIP, 51 members, password `infected`) and 399
answer IDs. The fields break down as:

| Family | Specimens | Fields |
|---|---|---|
| PowerShell | 3 | 15 |
| JScript | 9 | 72 |
| Batch | 9 | 72 (core 54, expansion 9, control 9) |
| VBScript | 9 | 72 (incl. representation, transform, control) |
| Python | 14 | 126 (incl. syntax, obfuscation, carrier, windows-chain) |
| Bash | 6 | 42 (role, attack, adapter, campaign, indicator, `stage-count`, `steps`) |

Answer formats are enums, MITRE ids, exact text, an integer, and `ordered_tokens` (`a>b>c`). Time
Trial was the same 399 fields on a 50 s server clock; Mastery was two such sets (798 IDs prefixed
`set1.`/`set2.`) on an 80 s clock.

## Solution

1. Opened the ZIP with `infected` (members use AES, so the stdlib `zipfile` is not enough; we used
   `pyzipper`). Nothing was executed. The Bash files are inert: they gate execution behind an
   inspection variable and only exist to be read.
2. Recovered each descriptor from its literal material. The encodings differed by family:
   PowerShell used Base64 JSON; JScript used byte arrays, colon/percent hex or Base64 plus
   XOR/add/reverse steps; Batch split the descriptor into indexed fragments that concatenate; VBScript
   declared `Material` and `Codec` constants; Python declared `MATERIAL_*`/`CODEC_*`; Bash used decimal,
   Base64 and percent-hex strings. The decoded Bash descriptors are `key=value|...` records, for
   example `steps=profile_host,acquire_remote,write_file,change_mode,dispatch_process`,
   `adapter=wget_arch_fallback`, `stage_count=5`, `cron_path=...`.
3. Read `steps` from the decoded record (never from the file name) and applied the prompt's rubric:
   role from the terminal intent (`acquire_remote` then `write_file` is `downloader`, `locate_local`
   then `dispatch_process` is `launcher`, a multi-stage profile/acquire/write/dispatch is
   `staged_chain`), and ATT&CK from the same graph (`acquire_remote` -> `T1105`, `decode_embedded` ->
   `T1140`, upload -> `T1041`, `dispatch_process` -> the language sub-technique such as `T1059.004`).
4. Took indicators (URL, wallet, mutex, file path, extension, note name, cron path) verbatim from the
   decoded record, and labels such as `stage_count` and `steps` from the same record.
5. Classified Batch expansion and control wrappers, and VBScript `Recover_*` wrappers, from the
   statement that reconstructs the descriptor and the construct enclosing it.
6. Submitted by family, then by specimen, then by field, because an `incorrect` reply carries no
   per-field detail (see below). The final accepted map is the 399-value `candidate-answers.json` in
   the Single Solve leaf.

## Time Trial and Mastery

Both modes have `prerequisite_required` locks (Time Trial after Single Solve, Mastery after Time
Trial), so the first start responses carried generic questions with no answer IDs or artifact.

Time Trial: the original play-through used three timed attempts of 50 s each (399 fields, a fresh
51-file ZIP per attempt). The first mapper had accidentally been built from the Single Solve bytes
and was rejected. For the third attempt, grouped diagnostics showed 307 of 399 fields accepted
(PowerShell, JScript, Python, Bash and the Batch/VBScript specimens that happened to match); the rest were Batch expansion/control
and VBScript control values. A source review of that fixture produced a locally audited full map, but
no graded `CORRECT` response is saved for those attempts, and we do not count them as the solve. The
Time Trial was later re-run in a ranked pass with a scripted solver and returned `CORRECT` on the
first submission (2.1 s from issue to graded result, inside the 50 s window).

Mastery: the original play-through only prepared an offline two-set pipeline; no assigned bundle or
graded response from that period is saved. Two later ranked passes (80 s window, two 399-field sets)
each returned `CORRECT` on one 798-field submission (2.6 s and 4.7 s total).

What changed per fixture: file names recur but structures do not. The Single Solve and the Time
Trial retry diagnostics disagreed on which Batch and VBScript specimens needed which expansion or
control value (the retry's corrections included `goto_dispatch`, `if_guard`, `delayed`,
`while_once` and `direct`), so Batch controls and expansions have to be classified from each
delivery's own source, with no per-filename tables.

## Dead ends and gotchas

- **Batch one-item `FOR` control.** The prompt says: "If a one-item FOR loop invokes a subroutine
  containing that statement, classify the outer wrapper as `for_once` rather than `call_subroutine`."
  The validator disagreed in Single Solve. Singleton `for_once` submissions for four such controls
  were rejected, and `call_subroutine` was accepted for all four. A fifth (`embedded-write.cmd`,
  which joins four fragments in a direct `for ... do call set` loop with no one-item wrapper) was
  accepted as `direct`. We do not know why the wording and validator differ; the rule that held in
  later modes was to classify structurally and not to trust the prompt text for this one field.
- **ATT&CK for Python chains.** Two Python specimens (`py2exe-service`, `pyarmor-pyinstaller`)
  whose graph ends in a handoff/`dispatch_process` were rejected with `T1105` and accepted with
  `T1059.006`, the language sub-technique, which is what the prompt's precedence rule says. In the
  same Single Solve, the Bash `staged_chain` (`profile_host,...,dispatch_process`) was accepted as
  `T1105`. The precedence text does not explain that difference, so we treated validator-accepted
  values as the rubric and recorded the per-language split.
- **`incorrect` is batch-level only.** The reply names no field, so a 198-field VBScript/Python
  submission that failed could only be narrowed by resubmitting per specimen (23 calls), then per
  field (18 calls for the two bad Python specimens), which is how the two attack ids were found.
  That is why Single Solve took 68 submissions (the player extended the cap; two were unused).
- **Literal field, not inference.** Indicators (URL, path, upload URL) are the decoded field exactly
  as stored. Forward slashes in Python `file_path` values stand as written, and an upload URL is
  the decoded value rather than a derived `.../upload`.
- **Order of transforms.** Reversal in the enclosing call, reversal in the decoder and XOR do not
  commute; follow the source order. Bash `Material='a''b'` is a single word, so adjacent quoted
  segments are joined before decoding.
- **Per-attempt fixtures.** Time Trial and Mastery assigned a different ZIP on every retry; a map
  built from another attempt's bytes fails. (Our own early Time Trial mistake of reusing the Single
  Solve map: see the day overview.)

## Takeaways

- Decode first, classify second: every field came from a descriptor recovered from literal bytes, with
  role/ATT&CK derived from `steps`. This is the "reconstruct encoded behavior descriptors" technique.
- Classify Batch/VBScript control wrappers structurally, per delivery, and expect the validator's
  enum to differ from the prompt's wording.
- When a grouped submission returns only `incorrect`, bisect by family, specimen, then field before
  changing any value, and keep each probe small enough to read.
- With a stable decoder and a validator-confirmed rubric, the Time Trial and Mastery maps were
  one-call, sub-5-second jobs.

## Evidence

Leaf base: `challenges/day06/script-analysis/script-analysis-day-06-bash-behavior-graphs/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/candidate-answers.json`,
  `single-solve/artifacts/static_decode.py`, `single-solve/artifacts/decoded-descriptors.json`,
  `single-solve/submission-request-NN-*.json` / `submission-response-NN-*.json` (68 attempts; the final
  response, attempt 68, is `CORRECT`).
- `time-trial/journal.md`, `time-trial/retry-02-language-results.txt`,
  `time-trial/retry-02-specimen-results.txt`, `time-trial/retry-02-field-results.txt`
  (the 307/399 diagnostics), `time-trial/ranked-1/result.json` (`CORRECT`).
- `mastery/journal.md`, `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json` (both `CORRECT`).
- Solver for the later passes: `tools/ranked/solvers/script-analysis-day-06-bash-behavior-graphs/solve.py`
  (shared bank code in `tools/ranked/solvers/_shared/`).
- Library: `library/script-analysis/descriptor-reconstruction.md`, `material_codec_decode.py`,
  `behavior_rubric.py`, `gotchas-d05-d09.md`.
