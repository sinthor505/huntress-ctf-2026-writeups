# Expansion Pack

| | |
|---|---|
| Day / Lane | 3 / Script Analysis |
| Challenge ID | `script-analysis-day-03-batch-expansion` |
| Technique | Static deobfuscation of generated PowerShell / JScript / Batch specimens; cmd.exe expansion and control-flow classification |
| Single Solve | Solved locally; graded result not preserved (see Evidence) |
| Time Trial | Independent (ranked pass `CORRECT`, 1 submission) |
| Mastery | Independent (`CORRECT`, 1 submission on the re-solve; ranked pass `CORRECT` in 2) |

## Summary

Each specimen hides a small descriptor (steps, adapter, campaign ID, URL or path) behind a
different obfuscation layer, and the questions ask for the descriptor values plus a few
style and structure classifications. The descriptor values are mechanical to recover. The
hard part was the Batch `*-control` classification, where the prompt's rule disagreed with
what the validator accepted, and two JScript style fields that a first-pass regex got wrong.
Once those were fixed, every mode was accepted in one or two submissions.

## The challenge

The prompt reads: "cmd.exe expands percents, delays expansion, CALLs twice, and bends FOR
loops into indirection. Reconstruct every variable exactly as the interpreter would see it,
without executing a single line, then follow the staged chain to its final handoff."

The delivery is an AES-encrypted ZIP (password `infected`) of 21 inert script samples:

- 3 PowerShell scenario samples (`resource-sample.ps1`, `collection-sample.ps1`,
  `impact-sample.ps1`);
- 9 JScript files (`host-context.js`, `memory-stage.js`, `embedded-write.js`,
  `remote-write.js`, `local-dispatch.js`, `staged-handoff.js`, `outbound-transfer.js`,
  `impact-plan.js`, `relay-handoff.js`);
- 9 Batch files with the same nine roles (`.cmd`/`.bat`).

Single Solve and Time Trial asked 159 questions. Mastery delivered two such sets (318 fields).
Questions fall into these groups:

- **Role / ATT&CK:** `*-role` and `*-attack`. A long rubric maps an ordered step graph such as
  `acquire_remote, write_file, dispatch_process, handoff_script` to a role (`staged_chain`)
  and a technique ID.
- **Indicators:** adapter, campaign ID, URL, file path, next stage, wallet, mutex, user agent.
- **Structure:** `graph` (stage topology) for Batch; function, invocation and property style
  for JScript; `expansion` and `control` for Batch.
- **PowerShell scenarios:** a scenario enum and ATT&CK ID per sample.

## Solution

1. **Decrypt in memory and parse as data.** The solver read the ZIP with `pyzipper` and never
   evaluated any specimen code.
2. **PowerShell:** each sample holds a `FromBase64String('...')` literal whose bytes are a
   JSON object with one-letter keys. The solver pulled the indicators out of it. Examples are
   `b`=URL and `c`=wallet for the resource sample, and `d`=URL, `b`=extension and `c`=note name
   for the impact sample. The scenario enum and ATT&CK ID are fixed per file (resource
   hijacking `T1496`, collection `T1555.003`, impact `T1486`).
3. **JScript:** `recover_*()` rebuilds a byte array from a literal and a pure `decode_*`
   function. The literal is one of several encodings: decimal, `%hh`, `hh:hh`, Base64, or
   split arrays joined with `concat`. The decoder is some mix of reverse, fixed XOR, key-array
   XOR or rolling XOR. The solver reimplemented those transforms as integer-array operations
   and parsed the resulting JSON (`steps`, `adapter`, `values`, `stageCount`). It did not
   `eval` anything.
4. **Batch:** the fixtures carry the descriptor twice. One copy is `set "fragmentN=..."`
   lines joined by a `call set` or `FOR ... CALL SET` statement. The other is a
   `rem HUNTRESS-CODEC:` / `rem HUNTRESS-MATERIAL:` marker pair, encoded with
   `plain_fragments`, `decimal_bytes`, `hex_pairs` or `reversed_text` plus a small transform.
   The solver decoded the marker and parsed the `v=1|steps=...|adapter=...|campaign_id=...`
   string. For example, `staged-handoff.bat` yields `acquire_remote,write_file,dispatch_process,handoff_script`,
   adapter `cscript.exe`, and `graph=cross_language`.
5. **Role and ATT&CK** came from the step list using the prompt's precedence. A terminal
   `handoff_script` or `dispatch_next_stage` makes the language sub-technique win (`T1059.007`
   for JScript, `T1059.003` for Batch). Otherwise `acquire_remote` gives `T1105`,
   `decode_embedded` gives `T1140`, upload steps give `T1041`, `dispatch_process` gives the
   sub-technique, and the benign decode/return graph gives `none`.
6. **`expansion` and `control`** were classified from the one reconstruction statement, not
   from the whole file. The generated fixtures wrap noise, decoy `goto` jumps and an unused
   `HUNTRESS_UNUSED` trailer around it. Our final classifier works on structure:

   ```python
   # mastery/artifacts/answer_map.py: signature -> answer the validator accepted
   SIG_TO_CONTROL = {
       "err": "errorlevel_guard", "goto": "goto_dispatch", "if_same": "if_guard",
       "if_init": "if_guard", "F": "call_subroutine",   # not for_once, see below
       "C": "call_subroutine", "direct": "direct",
   }
   ```

7. **Submit** all string answers keyed by the live `answer_ids`. A coverage check asserted
   that the mapper's key set equals the start response's IDs before any submission. The
   JScript style classification and the Batch controls are the two places where care mattered
   (next sections).

## Time Trial and Mastery

- **Time Trial** was the same 21-file bank with fresh values and the same 159 IDs. The
  Single Solve solver ran unchanged against the newly assigned archive. Windows were short:
  the journal records an 8 s window on an early attempt, and the later ranked pass had 50 s.
  The ranked pass took 1.4 s end to end (issue, download, solve, submit) with one
  submission.
- **Mastery** had two sets (`set1.*`, `set2.*`; 318 fields) under an 80 s window. It was
  delivered as two separate downloads (`artifacts[0..1]`, both named `evidence.zip`), not
  one bundle, and the Game Master `artifact` op fetches only the first. The driver had to
  read `artifacts[]` from the start response and hash each file.
- Batch controls were classified per file by structural signature, with no per-ID
  overrides, because the structure differs per set. In the re-solve, set 1 had `direct`,
  `F`, `C`, `err` and `goto` members, and set 2 added `if_same`. The full 318-field map was
  accepted in one call.
- In the ranked Mastery pass, 317 fields were accepted in the base call (`remaining=1`), and
  one set-1 control field with an `if_init` signature went out as its own group and was
  accepted on its first variant. That field was held back by the submission strategy, so
  the base call carried only trusted signatures. The exact file was not recorded in the leaf.

## Dead ends and gotchas

- **`for_once` versus `call_subroutine`.** The control prompt says "If a one-item FOR loop
  invokes a subroutine containing that statement, classify the outer wrapper as `for_once`".
  The first Single Solve and Time Trial maps followed that. The accepted maps used
  `call_subroutine` for the one-item `for /l %%Q in (1,1,1) do call :LABEL` shape. The
  journal records that the validator accepted the structurally derived map outright. One
  early Time Trial window had three Batch groups (dropper, uploader, relay) come back
  `incorrect`. The dropper is plausibly this `for_once` case. The uploader and relay causes
  were not diagnosed before the window closed.
- **Errorlevel guard misread as `direct`.** The old regex classifier looked for
  `if errorlevel` on the assignment line itself. In these fixtures the guard is a separate
  `ver >nul` plus `if errorlevel 1 exit /b 1` line before the `call set` join, so a launcher
  came out as `direct`. That should have been `errorlevel_guard`.
- **`call set` is not a subroutine.** A bare `call set "v=%%a%%%%b%%"` is CALL re-parsing,
  so `expansion` is `call_double` and `control` is `direct`, not `call_subroutine`. In
  `staged-handoff.bat` the statement `call set "recovered_configuration=%%fragment0%%..."`
  has `ver`/`if errorlevel 1 exit /b 1` immediately above it.
- **JScript `function-style`.** `var inspect_x = function inspect_x_body() {...}` is a
  `named_expression`. A regex that looked for `function inspect_...` first called it a
  `declaration`. Five fields in Single Solve and Time Trial were changed by fixing this
  (the old maps are not ground truth).
- **JScript `property-style`** counts accesses only on the variable assigned from
  `eval(recover_*())`. The fixtures also read properties on other objects, such as
  `v_e598["steps"] = v_d41a["steps"]`, and the question says to ignore those.
- **Stale candidates are not ground truth.** The first-pass Mastery candidate file came from
  bytes of an attempt that expired unsubmitted. The accepted map was derived from the bytes
  of the attempt that actually went `CORRECT`.
- **Local ZIP copies were damaged or stale** in several leaves (bad ZIP header, hash
  mismatches), so analysis used the extracted samples and, for Mastery, a fresh download
  with SHA-256 verification. See the day overview for our own tooling slips.

## Takeaways

- Classify a Batch fixture from the **execution trace or structure around the single
  reconstruction statement**, not from the prompt's prose or from regexes on one line. The
  technique is a static cmd.exe variable-expansion emulator (`%v:~a,b%`, `%v:old=new%`,
  `call set` second pass, `for` indirection) that also reports the wrapper frames.
- Treat a prompt rule that has the validator's feedback against it (here `for_once`) as a
  hypothesis. Submit by signature group, so a rejection tells you which structure was wrong.
- The descriptor is stored twice in the generated fixtures (fragments and the
  `HUNTRESS-MATERIAL` marker). Decoding both and comparing is a cheap check that the
  fragment join is right.
- With windows of 8 to 80 s, run the mapper offline on every saved variant first, and make
  one scripted pass (start, download, map, submit).

## Evidence

Leaf: `challenges/day03/script-analysis/script-analysis-day-03-batch-expansion/`

- Single Solve: `single-solve/journal.md`, `single-solve/artifacts/solve_expansion.py` (the
  decoder), `single-solve/artifacts/candidates.json`. There is no `result.json`. The journal
  states the Game Master already reported the challenge `CORRECT` from a prior attempt and
  that no submission was made in the recorded workflow. `progress.md` marks it
  "Player-directed local mark". We therefore make no claim of a graded solve for this mode.
- Time Trial: `time-trial/ranked-1/result.json` (`CORRECT`, 159 fields, 1 call, 1.43 s total),
  `time-trial/ranked-1/pass.log`, `time-trial/journal.md`, and the nested earlier attempt
  under `time-trial/time-trial/` (three groups `incorrect`).
- Mastery: `mastery/journal.md` (re-solve section), `mastery/ranked-1/result.json` (`CORRECT`,
  318 fields, 2 calls), `mastery/artifacts/answer_map.py`, `static_decode.py`,
  `batch_control_signature.py`, `control-signatures.json`.
- Library techniques: Batch control classification by structure signature; static Batch
  variable-expansion emulator; variant-corpus regression; Days 1-4 script-analysis gotchas.
