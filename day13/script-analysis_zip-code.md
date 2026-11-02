# Zip Code

| | |
|---|---|
| Day / Lane | 13 / Script Analysis |
| Challenge ID | `script-analysis-day-13-archive-delivery-chain` |
| Technique | offline static reconstruction of inert behavior descriptors in 105 scripts; validator-rubric diagnosis |
| Single Solve | Independent (CORRECT on call 48; response saved in the leaf) |
| Time Trial | Independent (CORRECT on call 91 of an untimed practice attempt; later ranked rerun CORRECT on call 1) |
| Mastery | Independent (CORRECT on call 1, untimed regular attempt; later ranked rerun CORRECT on call 1) |

## Summary

"Zip Code" is a 913-field static-analysis bundle: one AES ZIP of 105 scripts in about a dozen
languages, each carrying an inert, encoded descriptor, with questions about every script's role,
ATT&CK technique, adapter, stage graph and indicators. The offline decoder and mapper we built for
Single Solve was right for every field except two Batch "control" classifications, where the
validator accepted `call_subroutine` against a prompt that explicitly said `for_once`. Finding
those two cost most of the 48 Single Solve calls. With that override baked in, Mastery (4,565
fields) went CORRECT in one call.

## The challenge

Prompt: "A delivery archive nests scripts inside containers inside more containers. Trace each
member's role and handoff without executing a single one, and identify the link in the chain that
was never supposed to be delivered."

- Delivered: `evidence.zip`, AES-encrypted (password `infected`), 105 members, about 199 KB
  unpacked. Nothing in it is run; every answer comes from reading bytes.
- 913 answer fields over 104 of the 105 members: 540 `exact_text`, 149 `enum`, 102 `pattern`,
  60 `integer`, 60 `ordered_tokens`, 2 `mitre_attack_id`.
- Specimens: Bash (including advanced-syntax examples), Node.js, JScript, VBScript, Batch/CMD,
  PowerShell, Python (plain source plus recovered-carrier names: pyc, PyInstaller, PyArmor,
  cx_Freeze, py2exe), and 38 polyglot/host-container files (PHP webshell, Lua, AutoIt,
  AppleScript, WSF/HTA/SCT, TypeScript, WMI MOF).
- Question families repeat per role (`benign`, `stager`, `dropper`, `downloader`, `launcher`,
  `chain`, `relay`, `impact`, ...): `*-role`, `*-attack`, `*-adapter`, `*-campaign`, `*-url`,
  stage counts, ordered behavior graphs, encoding profiles, and `*-control` for Batch.

## Solution

1. **Inventory.** The 913 IDs map to 104 member names; exactly one member,
   `invoke-python-stage.ps1`, is never asked about. It declares `Executed = $false` and its
   handoff ID matches the one in `pyarmor-pyinstaller-recovered.py`, so it appears to be the PowerShell wrapper handing off to that recovered Python stage. Ten decoded `next_stage` edges name targets,
   and two of those targets (`stage-68de7826ac.ps1`, `stage-ae56e14aa0.ps1`) are not in the
   archive. We could not tell from the files whether either is "the link never delivered" the
   prompt refers to; the validator took the declared filenames verbatim (call 47).
2. **Decode each descriptor without running anything.** Every script carries the same logical
   record (role, attack, adapter, campaign, steps, host/stage profile) behind a per-language
   encoding: Python literals, VBScript/JScript/Node arrays, Batch `set` fragments, or a
   `HUNTRESS_ANALYSIS_V1:` comment marker in `HEX-`, `B64-`, `HB64-` or `FRAG-` form for the
   polyglots. The material itself comes in several representations (decimal arrays, reversed
   Base64, URL-safe Base64, `\u00XX` escapes, percent-hex) plus a documented byte transform:

   ```python
   if low in ('single_byte_xor', 'xor'): ...
   if low == 'rolling_xor':
       prev = nums[0] if nums else (int(m.group(1)) if m else 0)
       out = []
       for v in data: out.append(v ^ prev); prev = v
       return bytes(out)
   if low in ('add_modulo_256', 'add_mod_256'): return bytes((v - n) & 255 for v in data)
   ```

   The parsers read source as text and parse literals; none of them imports or runs a specimen.
   All 105 members decoded with zero errors.
3. **Map descriptor facts to answers, keeping three sources separate.** Declared descriptor
   fields (`attack`, `adapter`, `role`, host/stage profile), graph-derived classifications
   (terminal intent of the ordered step list) and syntax observations (Batch wrappers, Bash
   advanced syntax) are different evidence. For polyglot and `host-*` specimens the descriptor's
   own `attack` field was used; the validator accepted all 34 of those ATT&CK fields. For other
   families the graph rule applied (two Python chains ending in `dispatch_process` after remote
   acquisition took `T1059.006`).
4. **Batch controls.** The Batch questions say: if a one-item `FOR` loop invokes a subroutine that
   contains the reconstruction statement, classify the outer wrapper as `for_once`, not
   `call_subroutine`. The mapper followed that, with a regex for `FOR /L %%i IN (1,1,1) DO CALL
   :LABEL` whose label body contains the reconstruction. This is the one rule the validator
   disagreed with (see below).
5. **Submit by cohort, not whole.** The full map came back `incorrect` with no field feedback, so
   we submitted subsets: 190 direct indicators (accepted), Batch controls grouped by structure,
   then adapters, attacks, roles and profiles per language family. A cumulative `remaining`
   count was the only signal; call 48 delivered the last three PowerShell scenario fields and the
   Game Master returned CORRECT.

## Time Trial and Mastery

- **Time Trial.** Same 105-member, 913-field shape, but a fresh archive per attempt (different
  SHA-256 each time). The first two timed attempts (50-second windows) expired: the first full
  map was rejected as a batch (consistent with the `for_once` override problem; the journal does
  not state the cause), and the second hit the deadline boundary. We finished on an untimed
  practice attempt. The first 10 cohort calls confirmed 199 IDs, and only Batch control groups
  were ever rejected. After 90 calls had confirmed 418 IDs, we re-verified the mapper was
  byte-identical to the Single Solve one and submitted the remaining 495 values unedited in one
  call (call 91): CORRECT. Calls 11-90 were avoidable.
- **Mastery.** The bundle is an outer AES ZIP holding `evidence-01.zip` ... `evidence-05.zip`, each
  a 105-member archive with the same member names; answer IDs are `setN.<id>` (5 x 913 = 4,565).
  A small wrapper unpacked each inner archive into its own directory and ran the unmodified
  Single Solve decoder and mapper per set (105/105 members decoded, 913/913 candidates every set),
  then merged with set prefixes and applied the Batch override (three IDs changed, one each in sets
  2, 4 and 5). One submission of all 4,565 fields was CORRECT.
- **Ranked reruns.** Later (2026-10-26) a scripted solver using the same mapper, in a ranked
  Time Trial (50 s window) and Mastery (170 s window), went CORRECT on call 1 each, total pass
  times 2.1 s and 5.6 s. These were bragging-rights attempts, not the scoring path.

## Dead ends and gotchas

- **The prompt's rubric was wrong for Batch one-item `FOR` wrappers.** Source and question both
  said `for_once`; the validator rejected `for_once` for `batch-benign-control` and
  `batch-relay-control` individually and accepted `call_subroutine` (calls 13 and 27). In Time
  Trial, `for_once` was likewise rejected for the benign and chain controls. Seven other Batch
  controls were accepted as submitted from the mapper's other branches. The rule is a validator
  quirk, not a reading error: Mastery applied the same override to the three such wrappers it
  contained across five sets and was accepted in one call. We never found out why the stated rule
  and the validator differ.
- **Rejection is batch-level only.** `incorrect` on a 913-field call told us nothing about which
  field was wrong; two wrong Batch controls hid 911 good fields. Only the accepted-ID count moved on cohort calls.
- **Descriptor literals beat intuition.** `batch-benign-adapter` is `for`, a shell builtin, and the
  validator took it because the decoded descriptor says so.
- **Absent `next_stage` targets are still answers.** The two missing filenames were accepted
  verbatim.
- **Fixtures change per attempt.** Each Time Trial attempt and each Mastery set drew different
  specimens (different hashes, different descriptor values), so overrides keyed to one specimen
  would not carry; the Batch rule had to be structural.

## Takeaways

- Technique name: **descriptor reconstruction by static decoding** (decode the inert marker or
  literal, never execute), with the descriptor `attack`/`adapter`/`role` as the answer for
  polyglot and host-container families.
- When a large field map is rejected whole, bisect by *kind* of rule, not by file: confirmed
  cohorts shrink the unknown set faster than single IDs, and the Batch controls (the only rule
  family whose stated semantics could be suspect) should be isolated first and submitted by
  structural signature.
- Once an offline mapper has been validated on one mode of a challenge, submit the whole remainder
  of the next mode in one call. Time Trial's 80 extra calls bought nothing; Mastery's single call
  is the model.
- Treat a prompt's classification rule as a hypothesis when a validator can disagree; a
  structural alternate (`call_subroutine`) should be queued as a variant from the start.

## Evidence

Leaf base: `challenges/day13/script-analysis/script-analysis-day-13-archive-delivery-chain/`

- `single-solve/journal.md`, `single-solve/start_response.json`
- `single-solve/artifacts/static_decode.py`, `answer_map.py`, `archive_inventory.py`
- `single-solve/artifacts/phase5-call48-gm-response.json` (CORRECT),
  `diagnostic-single-batch-benign-control-call_subroutine.json`,
  `phase3-call27-batch-relay-control-call_subroutine.json`
- `time-trial/journal.md`, `time-trial/artifacts/call91-response.txt`,
  `time-trial/artifacts/submission-attempt-log.json`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/extract_bundle.py`, `prepare_sets.py`,
  `mastery_map.py`, `call01-response.txt`, `mastery/ranked-1/result.json`
- `tools/ranked/solvers/script-analysis-day-13-archive-delivery-chain/solve.py`
- Library: the script-analysis descriptor-reconstruction and Batch control classification entries
  (SCRIPT-TECHNIQUE-001; batch control signature rules)
- `progress.md` (grep `^| 13 .*archive-delivery-chain`)
