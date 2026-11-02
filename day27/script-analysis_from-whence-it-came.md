# FROM Whence It Came

| | |
|---|---|
| Day / Lane | 27 / Script Analysis |
| Challenge ID | `script-analysis-day-27-docker-build-attribution` |
| Technique | descriptor-marker decoding and attribution; the declared descriptor field beats the specimen's own step graph (new Dockerfile family on a reused mapper) |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"FROM Whence It Came" is a script-analysis bank of 1,659 questions over a ZIP of about 175
specimens. Day 27's only new content was four Dockerfiles, so the solve was an extension of the
previous day's validator-confirmed mapper rather than new analysis. The Dockerfiles' declared
`role`/`attack` values disagree with their own step graphs (the "skin-deep resemblance" the prompt
mentions), and the declared values were the accepted ones. A teammate scored Single Solve within
seconds of unlock, so ours was never graded. Time Trial and Mastery were ours via a fresh `retry`,
and went CORRECT in 4 calls and 1 call respectively.

## The challenge

Prompt: "A Dockerfile and several earlier specimens share strings, but sharing strings doesn't mean
sharing parents. Attribute each specimen using build-stage, semantic, and static-handoff evidence,
and expose the resemblance that's only skin-deep."

Delivery was a single `evidence.zip` (175 members) with 1,659 answer ids. Every answer is a string
in an `enum`, `exact_text` or similar format with `trim` normalization. The ids carry the family and
role, for example `polyglot-dockerfile-launcher-attack` or `bash-advanced-...`. They fall into
three groups:

- The three small `resource-*`, `collection-*`, `impact-*` PowerShell scenario blocks.
- Many languages with per-role specimens (Bash, Batch, VBScript, JScript, Node.js, the Python
  packaging variants, and the `host-*` webshell/host containers).
- A block of "polyglot" families, 44 ids each, covering PHP, Lua, WMI MOF, TypeScript, AutoIt,
  AppleScript, MSBuild, Perl, Ruby, JSP, Elixir, Erlang and others. Day 27 added exactly one
  family to this block: `polyglot-dockerfile`, 44 ids from 4 specimens.

## Solution

1. **Diff the bank against the previous day.** The Day 27 id set was a strict superset of Day 26's.
   The only additions were the 44 `polyglot-dockerfile` ids; nothing was removed or changed. That
   made the Day 26 mapper (which had gone CORRECT in all three modes) the starting point.
2. **Decode the Dockerfile markers.** The four Dockerfiles each hide a descriptor in a comment
   (`HUNTRESS_ANALYSIS_V1:...`) in one of four encodings: fragmented hex, plain hex, base64 and
   hex-then-base64. The library's descriptor decoder already handled all four, with 0 decode errors
   over the 175 members. Each Dockerfile is a single `FROM scratch AS analysis_<hex>` stage, and an
   `ARG` copy of the descriptor equals the marker payload (checked per specimen, so no second stage
   or divergent copy was hiding a different answer).
3. **Read the fields from the descriptor, not from the Dockerfile.** Most answers (`campaign`,
   `adapter`, `url`, `file-path`, `obfuscation`, `stage-count`, `steps`, ...) came straight from the
   decoded descriptor. The ambiguous ones were `role` and `attack`.
4. **Declared `role`/`attack` win.** The Dockerfile descriptors declare values that do not match the
   specimen's own `steps` graph. For example, the specimen declared as `downloader` has the graph
   `decode_configuration>return_configuration`, which contains no download step, and the declared
   `launcher` (graph `acquire_remote>dispatch_process`) declares `T1059` where a graph reading
   gives `T1105`. Day 26's Erlang family had the same shuffle and its declared values were
   accepted, so the solver made the declared value variant 0 of each group and kept the
   graph-derived readings as alternates:

   ```python
   # cand-adapter/solve.py
   NEW_FAMILIES = ('dockerfile',)   # own attack group: declared, then graph reading, then bare T1059
   LANG_ATTACK = {..., 'elixir': 'T1059', 'erlang': 'T1059', 'dockerfile': 'T1059'}
   ```
5. **Submit by group.** Role, attack and the Batch `control` fields were submitted as their own
   groups so a wrong reading could be isolated, with everything else in one bulk call.

Single Solve is the part we did not get to test. A teammate had already solved it, and each of our
Single Solve submissions returned `already_solved` with nothing graded (four rounds, 52 ledger
entries). The Single Solve answer map was never validated by the Game Master. It was validated
indirectly: the same mapper, unchanged, was accepted in Time Trial and Mastery.

## Time Trial and Mastery

- **Fresh attempts.** The first `start` for Time Trial and Mastery returned the team's CORRECT
  readout, which has no artifact and no answer ids. Our first automatic attempts raised
  "artifact missing" for that reason. A `retry` issued a fresh attempt with its own artifact
  (the team readout is not a verdict and does not block a retry).
- **Same schema as Single Solve.** Time Trial and Mastery reused Single Solve's named ids with the
  same 1,659-id set, not the hash-based ids and layered markers of Days 23 to 26. Mastery was 11 sets
  of 1,659 ids, 18,249 answers in one bulk submission. The dispatch should key on the id shape in
  the start response, not on the day.
- **Time Trial:** the untimed attempt went CORRECT in 4 calls (control, role, a 1,652-id bulk, then
  attack, accepted group by group; the last call finished it). The timed attempt (50 s window)
  went CORRECT in 1 call, 8.94 s from issue to result locally. A later timed rerun needed 4 calls
  for the same reason (group-by-group submission order) and went CORRECT in 13.4 s.
- **Mastery:** untimed 1 call (19.1 s total); timed (350 s window) 1 call, 16.9 s total.
- **Variant 0 everywhere.** Every group was accepted at variant 0 (the declared reading); none of
  the graph-based alternates was ever needed. Checked afterwards on 52 Dockerfile specimens across
  Time Trial and Mastery: the decoded fields matched the accepted plans with 0 mismatches
  (44/44 and 484/484 per attempt). In every set the declared attack disagreed with the graph for
  all four Dockerfiles, and the declared role for three of four.

## Dead ends and gotchas

- **`already_solved` is not a verdict.** All of Single Solve's 52 submissions were
  `already_solved` and said nothing about our mapper. It looked like "0 accepted of 1,659" in the
  log, which is an artifact of nothing being graded, not a systematic error. The right move was to
  go to Time Trial and Mastery, which support independent `retry`.
- **The status payload is not a fixture.** Re-running the solver on the team-solved readout fails
  with no artifact. (An automation-side contract issue, collected in the day overview.)
- **The graph is a decoy.** Reading role and attack from `steps` (a plausible "semantic" approach
  given the prompt) gives wrong answers for these Dockerfiles. The prompt's "resemblance that's only
  skin-deep" describes the shared strings across families, but the validator wanted the descriptor's
  own declaration.
- **Batch controls.** Batch one-item `FOR ... CALL` controls kept a `for_once` alternate as in earlier
  days. It was not needed this time: `call_subroutine` was accepted.
- **Not determined:** why the declared values disagree with the graphs. We could not tell from any
  file whether it is deliberate (a decoy to test attribution) or a generator quirk. We only know the
  declared values were accepted on Day 26 and Day 27.

## Takeaways

- **Technique names:** descriptor marker decoding (FRAG/HEX/B64/HB64), bank superset diffing across
  days, "declared descriptor field wins". Diff the new day's id set against the previous day first,
  and if only one family is new, extend the validated mapper by one family instead of rewriting.
- Verify the one new family cheaply and offline: one `FROM` stage, the `ARG` copy equals the marker,
  decode errors zero. Then the only open questions are the `role`/`attack` groups, which should be
  submitted separately with the declared reading first.
- Treat a team-scored Single Solve as ungraded for us, not as evidence about our answers, and move to
  the modes that can be re-issued.

## Evidence

Leaf base path: `challenges/day27/script-analysis/script-analysis-day-27-docker-build-attribution/`

- `single-solve/journal.md`, `single-solve/team_solved.json`, `single-solve/start_response.json`
- `single-solve/artifacts/cand-adapter/solve.py`, `single-solve/artifacts/cand-adapter/legacy_solve.py`
  (`dockerfile_warnings`), `single-solve/artifacts/feedback-round1.json` to `feedback-round3.json`
  (all `already_solved` or `cap_reached`)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`
- `challenges/day27/_run/lane-script-analysis-day-27-docker-build-attribution.log`
- `challenges/day27/_run/librarian/script-analysis-day-27-docker-build-attribution.md`
- `progress.md` (grep `^| 27 .*docker-build-attribution`)
