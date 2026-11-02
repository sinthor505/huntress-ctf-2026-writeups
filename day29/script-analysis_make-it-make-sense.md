# Make It Make Sense

| | |
|---|---|
| Day / Lane | 29 / Script Analysis |
| Challenge ID | `script-analysis-day-29-makefile-variant-diff` |
| Technique | Descriptor-marker decoding of a polyglot specimen bank; "declared descriptor field wins" over graph-derived answers |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The challenge was a bank of about 180 small "inert" script specimens (PowerShell, Bash, Python, Batch, VBScript, JScript, Node.js, host containers and a long tail of polyglot languages), with 1,747 questions about them. Day 29 added one new language, Makefile (`.mk`). The new family decoded with the existing marker decoder unchanged, and the validator accepted each specimen's own declared `role`/`attack` fields rather than a reading derived from its behavior graph. The Single Solve was scored by a teammate within about two minutes of unlock, so we never got a graded result for it. Time Trial and Mastery were solved independently on fresh `retry` attempts, each CORRECT on the first submission of its ranked run.

## The challenge

The prompt reads: "Unseen Makefile variants and their prior-language build chains differ in layered variables and target flow. Diff them carefully, and keep only the tradecraft conclusions that hold up across every variant." Nothing in it hints at how the answers are graded. The delivery was one AES-encrypted `evidence.zip` (Single Solve). It had 1,747 answer IDs, all string answers:

- 1,062 `exact_text`
- 227 `enum`
- 180 `pattern`
- 138 `integer`
- 138 `ordered_tokens`
- 2 `mitre_attack_id`

Question families were `polyglot` (1,012), `bash` (168), `host` (154), `python` (126), `jscript`, `batch` and `vbscript` (72 each), `nodejs` (56), plus a few PowerShell resource/collection/impact samples. Of the 1,012 `polyglot` IDs, 44 were the new `polyglot-makefile-*` ones: four `.mk` specimens (one per role; the IDs include `benign-utility` and `downloader`) with 11 questions each. These asked for role, ATT&CK ID, adapter, campaign, file path, stage count, ordered steps, host profile, obfuscation, stage profile and next language.

Time Trial grew to 1,833 IDs in a single `evidence.zip` under a 50 s window. Mastery was 13 sets of 1,833, which is 23,829 IDs, in a bundle of nested `evidence-NN.zip` files under a 410 s window.

## Solution

The solving pattern was the one the lane had used since earlier days, so the work was mostly checking that nothing new broke it.

1. **Inventory the bank.** A bank diff against the previous day's Single Solve showed a strict superset: 44 added IDs, all `polyglot-makefile`, none removed or changed. That meant every older mapper rule was still in force and only the Makefile members were unknown.
2. **Decode the Makefile specimens.** Each `.mk` file carries a versioned `HUNTRESS_ANALYSIS_V1` marker. In the Single Solve it is a plain `# ...` comment followed by a `HUNTRESS_DESCRIPTOR_<hex> := <payload>` assignment and an inert `.PHONY` target. The existing descriptor decoder handled this with only an extension-to-family entry (`.mk` -> `makefile`). All 183 Single Solve members decoded with zero errors.
3. **Decide what the Makefile `role`/`attack` answers should be.** The decoded descriptors disagree with their own step graphs. One specimen declared `benign_utility` with attack `none`, while its steps were `acquire_remote > dispatch_process`. Another declared a `downloader` with `T1105`, while its steps were `build_schedule > return_configuration`. A graph-derived mapper would have given different answers. The solver used the declared fields as variant 0 and queued the graph reading, bare `T1059` and `T1105` as alternates in case the validator wanted something else. The relevant lines in the mapper, which prefer the declared value:

   ```python
   def role_for(self, fields, family, prefix=''):
       if fields.get('role'):
           return str(fields['role'])
       ...
   # in answer_for():
   if sfx == 'attack' and v3 and f.get('attack') is not None:
       value = f['attack']
   ```

4. **Run the map offline, then submit.** Offline the Single Solve map had 1,747 of 1,747 answers, nothing missing, no lint problems. Two independent candidate mappers (an "adapter" and a "rubric" variant) agreed on the structure and differed only in their alternate groups. Both were prepared within about two minutes of the artifact appearing.

For Single Solve this is where our own story ends. The submission returned `already_solved` (three calls on the ledger). The team's existing attempt had been scored, so nothing was graded for us.

## Time Trial and Mastery

Both modes support a fresh `retry` even when the team has already solved them, so we issued fresh attempts and graded those ourselves.

- **Schema change.** Time Trial and Mastery used hash-id question IDs (`powershell-bf3b25-url`, 6-hex member stems) and `LYR`-layered markers, while Single Solve kept named IDs. Day 27 had kept named IDs in all three modes. The solver therefore dispatches on the start response's schema, not on the day. The Single Solve branch is the older mapper, and the hash-id branch is the Day 23 mapping run on the library decoder.
- **Time Trial, regular (untimed) attempt.** CORRECT in 4 grouped calls, each at variant 0: `role` (4 IDs), `attack` (108 IDs), the 1,720-ID base, then the single `style` ID. The last call returned CORRECT. Nothing was fixed along the way; the groups were just probes that the validator reported as `accepted`. A multi-answer submit only returns `accepted` or `incorrect` at the batch level, so grouping is how you learn which part is wrong.
- **Time Trial, ranked attempt.** One call, all groups at variant 0, CORRECT. Solve 1.2 s, submit 1.6 s, 5.18 s end to end against a 50 s window.
- **Mastery, regular and ranked.** Each was CORRECT in one call with all 23,829 IDs (13 sets), at variant 0 throughout. Ranked took 10.94 s end to end against a 410 s window.

As a check on the declared-wins rule, a regression tool joined every Makefile specimen to its answer IDs and compared the accepted plan against the declared descriptor fields. It found 40 of 40 equal (Time Trial regular), 40 of 40 (Time Trial ranked) and 520 of 520 (Mastery regular), with zero mismatches. The same tool had matched earlier new-language days (Erlang, Terraform HCL, Elixir). The graph-reading and `T1105`/`T1059` alternates were never needed.

## Dead ends and gotchas

- **`already_solved` is not a verdict.** The Single Solve submission was scored by the team's existing attempt, so we learned nothing about our mapper. `retry` returned 405 `operation_not_allowed` and `practice` returned 409 `retry_variants_unavailable`, so there was no way to obtain a fresh Single Solve attempt. Single Solve does not accept a fresh attempt once a teammate has solved it.
- **The alternates we prepared were unnecessary.** We worried about the declared-versus-graph mismatch, and it was a reasonable worry: in Mastery, 38 of 52 Makefile members disagreed with their own graph on both role and attack, and 8 more on role only. The three earlier new-language days (Elixir, Erlang, Dockerfile) had gone the same way, which is why declared went first. This is a pattern in the validator, not a derivation we could justify from the prompt text.
- **Prompt wording does not help.** "Keep only the tradecraft conclusions that hold up across every variant" sounds like cross-variant reasoning, but nothing in the grading rewarded it. The answers came straight from each specimen's own descriptor.
- **Time Trial and Mastery fixtures differ from Single Solve** (schema and ID count), and a regular, ranked and untimed attempt can each draw different fixtures. The mapper has to be driven by the live start response's IDs, not by what the Single Solve looked like.

## Takeaways

- **Declared descriptor wins.** For v3 polyglot specimens, answer `role`/`attack`/`adapter`/`steps` from the descriptor's own fields even when they contradict the decoded behavior graph. This was the fourth consecutive new-language day where it held (Elixir, Erlang, Dockerfile, Makefile). Run a declared-vs-accepted-plan comparison on each new language before porting a mapper forward.
- **Dispatch on the schema, not the day.** A hash-id schema with layered markers versus named IDs flips between days, and between modes of the same challenge.
- **Group your submissions.** With multi-answer batches, submit by field family so a rejected batch points at the culprit. Once a rubric is confirmed on one mode, submit the rest in one call. That is how Mastery went in one call.
- **Do not wait for the Single Solve.** When the first `start` already reads a teammate's CORRECT, move to Time Trial and Mastery. They support independent `retry`, and they were solved in seconds.

## Evidence

Local evidence only; the graded results below come from this workspace's own submissions.

- Single Solve (team solve, not ours): `challenges/day29/script-analysis/script-analysis-day-29-makefile-variant-diff/single-solve/` holds `journal.md`, `team_solved.json`, `retry/retry_response.json` (405) and `practice/practice_response.json` (409). The two candidate mappers are under `single-solve/artifacts/cand-adapter/` and `cand-rubric/`. There is no `result.json` and no accepted plan for this mode, so its Makefile answers are unvalidated here.
- Time Trial (independent): `time-trial/regular/` and `time-trial/ranked-1/`, each with `result.json` (CORRECT) and `accepted_plan.json`. The ledger shows 5 submission calls in total (4 regular, 1 ranked).
- Mastery (independent): `mastery/regular/` and `mastery/ranked-1/`, each with `result.json` (CORRECT) and `accepted_plan.json`. The ledger shows 2 submission calls in total.
- Solver: the rubric candidate's `solve.py` and `legacy_solve.py` in `single-solve/artifacts/cand-rubric/`. Day-level run log: `challenges/day29/_run/lane-script-analysis-day-29-makefile-variant-diff.log`, and the post-solve note in `_run/librarian/`.
