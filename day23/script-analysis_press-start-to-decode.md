# Press Start to Decode

| | |
|---|---|
| Day / Lane | 23 / Script Analysis |
| Challenge ID | `script-analysis-day-23-game-resource-loader` |
| Technique | offline decoding of inert behaviour descriptors (`HUNTRESS_ANALYSIS_V1`), new GDScript family, layered `LYR` markers |
| Single Solve | Solved (journal-recorded): journal and attempt log show CORRECT on call 1; no `result.json` was saved |
| Time Trial | Independent: ranked rerun `result.json` and `accepted_plan.json` are CORRECT; the original untimed attempt is journal-recorded (CORRECT, call 1) |
| Mastery | Independent: ranked rerun `result.json` and `accepted_plan.json` are CORRECT; the original untimed attempt is journal-recorded (CORRECT, 10 calls) |

## Summary

The prompt dresses a large "indicator recovery" bank up as a GDScript resource loader. The real work
is decoding one inert descriptor embedded in each specimen and answering 1,461 to 10,465 questions
from it. The mapper already existed from Days 10-22, so Single Solve passed on the first call. Time
Trial added a new layered encoding (`LYR`) and a different question schema. Mastery exposed a rubric
bug that no earlier mode could show: a staged chain that ends in `dispatch_process` takes the
language sub-technique, not `T1105`.

## The challenge

Prompt: "A GDScript resource loader builds its strings in layers, one power-up at a time. Undo the
construction, recover the exact indicators, and correlate its stage relationships with the rest of
the corpus to find the level that doesn't belong."

Delivery was a static `evidence.zip` of script specimens:

| Mode | Questions | Delivery |
|---|---|---|
| Single Solve | 1,461 | one zip, named question IDs (`polyglot-gdscript-staged-chain-role`) |
| Time Trial | 1,495 | one zip (157 members), hash-style IDs (`polyglot-gdscript-a2bd79-role`) |
| Mastery | 10,465 | an outer zip of 7 AES-encrypted `evidence-NN.zip` sets (password `infected`), IDs prefixed `setN.`, 1,495 per set |

Answer kinds were mixed: `exact_text` (the largest group), `enum`, `pattern`, `integer`,
`ordered_tokens` and a couple of ATT&CK IDs. The questions ask for descriptor fields of specific
specimens (role, attack technique, adapter, campaign id, URL, file path, stage count, step graph,
stage profile, obfuscation profile, next language). The "odd one out" wording in the prompt is flavour:
the journals record no question that needed cross-corpus reasoning.

## Solution

1. **Identify the bank.** The Single Solve bank was the Day 22 bank plus 44 IDs. All 44 were
   `polyglot-gdscript-*`: four roles (benign utility, downloader, launcher, staged chain) times eleven
   fields. The GDScript samples use the same v3 descriptor schema as every other polyglot family:
   `language, role, verdict, attack, adapter, host_profile, obfuscation_profile, stage_profile,
   next_language, stage_count, steps, campaign_id, url, file_path`. No new decoder was needed.
2. **Decode the descriptor.** Each specimen carries a `HUNTRESS_ANALYSIS_V1:<KIND>-<payload>` marker
   in a comment (and a duplicate `const DESCRIPTOR_...` string). The GDScript staged-chain specimen used
   `HB64`, hex-wrapped Base64. Decoding it gives a flat `key=value|...` record, e.g.
   `role=staged_chain|attack=T1059|steps=decode_embedded,write_file,dispatch_process|...`.
3. **Answer from the declared field for v3 families.** The question text prints a graph-to-ATT&CK
   precedence rule. For the v3 polyglot and host families the validator wanted the descriptor's own
   `attack`, `role` and `adapter`. The sample above is the clearest case: the graph rule would give
   `T1140`, but the descriptor declares `T1059`. The non-v3 families (Python, Bash, Node.js, JScript,
   VBScript, Batch) use the ordered-graph rubric instead.
4. **Take structural answers from the source.** Batch `*-control`, VBScript `control` and JScript
   function/invocation/property style come from the specimen's own structure (a signature classifier),
   never the file name.
5. **Check, then submit once.** The mapper reproduced `answers=1461 expected=1461 missing=0 extra=0`
   from the exact assigned bytes (SHA-256 verified first), and a pre-submission lint of kinds, enums and
   ATT&CK strings was clean. One full-map submission returned CORRECT.

## Time Trial and Mastery

**Time Trial.** The question IDs were a disjoint set from Single Solve, so the Single Solve mapper
could not be reused as-is. Differences found by decoding the real bytes:

- **`LYR` layered marker.** Every v3 polyglot specimen (80 of 80 affected members, including five
  GDScript files whose marker is split) used `HUNTRESS_ANALYSIS_V1:LYR-<op>_<op>_...-<payload>`, with ops
  such as `b32_b32_b32`, `hex_rev`, `b64_frag_b32`. This is the "layers"
  of the prompt. The ops are `hex`, `b32`, `b64`, `frag` (dot-chunking, undone by dropping the dots) and
  `rev`. They are undone right to left, because the rightmost op was applied last. The marker's op list
  also equals the specimen's own `obfuscation_profile` field, which was a useful self-check.
- **Uniform 13-question v3 template**, including derived fields (`url-host`, `path-leaf`,
  `campaign-token`, the part of the campaign id after its first hyphen).
- **PowerShell families relabelled** (`powershell-<id>-url/secondary/tertiary`); the scenario type had
  to be inferred from field shape.
- **Merged compound answers** for graph families, e.g. a Batch `-style` of `graph|control|expansion`.

All new suffixes were hand-checked against decoded values before submitting. One full-map call went
CORRECT.

**Mastery.** Seven AES-nested sets, each decoded independently (member basenames repeat across sets).
The first full submission was `incorrect`, so the rest went by set prefix (10 calls total):

| Call | Scope | Result |
|---|---|---|
| 1 | all 10,465 | incorrect |
| 2 | set 1 | accepted |
| 3-4 | set 2 | incorrect, then accepted after fix 1 |
| 5 | set 3 | accepted |
| 6-7 | set 4 | incorrect, then accepted after fix 2 |
| 8-9 | sets 5, 6 | accepted |
| 10 | set 7 | CORRECT (whole Mastery) |

The two fixes changed exactly 11 of 10,465 fields: 8 Python `attack` answers (`T1105` to `T1059.006`)
and 3 Node.js ones (`T1105` to `T1059.007`).

**Ranked reruns (bragging rights only).** Later reruns used fresh fixtures. The Time Trial reran with a
50 s window and the Mastery with a 230 s window. Both went CORRECT in one call: Time Trial in 3.95 s
and Mastery in 7.4 s of local wall time. Earlier ranked attempts hit the submission cap before
the ranked solver was fixed. Time Trial ranked-2's reject list was exactly six Python/Node.js `attack`
fields (the `T1105` default again), and `progress.md` also records a dropped v3 `attack` alternate group
fixed along the way. I did not trace the Mastery ranked-1 failure further. See the day overview for that
tooling story.

## Dead ends and gotchas

- **A confirmed set does not confirm a rule for shapes it lacks.** Single Solve, Time Trial and Mastery
  set 1 contained no Python or Node.js staged-chain specimens, so the mapper's `attack_for` looked
  validated when it was not. It tested `acquire_remote` (-> `T1105`) before the terminal
  `dispatch_process` rule. Sets 2/4/5/6 held the shape
  `profile_host > acquire_remote > write_file > change_mode > dispatch_process` and were rejected.
- **How the bug was localised without spending calls.** Diffing the mapper's hand-written rubric against
  the library's reference `derive_attack` showed disagreement on exactly the 8 Python attack fields. A
  cross-set table of each risky classification against accepted/rejected sets then showed Node.js
  `T1105` appearing only in rejected sets (4 and 6). Bash staged chains have no `attack` question in this
  bank, so Bash never mattered.
- **The fix, as it stood in the mapper:**

  ```python
  # staged chain that acquires remotely but TERMINATES in dispatch_process takes the
  # language sub-technique, not the ingress technique T1105
  if family in ('python', 'nodejs', 'bash') and 'acquire_remote' in steps \
          and steps[-1] == 'dispatch_process':
      return LANG_ATTACK.get(family, 'T1059')
  if 'acquire_remote' in steps:
      return 'T1105'
  ```
- **The prompt's printed graph rule is not the answer for v3 families.** GDScript `attack` stayed at the
  declared value. This held from Day 10 onward and was confirmed again here.
- **The question IDs change between modes.** Named IDs in Single Solve, hash IDs in Time Trial and
  Mastery, with different families, so dispatch on the live start response rather than on the challenge.
- One ranked solver also hit an antivirus file lock while extracting a hex-payload JS specimen to disk;
  an in-memory extraction cache fixed it. That is local tooling, not a challenge property.

## Takeaways

- Technique name: **inert descriptor decoding** (`HUNTRESS_ANALYSIS_V1` markers: HEX/B64/HB64/FRAG, and
  from this day `LYR` layered ops undone right to left). Declared descriptor field wins for v3 families;
  an ordered-graph rubric applies to the rest, with a structural classifier for Batch controls.
- A bank that is a strict superset of the previous day (`bank_inventory --against`) reuses the old
  mapper. A *disjoint* ID set means the schema changed, so diff it by hand.
- For a multi-set Mastery, diff your hand-written rubric against the library reference and tabulate each
  risky classification against which sets were accepted, before submitting per set. The value that appears
  only in rejected sets is the suspect. A per-set submission loop (set 1, set 2, ...) is a cheap
  bisection when the validator only reports batch-level `incorrect`.
- We could have avoided the Mastery fix by running the library's `derive_attack` as a cross-check against
  the mapper on Single Solve; its Python/Node.js staged-chain branch already existed.

## Evidence

Leaf: `challenges/day23/script-analysis/script-analysis-day-23-game-resource-loader/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/submission-attempt-log.json`,
  `time-trial/ranked-3/result.json`, `time-trial/ranked-3/accepted_plan.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py` (`attack_for`),
  `mastery/artifacts/submission-attempt-log.json`, `mastery/ranked-2/result.json`,
  `mastery/ranked-2/accepted_plan.json`; `time-trial/ranked-2/result.json` and `mastery/ranked-1/result.json`
  (pre-fix cap-reached attempts)
- `library/script-analysis/descriptor-schema-and-rubrics.md`, `descriptor_decode.py` (`decode_layers`)
- `progress.md` (grep `^| 23 .*game-resource-loader`)
- No per-challenge librarian note or lane log exists for this challenge (the Day 23 `_run/` holds only
  other lanes' logs).
