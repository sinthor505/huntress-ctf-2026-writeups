# Out of Order

| | |
|---|---|
| Day / Lane | 28 / Script Analysis |
| Challenge ID | `script-analysis-day-28-terraform-cloud-timeline` |
| Technique | descriptor-decode bank solve with a new polyglot language (Terraform HCL); declared descriptor fields win |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Out of Order" is the Script Analysis lane's cumulative inert-specimen bank (about 1,700 to 21,000
questions) with one new language added for the day: Terraform HCL (`.tf`). The Terraform theme in the title
and prompt is flavor. We saw no question in the saved question sets that asked about resource ordering
or a timeline. The new `.tf` members decoded through the existing generic descriptor path with no code change,
and the descriptor's own declared `role` / `attack` / `steps` values were accepted. Time Trial and Mastery were
CORRECT. The Single Solve had already been scored by a teammate, so our offline map for it was never graded.

## The challenge

Prompt: "Terraform HCL describes cloud resources, expressions, and dependencies that change over time.
Assemble the timeline from resources, prior scripts, and relationship evidence - without planning or
applying anything - and find the change that happened out of order."

Delivery was one `evidence.zip` per attempt (SHA-256 verified against the start response). Question counts:

- Single Solve: 1,703 answers, named IDs (`polyglot-terraform_hcl-staged-chain-role`, ...). The four Terraform
  members were `polyglot-terraform_hcl-{benign-utility,downloader,launcher,staged-chain}.tf`, 44 questions in total.
- Time Trial: 1,781 answers, hash-stem IDs (`polyglot-terraform_hcl-767e8f-url`, ...). The Terraform part was 4 members, 52 questions.
  Of the 1,781, 1,326 were `polyglot-*`; the rest were the older Python, Bash, JScript, Batch, VBScript, Node.js
  and PowerShell families.
- Mastery: 12 sets in a nested ZIP, the Time Trial schema repeated per set with a `setN.` prefix, 21,372 answers.

Answer kinds across the bank: `exact_text`, `enum` (role), `ordered_tokens` (the `a>b>c` behavior graph), `pattern`
(MITRE ATT&CK technique ID) and `integer` (stage count). Every Terraform member carried the usual descriptor
fields: role, attack, ordered steps, stage count, stage profile, obfuscation profile, next language, campaign id,
URL, file path, plus derived `url-host`, `path-leaf` and `campaign-token`.

## Solution

1. **Recon.** The start response gave the schema. The Single Solve used named IDs, while Time Trial and Mastery
   used hash-stem IDs, so the mapper was dispatched on the question schema, not on the day.
2. **Reuse the bank solver.** The solver was the previous days' validator-confirmed one, with its decoder library
   (`descriptor_decode`, `script_structure`) copied beside it. Each specimen carries an inert descriptor marker
   in a comment, wrapped in encoding layers. The decoder peels the layers and returns the structured descriptor. Nothing is executed.
3. **Terraform needed nothing new.** The `.tf` members decoded generically. The solver ran against the real
   Single Solve artifact with `answers=1703 expected=1703 missing=0 lint=0` before any submission.
4. **Answer from the declared descriptor, not the graph.** For the v3 polyglot families the descriptor's own
   `role`, `attack` and `steps` are independent of each other. In the accepted Time Trial plan, a Terraform member
   with `role=uploader` and steps `transmit_remote>dispatch_process` and another with `role=uploader` and steps
   `decode_embedded>build_schedule>return_configuration` both took `T1041`. A `benign_utility` member whose graph
   ended in `dispatch_process` took `none`, and a `dropper` took `T1140`. The prompt prints a graph-precedence rule for the
   ATT&CK field, but following it would have contradicted these accepted values.
5. **Submit by family.** The solver emitted one base map plus alternate-reading groups (`style`, `role`, `attack`) for
   fields it was unsure of. The first Time Trial attempt went in as four calls (style, 1,672 base fields, role, then
   the 105-field attack group, which came back CORRECT). In every group the first variant was accepted. The
   library's `declared_vs_plan.py` later confirmed 40 of 40 Terraform fields in the saved Time Trial equal the declared descriptor values.

## Time Trial and Mastery

- **Single Solve.** Our first submission returned `already_solved`: a teammate's attempt had scored it, so nothing was
  graded for us and automation stopped submitting there. Its journal does not record a `retry` or `practice`
  probe for this ID, so we cannot say from our files whether a fresh Single Solve could have been issued. The
  1,703-answer map was never validated by the Game Master in this mode.
- **Time Trial.** The `start` response read as the team's CORRECT, so we issued a `retry` for a fresh attempt. An
  untimed attempt went CORRECT (4 calls, 1,676 fields accepted incrementally, then the attack group). A ranked attempt (50 s window,
  hall-of-fame eligible) went CORRECT in 1 call, solved in 4.77 s locally.
- **Mastery.** The same approach with a per-set loader for the nested set ZIPs: untimed attempt CORRECT in 1 call
  (all 21,372 answers), then a ranked attempt (380 s window) CORRECT in 1 call, 8.74 s locally. Terraform members
  across the 12 sets drew several roles and attack values (`T1053`, `T1041`, `T1105`, `T1140`, `T1059` and `none`),
  all accepted at their declared values.

Our progress ledger counts 5 submission calls for Time Trial (4 untimed plus 1 ranked) and 2 for Mastery (1 untimed plus 1 ranked). Attempt
records on Time Trial and Mastery show that each attempt's fixtures differ (the ranked Time Trial's `attack`
values for the same file stems differ from the untimed attempt's). Mappers must therefore be run per attempt, never copied by ID.

## Dead ends and gotchas

- **The title and prompt were not predictive.** Nothing in the question families asked for a Terraform
  timeline or an "out of order" change. The Terraform content was one more descriptor-bearing language in a bank
  that kept growing cumulatively.
- **Attack rubric differences between modes.** The Single Solve ATT&CK question text was lenient: it said a parent
  technique, an earlier ingress technique, or `none` for certain relays would also be accepted. The Time Trial
  text was strict: "exact technique ID recorded in the specimen; no parent-technique or alternate-technique alias is
  accepted." The accepted value was the declared one in both cases, so the strict wording was the one to
  follow. We could only confirm this in Time Trial and Mastery, since the Single Solve was not graded for us.
- **Unvalidated alternates.** The solver flagged the Terraform `attack` fields (`v3-attack-new-family`) and Batch
  `control` fields (`batch-control-F`) as unvalidated. Because the first variant was accepted on every group, none of the alternates were used.
- Submitting a whole mode in one call (as the Mastery was) is only safe once a mapper is validator-confirmed.
  Here the Time Trial had already confirmed it, so Mastery needed 1 call per attempt.

## Takeaways

- **Declared descriptor fields win.** A new language in this bank rarely needs a new decoder. Decode the
  descriptor marker and answer from its own fields, not from the printed graph rule or from the steps.
  Terraform was the fourth new language in a row (after Elixir, Erlang and Dockerfile) where this held.
- **Dispatch on schema and set structure, not on the day.** Named IDs in Single Solve, hash-stem IDs in Time Trial,
  `setN.` prefixes in Mastery.
- **Run a Time Trial before trusting a Mastery map.** When Single Solve is team-scored, the Time Trial is the
  cheapest real validation of an unvalidated mapper, as it was here.

## Evidence

Leaf base path: `challenges/day28/script-analysis/script-analysis-day-28-terraform-cloud-timeline/`

- `single-solve/journal.md`, `single-solve/team_solved.json`, `single-solve/start_response.json`
- `single-solve/artifacts/cand-adapter/solve.py` (schema-dispatching solver; `legacy_solve.py` is the named-ID path)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`
- `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- `challenges/day28/_run/lane-script-analysis-day-28-terraform-cloud-timeline.log`
- Library: the descriptor-decoder / descriptor schema and rubrics entries and the declared-vs-plan check
  (`library/script-analysis/`)
- `progress.md` (grep `^| 28 .*terraform-cloud-timeline`)
