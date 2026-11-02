# Next, Next, Finish

| | |
|---|---|
| Day / Lane | 22 / Script Analysis |
| Challenge ID | `script-analysis-day-22-installer-script-chain` |
| Technique | inert-descriptor decoding of a new script family (NSIS `.nsi`) inside a large multi-language bank; declared descriptor wins |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Next, Next, Finish" is the Day 22 script-analysis bank: 1,417 questions over 152 small, inert
scripts in one ZIP. Only one family was new, NSIS installer scripts (`.nsi`). They carry the same
`HUNTRESS_ANALYSIS_V1:` comment marker as earlier families, in four different encodings. The
existing marker decoder and the validator-confirmed rubric from earlier days handled all of it
unchanged, so the Single Solve went CORRECT in one submission about three minutes after
the attempt started, and Time Trial and Mastery each went CORRECT in one call on the first
rehearsal and the first ranked attempt.

## The challenge

Prompt: "An NSIS installer script clicks through its own setup wizard while carrying a little
something extra. Separate the real setup actions from the modeled payload behavior, and recover
every cross-language handoff from the final bytes. You won't need to press Next once."

Nothing here is executable. `start` returned one `evidence.zip` (Single Solve and Time Trial) and
1,417 answer ids with named, human-readable ids such as `polyglot-nsis-launcher-attack`. Mastery
delivered six such bundles (`evidence-01.zip` ...) and 8,502 ids, prefixed `set1.` to `set6.` (6 x
1,417). Every value is a string. The bank is cumulative: it re-asks the families of earlier days
(PowerShell samples, Batch, VBScript, JScript, Python variants, Bash, Node.js, PHP, Lua, Ruby, Perl,
Groovy, C# script, Google Apps Script, WMI MOF, TypeScript, AutoIt, AppleScript, MSBuild project,
AutoHotkey, GitHub Actions YAML) and adds 44 NSIS ids: four `.nsi` files (benign utility,
downloader, launcher, staged chain) with about eleven fields each (`role`, `attack`, `adapter`,
`campaign`, `url` or `file-path`, `stage-count`, `steps`, `host-profile`, `obfuscation`,
`stage-profile`, `next-language`).

The `attack` question prints a long precedence rule (terminal `handoff_script` or
`dispatch_next_stage` gives the language sub-technique, `acquire_remote` gives ingress transfer,
and so on) and says the parent technique is also accepted where a sub-technique is canonical.

## Solution

1. **Recon and bank inventory.** We checked the new bank against Day 20 and Day 21 with the
   library's bank inventory tool. Day 20's families plus Day 21's 44 GitHub Actions ids (already
   validator-confirmed) accounted for everything except 44 `.nsi` ids.
2. **Decode the NSIS marker.** Each `.nsi` hides its descriptor in a comment line,
   `; HUNTRESS_ANALYSIS_V1:<KIND>-<payload>`, where the kind is one of `HEX`, `B64`, `HB64`
   (base64 of text that is itself hex-encoded) or `FRAG` (hex split into dotted fragments).
   The marker decoder from Day 21 reads all four:

   ```python
   if kind == 'HEX':  raw = bytes.fromhex(payload)
   elif kind == 'B64': raw = base64.b64decode(payload, validate=True)
   elif kind == 'HB64': raw = base64.b64decode(bytes.fromhex(payload), validate=True)
   elif kind == 'FRAG': raw = bytes.fromhex(payload.replace('.', ''))
   ```

   The result is a small key/value or JSON descriptor holding the role, attack, adapter, campaign,
   steps and so on.
3. **Answer from the descriptor, not the graph.** The Day 10-20 rule applied again: for `polyglot-*`
   and `host-*` specimens the descriptor's own `role`, `attack` and `adapter` fields are the
   answer, even where the printed precedence rule would give something else. The NSIS values show
   why this matters. In the accepted plan the benign utility's steps are
   `decode_embedded>write_file>dispatch_process`, yet its declared attack is `none`. The launcher's
   steps are `decode_configuration>return_configuration`, which the rule reads as benign/none,
   yet its declared attack is `T1059`. The downloader's steps contain no `acquire_remote`, yet its
   declared attack is `T1105`. The staged chain (`acquire_remote>dispatch_process`, next language
   VBScript) took the parent `T1059`, not `T1059.005`. Reading the graph would have been wrong on
   several of these.
4. **Structural controls.** The Batch and VBScript `*-control` fields (14 to 18 per attempt) are
   classified from the script's own structure with the library's structural classifiers, not from
   names.
5. **Submit.** Two independently written solvers (one that copied the Day 21 solver, one built from
   the library rubric) produced 1,417 answers each with no lint failures; they differed in a few
   alternate groups (NSIS attack, Batch control `for_once`, collection attack, non-v3 staged-chain
   attack). One call with the preferred variant of every group came back CORRECT.

## Time Trial and Mastery

- Time Trial: same 1,417-id schema, fresh fixtures. A regular (untimed) attempt went CORRECT in
  one call (6.47 s of local work). The ranked attempt had a 95 s window and went CORRECT in one
  call (7.06 s total).
- Mastery: six sets in one attempt, 8,502 ids. The regular attempt went CORRECT in one call;
  the ranked attempt (200 s window) went CORRECT in one call (28.01 s). Each set was solved
  independently and merged.
- The question text and answer ids are identical across modes, but the fixture values are not:
  campaign ids and, notably, the Batch/VBScript control values change per attempt. For example the
  `batch-benign-control` answer was `errorlevel_guard` in the Single Solve and a different value in
  each later Time Trial fixture. A solver that cached per-id answers from one mode would have been
  wrong on 14-18 fields.
- We later reran the Time Trial and Mastery several more times (bragging-rights speed attempts
  only). Every logged call in those runs was `accepted` or `CORRECT`; none was rejected. Some of
  them submitted the attack and control groups first, then the rest, only as a submission-order
  choice. They are not the scoring basis.

## Dead ends and gotchas

- **No rejections at all.** Nothing in the lane log was marked incorrect in any mode, so there is
  no rejected-spelling story to tell. The only unvalidated risk going in was the NSIS family: the
  solvers' own notes say there was "no validator evidence yet" for it, and the declared-attack
  rule was a bet extended from other polyglot families. The first CORRECT settled it.
- **One solver failed on some Mastery sets.** In two sets of one early ranked Mastery run (and
  one set in a later run) the adapter-style solver exited with lint errors: JScript `relay` and
  `impact` specimens produced empty or non-string values. The second solver covered those sets and
  the merged submission was CORRECT. We did not diagnose why those JScript fixtures differed from the others.
- The `polyglot-nsis-benign-utility` file has a `file-path` indicator where the other three have
  a `url`. Reading the question list rather than assuming one suffix set per family avoids a
  missing-id error.

## Takeaways

- **Marker-comment descriptors scale across hosts.** A new host language (here NSIS, `;` comments)
  usually only changes the comment syntax; the four payload encodings (`HEX`, `B64`, `HB64`,
  `FRAG`) were already handled. Check the marker decoder before writing anything new.
- **Declared descriptor wins** for `role`, `attack` and `adapter` on polyglot/host families, even
  when the question prints a graph precedence rule and the declared value contradicts it.
- **Controls are fixture-specific.** Classify Batch and VBScript controls structurally from each
  attempt's own files; never reuse another mode's answers.
- Running two independent solvers and diffing them is cheap insurance for a 1,417-field bank,
  and the groups where they disagree are exactly the ones to hold as alternate submissions.

## Evidence

Leaf base: `challenges/day22/script-analysis/script-analysis-day-22-installer-script-chain/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (Day 21 solver reused; `marker_descriptor`)
- `single-solve/artifacts/cand-rubric/solve.py` and `single-solve/artifacts/cand-rubric/lib/`
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (further reruns in
  `ranked-2` to `ranked-4`, `ranked-vm-1`)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` (reruns in `ranked-2`, `ranked-3`)
- `challenges/day22/_run/lane-script-analysis-day-22-installer-script-chain.log`
- `progress.md` (grep `^| 22 .*installer-script-chain`)
- No librarian note exists for this challenge in `challenges/day22/_run/`.
