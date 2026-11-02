# By Any Other Name

| | |
|---|---|
| Day / Lane | 11 / Script Analysis |
| Challenge ID | `script-analysis-day-11-native-api-semantics` |
| Technique | static descriptor recovery from 95 inert script specimens; matching a validator's ATT&CK and Batch-control rubric |
| Single Solve | Solved (journal-recorded): the journal records CORRECT on the second 20-attempt authorization; no result file was saved |
| Time Trial | Independent: original CORRECT is journal-recorded; later ranked reruns each hold a `result.json` reading CORRECT |
| Mastery | Independent: original CORRECT is journal-recorded; later ranked reruns each hold a `result.json` reading CORRECT |

## Summary

The prompt talks about native API calls and a "claimed purpose the evidence doesn't support", but the
811 questions were the same generated bank of script-specimen fields used on Day 10, plus 58 new ids
(TypeScript polyglots, among others). Recovering the fields was a mechanical static decode; the cost
was in the validator's rubric. Two rules decided the outcome: Python staged chains take a
language-specific ATT&CK sub-technique, and a one-item `FOR /L ... DO CALL` Batch wrapper is not
reliably `for_once`. All three modes ended CORRECT.

## The challenge

Prompt: "These scripts reach for native APIs by name, but a name isn't a behavior. Map each reference
to the receipt-backed action it models, and flag the call whose claimed purpose the evidence doesn't
support." No receipt file or "mismatched call" answer exists in the questions; the journal notes this
discrepancy and the questions are what counted.

Delivery:
- Single Solve: one AES-encrypted ZIP (password `infected`) of 95 specimens across PowerShell,
  JScript/JavaScript, Batch, VBScript, Python, Bash, Node.js, PHP, Lua, TypeScript and WMI MOF,
  untimed. 811 answer ids.
- Time Trial: the same 811 ids over a fresh archive (new SHA-256, same ids), server-enforced window of
  50 s.
- Mastery: a bundle of two full copies of the challenge (`set1.*`, `set2.*`, 1622 ids, one
  `evidence-NN.zip` per set), window of 80 s.

Id suffixes in Single Solve, from the start response: `role` (91), `attack` (94), `adapter` (81),
`steps` (50), `style` (27), `control` (18), and `expansion`/`graph`/`representation`/`transform` (9
each), plus about 414 exact-text indicator and enum fields (URLs, wallet, mutex, campaign id, profile
names). Every answer is a string.

## Solution

1. **Inventory.** The archive opened offline with `pyzipper` (stdlib `zipfile` cannot read the AES
   members). Nothing was executed.
2. **Descriptors are data.** Each specimen carries an inert hex-encoded descriptor, for example a
   `HUNTRESS_ANALYSIS_V1:HEX-...` comment and a descriptor variable holding the same hex. It decodes
   to a `|`-separated record: `language`, `role`, `verdict`, `attack`, `adapter`, `steps`,
   `campaign_id`, `url`, `file_path`, `next_language`, and so on. Older specimens (Python, Bash,
   Batch, VBScript) use per-language transform chains (`identity`, XOR, rolling XOR, add/subtract
   modulo 256, reverse) which the decoder reimplemented statically. `static_decode.py` decoded all
   95 specimens with no errors.
3. **Direct fields.** Indicators, profiles, stage counts, step lists (joined with `>`) and
   next-language fields are read straight from the decoded record. Role and adapter come from the
   descriptor's own fields, not from tokens in the question's `--plan` text.
4. **Rubric fields.** `attack`, Batch `*-control`, VBScript control/representation/transform and
   JScript style fields are not in the descriptor and need a classification of the source.
   Those were the hard part (next two sections).
5. **Submission.** Group by family, find which groups the validator accepts, and fix the rest.

The accepted Single Solve map differs from the first guess in a small number of fields; the final
rule set (also baked into the later per-mode solver) is:

```python
# staged chain: profile_host, acquire_remote, decode_embedded, write_file, dispatch_process
if family == "python":      attack = "T1059.006"   # NOT T1105
if family in ("bash", "nodejs"): attack = "T1105"   # ingress accepted on Day 11
if descriptor_has("attack"): attack = descriptor["attack"]   # v3 descriptor claim wins
```

## Time Trial and Mastery

- **Same engine, new constraints.** The ids and rubric are the same, but windows were 50 s (Time Trial)
  and 80 s (Mastery). A retry issues a fresh variant (new archive hash), and an expired attempt
  accepts nothing, so everything (group files, id lists, descriptor fixes) was precomputed before
  `retry` and the window ran as one scripted pass: retry, download, decode, map, grouped submits.
- **Time Trial** needed 9 submissions in total: four were wasted on an expired window (building lists
  inside the window), one full-map attempt was `incorrect`, then groups A-E (solid fields, role and
  adapter, attack, control, expansion/style/graph/representation/transform) were accepted in one
  window.
- **Mastery** needed 26 submissions against a raised cap of 38 across several windows. Set 1 Batch
  controls were largely known forms; set 2 carried four controls that stayed unconfirmed
  (`errorlevel_guard`, `if_guard`, a one-item `FOR /L` launcher, `goto_dispatch`). An Opus
  review ranked the one-item `FOR /L` launcher as the likeliest wrong answer and proposed
  `call_subroutine`; in the next window all three one-item `FOR /L` controls (set 1 dropper, set 2
  chain, set 2 launcher) were accepted as `call_subroutine` and the mode went CORRECT.
- **Later ranked reruns** (Time Trial on 2026-10-26 and 2026-11-01; Mastery on 2026-10-26 and twice on 2026-11-01) used a per-challenge solver that
  derives everything from the delivered bytes plus the start response. Four of the five reruns
  went CORRECT in one call. The 2026-10-26 Time Trial rerun took 12 calls: its fresh fixture rejected
  `for_once` and accepted `call_subroutine`, so `call_subroutine` became the primary for every mode
  and `for_once` stayed as a group alternate (Single Solve alone had accepted `for_once`).

## Dead ends and gotchas

- **Graph reading vs. declared field.** The question text prints a graph precedence ("terminal
  `handoff_script`... `dispatch_process` is the language-specific Command and Scripting Interpreter
  sub-technique..."). We first derived `attack` from that graph and ignored the descriptor's own
  `attack` claim, deliberately, because the two disagreed on 22 v3/host fields. The validator
  rejected the graph values: for the four PHP fields tested alone, the descriptor-claim
  alternatives were accepted and the graph values were not, and a later 18-field group of
  descriptor claims was accepted whole. The descriptor claim wins where one exists.
- **The seven `dispatch_language` fields** (PHP, Lua, WMI MOF, TypeScript, HTA/JScript polyglots) were
  the ones the question's precedence text does not name. We guessed the source language's
  interpreter technique; this turned out moot because the declared `attack` wins for all of them.
- **Python staged chains were the real 20-attempt sink.** The first Single Solve authorization used
  all 20 submissions: 685 of 811 ids were confirmed by family, and the remaining 126 Python ids
  kept returning `incorrect`. The mapper checked `acquire_remote` before `dispatch_process`, so the
  two Python staged chains (`py2exe-service`, `pyarmor-pyinstaller`) came out as `T1105`; the
  validator wanted `T1059.006`. We found it by diffing against the accepted Day 5 Python carrier
  lab (same 14 Python specimens): only those two `attack` values differed in kind. The second
  authorization went CORRECT on attempt 1.
- **Do not generalize that rule across languages.** Bash and Node.js staged chains of the same
  shape were accepted as `T1105` on Day 11, in every mode, because the question text allows the
  ingress transfer there.
- **One-item Batch `FOR /L` wrappers.** The question says "classify the outer wrapper as `for_once`
  rather than `call_subroutine`". Single Solve and the first Time Trial accepted `for_once`;
  Mastery and the fresh 2026-10-26 Time Trial wanted `call_subroutine`. The expected value is not
  determined by the question text, and varies with the assignment. Probe these controls alone
  with `call_subroutine` first and `for_once` as the fallback.
- **Batch controls are the usual suspect.** A full map that is rejected with no field-level feedback
  almost always has a rubric-sensitive Batch `*-control` inside it. Submitting everything except
  controls first, then controls by group, localizes it cheaply.
- **Expired windows still count.** Time Trial lost four submissions to building lists inside a 50 s
  window.
- **Python `report` vs. descriptor adapter.** `python-source-benign-adapter` took the descriptor's
  `argparse_namespace`; the plan token `report` was never needed.

## Takeaways

- Decode first, then read only the descriptor; do not trust the prompt's thematic framing ("native
  APIs", "receipts"), which here described nothing in the questions.
- For a challenge whose family was solved on an earlier day, diff the new candidate map against the
  earlier accepted map (ignoring campaign hashes) before spending submissions. That diff found the
  Python staged-chain exception that 20 submissions had not.
- Treat rubric fields by family and shape, not by the prompt's printed rule: a descriptor's own
  `attack` field beats the printed graph precedence, and Python staged chains are a documented
  exception to the ingress-transfer reading.
- For timed windows, precompute every group and id list, and submit the large non-control block in
  one call.
- Searchable terms: descriptor decode, `HUNTRESS_ANALYSIS_V1`, staged chain ATT&CK, one-item
  `FOR /L` Batch control.

## Evidence

Leaf: `challenges/day11/script-analysis/script-analysis-day-11-native-api-semantics/`

- `single-solve/journal.md`, `single-solve/artifacts/static_decode.py`, `answer_map.py`,
  `py_map_v2.json` (corrected Python map), `submission-attempt-log.json` (the first 20 rounds)
- `time-trial/journal.md`, `time-trial/artifacts/fix_map.py`, `run_window.sh`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/artifacts/prep_sets.py`, `adaptive.py`,
  `mastery/ranked-1/`, `ranked-2/`, `ranked-3/` (each `result.json` CORRECT)
- Per-challenge solver: `tools/ranked/solvers/script-analysis-day-11-native-api-semantics/solve.py`
  with `tools/ranked/solvers/_shared/bank_d11.py`
- `progress.md` rows for Day 11 (all three modes marked Solved on 2026-10-20)

Note: the original Time Trial and Mastery submission responses were not saved; those CORRECTs rest
on the journals. The ranked reruns are the saved graded evidence for those two modes.
