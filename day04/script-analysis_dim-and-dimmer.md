# Dim and Dimmer

| | |
|---|---|
| Day / Lane | 4 / Script Analysis |
| Challenge ID | `script-analysis-day-04-shell-contrast` |
| Technique | Static recovery of obfuscated script descriptors (JScript, Batch, VBScript, PowerShell), then structural classification |
| Single Solve | Independent (our own CORRECT response is saved in the leaf) |
| Time Trial | Independent (seventh timed window; later ranked reruns also CORRECT) |
| Mastery | Independent (one submission; later ranked reruns also CORRECT) |

## Summary

The challenge ships 30 inert script specimens, each hiding a small "descriptor" (ordered behavior
steps, adapter, campaign id, indicators) behind a per-language obfuscation. Every answer is either
a value read out of the decoded descriptor or a classification of how the specimen's own source
reconstructs it. Decoding was mechanical; almost all of the lost attempts went to a handful of
Batch and VBScript control-flow classifications where the prompt's wording disagreed with what the
validator accepted.

## The challenge

The prompt says "These VBScript specimens hide their material behind character codes, reversible
transforms, and winding control flow. Unwind every representation on paper, classify each
behavior, and find out which stage passes the baton to which." Two things in that wording and in
the challenge ID (`shell-contrast`) are misleading. The archive is not VBScript-only, and there is
no shell/Bash family. VBScript is just the language new on this day.

- **Artifact:** one AES-encrypted ZIP (`evidence.zip`, password `infected`) with 30 specimens:
  3 PowerShell, 9 JScript, 9 Batch/CMD and 9 VBScript. Nothing is executed.
- **Answers:** 231 string fields keyed by IDs like `batch-stager-control` or
  `vbscript-uploader-transform`. Formats are enums (role, ATT&CK id, expansion style, control
  wrapper, representation, transform) and exact text (URL, campaign id, adapter, file path,
  next-stage name).
- **Time Trial:** same 231 fields, a fresh randomized archive on every retry, 50 s window.
- **Mastery:** 462 fields (two sets of 231) delivered as an outer bundle holding
  `evidence-01.zip` and `evidence-02.zip`, 80 s window.

## Solution

1. **Decode each family's descriptor statically.** The same logical descriptor (`steps`,
   `adapter`, `campaign_id`, `url`, `file_path`, `next_stage`, ...) is stored four ways:
   - PowerShell: a Base64 literal that decodes to JSON.
   - JScript: byte arrays or packed strings, with a reversible transform applied by the recover
     function (key bytes, previous-byte chaining, add/subtract).
   - Batch: numbered `set "itemN=..."` fragments concatenated in source order into a
     `v=1|steps=...|adapter=...` line.
   - VBScript: a `Material_*` string plus a `Codec_*` label of the form
     `representation|transform|key`. Representation was one of hex compact/colon/dash, decimal
     pipe/comma/dash, reversed hex or decimal, Base64; transform one of identity, single-byte XOR,
     repeating-key XOR, add/subtract mod 256, reverse, rolling XOR, reverse-then-XOR.
2. **Read values straight from the descriptor.** Campaign ids, URLs, paths, adapters, wallet and
   mutex indicators, impact extensions and note names, and `next_stage` all come out of the
   decoded text. The VBScript decoder was a direct inversion of the label:

   ```python
   rep, trans, *key = codec.split('|')
   if rep == 'hex_compact':   a = bytes.fromhex(material)
   elif rep == 'decimal_pipe': a = bytes(int(x) for x in material.split('|'))
   elif rep == 'reversed_decimal': a = bytes(int(x) for x in material.split(','))[::-1]
   # ...
   if trans == 'repeating_key_xor':
       b = bytes(x ^ keybytes[i % len(keybytes)] for i, x in enumerate(a))
   elif trans == 'rolling_xor':      # output = stored ^ previous; previous = stored
       prev = k; out = []
       for x in a: out.append(x ^ prev); prev = x
       b = bytes(out)
   ```

3. **Derive role and ATT&CK from the ordered `steps`.** For example `acquire_remote,
   dispatch_memory` is a `stager` (`T1105`), a `write_file` decode step is a `dropper` (`T1140`),
   exfiltration is an `uploader` (`T1041`), and a chain ending in a terminal dispatch plus a
   handoff is `staged_chain`. The validator accepted the bare parent `T1059` for launcher and
   staged-chain graphs ending in a process dispatch, not a language sub-technique. The three
   PowerShell samples map directly: resource hijacking `T1496`, browser credential store
   `T1555.003`, synthetic impact `T1486`.
4. **Classify style and wrapper fields from the source structure** (JScript function, invocation
   and property style; Batch expansion form and control wrapper; VBScript decode-call wrapper).
   This is where the real work and all of the rejected submissions were.
5. **Submit all 231 answers in one call.** The Single Solve's first broad submission came back
   `incorrect`. Splitting into bounded groups showed 230 of 231 fields accepted, so the problem was
   confined to the Batch control fields (see below). After correcting `batch-stager-control` and
   `batch-impact-control` to `call_subroutine`, the submission returned CORRECT.

## Time Trial and Mastery

**Time Trial** took seven timed windows (the first expired before a submission, six retries
followed). Each retry delivered a freshly randomized archive that exercised a decoder branch or a
classification the previous one had not, so each window ended with a different offline-fixable
problem:

| Window | What went wrong |
|---|---|
| 1 | Parser gaps in randomized representations; expired unsubmitted |
| 2 | `incorrect`: a Batch assignment `item9=%item0%%item1%%item2%%item3%` (an assembled variable, not a fragment) was appended to `next_stage` |
| 3 | `incorrect`: JScript material stored as a reversed numeric array followed by repeating-key XOR had no decoder; then expired while splitting into lanes |
| 4 | A short four-byte numeric array (a distractor) was picked instead of the long colon-hex `packedText`; then four control fields were wrong (`direct` submitted; needed `for_once`, `if_guard`, `for_once`, `for_once`); expired |
| 5 | Two Batch controls submitted as `for_once`/`call_subroutine`; source has `IF ERRORLEVEL 1 EXIT /B 1` before the join, so `errorlevel_guard`; expired |
| 6 | Three Batch controls wrong; the `IF DEFINED ... SET cfg=` guard means `if_guard`, a bare multi-item FOR means `for_once`; expired |
| 7 | 230 of 231 accepted; the last field (`batch-launcher-control`) sits inside an outer `CALL :HUNTRESS_RECOVER` wrapper, so `call_subroutine`. CORRECT before the 50 s deadline |

**Mastery** refused to start (`prerequisite_required`) until the Time Trial was CORRECT. Once it
was, the bundle was started, the outer hash checked, both inner archives decrypted, and all 462
IDs mapped in one pass. It was accepted in about 32 s of the 80 s window on a single submission,
because the Single Solve and Time Trial fixes had already settled every classification.

Later ranked reruns of the sweep (two for Time Trial, two for Mastery) went CORRECT in one
submission each, with end-to-end times of roughly 2 s, using a consolidated solver built from the
same decoders.

## Dead ends and gotchas

- **The prompt's Batch `control` hint is wrong in practice.** It says a one-item `FOR` loop that
  calls a subroutine should be `for_once` rather than `call_subroutine`. The validator rejected
  `for_once` in the Single Solve for the stager, impact and the FOR-wrapped downloader and accepted
  `call_subroutine`. Reading the description did not predict this; only the validator did.
- **Batch controls are classified by structure, not by intuition.** What was eventually accepted:
  one-item `for /l ... do call :LABEL` is `call_subroutine`; `IF ERRORLEVEL 1 EXIT /B 1` before a
  multi-item `FOR ... CALL SET` join is `errorlevel_guard`; `if defined X set "cfg="` before the
  join is `if_guard`; a bare multi-item FOR join after a plain `set` is `direct`; an outer
  `CALL :HUNTRESS_RECOVER` around the FOR is `call_subroutine`. Guessing these one 50 s window at a
  time cost windows 4 to 6.
- **Assembled variables are not fragments.** Batch descriptors are built from numbered `itemN`
  assignments, and one specimen also assembled `itemN` from other variables. Treating that as
  another literal fragment appended `%item0%...` text to `next_stage`.
- **Distractor arrays and operation order.** A four-byte numeric array can sit next to the real
  material, and reverse versus XOR must be replayed in statement order, not inferred.
- **VBScript `control` has no `if_guard`.** The enum is `direct`, `error_guard`, `for_once`,
  `do_once`, `while_once`, `select_case`. A one-pass `For ... Next` wrapper is `for_once`; sending
  `direct` for it was rejected.
- **`incorrect` carries no per-field feedback.** Narrowing came from submitting bounded groups and
  reading which IDs were reported accepted. During a 50 s window that is expensive.
- **ATT&CK primary.** The bare parent `T1059` was accepted for terminal-dispatch graphs; the
  language-specific sub-technique was not what our accepted maps carried on this day.
- **Local copies of the early ZIPs may be damaged** (CRLF translation); re-check the hash from the
  start response.
- Fixture and window-timing mistakes in our own handling of the early retries are collected in the
  day overview.

## Takeaways

- Treat the specimen's own reconstruction structure as ground truth for wrapper classifications;
  do not trust prose hints in the question over accepted maps.
- Build a regression corpus from every specimen variant seen so far plus the accepted answer maps,
  and run new decoders against it before each timed window. Seven windows of one-bug-each is the
  cost of not doing so (variant-corpus regression).
- Prefer a single statement-order replay (static emulation of the `set` / `Material_` / recover
  logic) over regexes keyed on one fixture's shape.
- Once a mapper is validator-confirmed on the Single Solve and Time Trial, Mastery is a mechanical
  one-submission pass.

## Evidence

Leaf base path: `challenges/day04/script-analysis/script-analysis-day-04-shell-contrast/`

- `single-solve/journal.md`, `single-solve/artifacts/derive_static.py`,
  `single-solve/artifacts/solve_from_bytes.py`, `single-solve/artifacts/decoded-descriptors.json`
- `single-solve/artifacts/submissions/fix-batch-impact-control.json` (the CORRECT response)
- `time-trial/journal.md`, `time-trial/attempt-06/artifacts/batch-launcher-correction-response.json`
  (CORRECT), `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/attempt-01/artifacts/solve_mastery_from_bytes.py`,
  `mastery/attempt-01/artifacts/submission-response.json` (CORRECT),
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- Consolidated solver used for the ranked reruns: `tools/ranked/solvers/script-analysis-day-04-shell-contrast/solve.py`
- `progress.md` rows for Day 4 `script-analysis-day-04-shell-contrast` (all three modes Solved, 2026-10-18)
