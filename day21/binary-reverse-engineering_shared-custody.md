# Shared Custody

| | |
|---|---|
| Day / Lane | 21 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-21-shared-library-boundaries` |
| Technique | cumulative recap bundle; ELF host + shared-object pair (`e_machine`, HCFG config record in the plugin) |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Shared Custody" is the Day 21 entry in the lane's cumulative recap series. The delivered archive carries every
earlier day's specimen (`d01` to `d20`) plus a new pair: an ELF loader and an ELF shared-object plugin. Only 12
of the 252 questions were new. The other 240 repeat Day 20's questions, so the Day 20 answer map was reused and
extended with a small `map_day21` function. All three modes went CORRECT on the first submission call of their
first attempt.

## The challenge

Prompt: "No binary is an island - this one leans on shared libraries for everything. Trace the exports, the
imports, and the loader's handshake, and mind the boundaries." The story text does not describe the questions
at all, which is typical of this lane.

Delivery: one AES-encrypted ZIP (`evidence.zip`, password `infected`). Single Solve and Time Trial each gave one
archive and 252 answer IDs (21 days x 12 fields). Mastery gave five archives and 1,260 IDs, with a `setN.`
prefix on every ID. Every answer is a string. Time Trial had an 80 s window on the ranked attempt and Mastery
325 s; the untimed regular attempts had no deadline.

The 12 new `d21_*` fields:

- `d21_artifact_sha256_1` and `_2`: SHA-256 of the first and second `d21--` member in ZIP central-directory order.
- `d21_analysis_loader_machine` and `d21_analysis_plugin_machine`: the ELF `e_machine` of the member whose
  basename ends `loader.elf` or `plugin.cmxs`, as an unsigned base-10 integer (the question says not to
  substitute an architecture label).
- `d21_analysis_config_{campaign,domain,mutex,url}` and `d21_analysis_plugin_config_{campaign,domain,mutex,url}`:
  the top-level string fields of the first CRC32-valid HCFG record. The record layout is `HCFG`, version byte,
  XOR key byte, little-endian length, little-endian CRC32 of the decoded payload, then the XOR-encoded JSON.
  The `config_*` questions name the plugin and note that "the loader has no scored HCFG config". The
  `plugin_config_*` questions name the plugin directly.

## Solution

1. **Recon.** The start response showed 252 IDs, one artifact, and a SHA-256 that matched the download. The
   question IDs for `d01` to `d20` were identical to Day 20's, so this was the same recap-bundle shape we had
   already solved. The library's recap-bundle entry gave the field catalog and the AES-ZIP, member-order
   procedure.
2. **Reuse the Day 20 mapper.** Every `d01` to `d20` value came from the Day 20 accepted map's logic. The
   solver's output for those 240 IDs was checked equal to Day 20's accepted values before anything was
   submitted.
3. **Map the new pair.** Members were found by basename suffix, not position. The hash fields came straight
   from the central-directory listing. `e_machine` was read from the ELF header of each file. A library HCFG
   scanner found exactly one CRC-valid record in the plugin, at file offset 4364, and none in the loader.
4. **Fill both config families from the plugin.** The core of `map_day21`:

   ```python
   loader = member_path(unique("loader.elf")).read_bytes()
   plugin = member_path(unique("plugin.cmxs")).read_bytes()
   put(answers, "d21_analysis_loader_machine", elf_e_machine(loader))
   put(answers, "d21_analysis_plugin_machine", elf_e_machine(plugin))
   # The loader carries no scored record; the plugin's first CRC-valid record serves both families.
   rec = valid_json_record(HCFG.scan(plugin))
   for key in ("campaign", "domain", "mutex", "url"):
       put(answers, f"d21_analysis_config_{key}", rec["decoded_json"][key])
       put(answers, f"d21_analysis_plugin_config_{key}", rec["decoded_json"][key])
   ```
5. **Lint and submit.** The solver asserted that its key set equalled the start response's `answer_ids` and
   ran an answer-format lint, then submitted all 252 answers in one call. Result: CORRECT, one call
   (`progress.md`: "252-field map").

Accepted values for the new fields in the Single Solve: both `e_machine` fields `62` (x86-64), campaign
`BR21-000`, domain `node-000.day21.example.invalid`, mutex `Global\HuntressBR21_000`, URL
`https://node-000.day21.example.invalid/api/BR21-000`, and the two SHA-256 values in central-directory order.
The config values are the same for `config_*` and `plugin_config_*`.

## Time Trial and Mastery

Nothing new had to be solved. The same mapper ran against fresh fixtures:

- **Time Trial.** Untimed regular attempt: CORRECT, 1 call, 9.0 s end to end. The first ranked attempt was
  CORRECT in 1 call, 8.17 s of an 80 s window.
- **Mastery.** Five archives, 1,260 IDs, one mapper run per delivery with the `setN.` prefix handled by the
  solver. Regular attempt: CORRECT, 1 call, 13.85 s. First ranked attempt: CORRECT, 1 call, 11.1 s of a 325 s
  window.

Later ranked reruns (Time Trial four more times, Mastery once, between 2026-10-31 and 2026-11-01) were also
CORRECT. Those were speed-scoreboard reruns, not the first solve. The Time Trial reruns carried two
member-order alternate groups (the swapped order of the two hash answers) and so took 3 submission calls: the
250 non-group answers, then each group on its own, with variant 0 (central-directory order) accepted every
time. The first ranked attempt had no groups (`groups: 0`). We could not determine from the files why the
alternates appeared only in the reruns; the central-directory reading was correct throughout.

## Dead ends and gotchas

- **No dead ends on this challenge.** All 12 new fields were new to the validator and untested, and the first
  submission still passed. The risk was in guessing the conventions, so the solver carried alternates for the
  member order, the config record choice and a `has_hcfg_section` predicate, but none was ever needed.
- **`e_machine` is the numeric value.** The question text says not to substitute a label. Earlier days'
  `machine` fields accept labels such as `x86-64` or `I386`; here the integer `62` is the answer.
- **The loader is a decoy for the config questions.** It carries no valid HCFG record. Reading the "loader has
  no scored HCFG config" sentence literally and scanning only the plugin was correct, and both the `config_*`
  and `plugin_config_*` families take the same plugin record.
- **Members are identified by suffix, not index.** The archive holds `d01--` to `d21--` members and the
  loader/plugin pair is found by `loader.elf`/`plugin.cmxs`. The sha256 fields instead use central-directory
  position, so the two selection rules must not be mixed up.

## Takeaways

Technique names for searching: **cumulative recap bundle**, **ELF `e_machine` read**, **HCFG record scan with
CRC32 validation**. When a day's questions are a strict superset of the previous day's, diff the question list
first. Here only 12 IDs were new, and the right move was to port the previous accepted map, assert that its
output on the old IDs is unchanged, and write only the new reader. Run it against every Mastery delivery before
submitting. Reading the "which member" and "which order" sentences in each question literally was enough; no
disassembly was needed, despite the prompt's talk of exports, imports and the loader handshake.

## Evidence

Base path: `challenges/day21/binary-reverse-engineering/binary-reverse-engineering-day-21-shared-library-boundaries/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (the `map_day21` function), `single-solve/artifacts/cand-rubric/solve.py`
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (first ranked), `time-trial/ranked-2/` to `ranked-4/` and `ranked-vm-1/` (reruns with the member-order groups)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `challenges/day21/_run/lane-binary-reverse-engineering-day-21-shared-library-boundaries.log`
- Library entry BRE-RECAP-B105 (cumulative recap bundles) and the HCFG record scanner (HCFG-001)
- `progress.md` (grep `^| 21 .*shared-library-boundaries`)
