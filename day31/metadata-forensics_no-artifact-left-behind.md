# No Artifact Left Behind

| | |
|---|---|
| Day / Lane | 31 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-31-no-artifact-left-behind` |
| Technique | execution-chain graph reconstruction across mixed-format artifacts, layered on a full Day 1-30 recap |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 31, the event's 24-hour finale, closed the metadata-forensics lane with a new "graph" block on
top of a recap of every prior day's format reader: rebuild the single execution-and-delivery chain
through a ZIP of 22+ extensionless artifacts, each naming its own next hop in a different file
format (PowerShell, PNG, SQLite, EML, ZIP, and a JSON variant in Time Trial). The insight that
mattered was that the "obviously broken" edges — PowerShell launchers with mismatched quotes — are
still part of the accepted chain; a reading that dropped them as malformed disagreed with the
validator in a third of the Mastery sets. The Single Solve was already scored by a teammate before
our own submission landed; Time Trial and Mastery were both solved independently, though a
parallel-submission bug in the release pipeline twice made a genuine Time Trial CORRECT look like a
failure before it was recognized and fixed.

## The challenge

Prompt: "Everything is evidence now. Reconstruct the complete execution-and-delivery chain, defend
every edge with primary metadata, and identify the tempting conclusion the corpus does not support."

- **Single Solve:** one question, `graph.execution_chain_artifact_ids` for `case-001.zip` (inside
  `metadata-day-31-evidence.zip`), answer format "compact valid JSON array with exact values and
  order; no Markdown or explanatory text". The case held 22 extensionless
  `artifacts/<word>-<hex4>` members: 5 PowerShell, 6 PNG, 6 SQLite, 3 EML, 2 inner ZIP.
- **Time Trial:** 36 fields — the same graph field, Day 30's recap field
  (`relation.identical_sha256_groups`), one recap field per file format from Days 1-29 (JPEG, PNG,
  PDF, ZIP, PE, .NET, MSI, LNK, OOXML, email, audio, video, ELF, Mach-O, APK, JAR, SQLite, EVTX,
  registry, Prefetch, pcap, disk image, X.509), plus a synthesis-ordering field. The case format
  added a JSON hop record not present in the Single Solve.
- **Mastery:** 15 sets of the same 36-field question, 540 answers total, delivered as one outer ZIP
  of 15 `evidence-NN.zip` bundles, each holding its own `case-001.zip`.

## Solution

1. **Library check first.** No existing library entry covered a `graph.*` field; everything else
   fell through to the evidence-bundle toolkit's generic per-format readers
   (`question_facts.py`), already proven on Day 30's cluster field.
2. **Read every member by its real bytes, not its name.** Each format hides its "next hop" in one
   primary field: a PowerShell execution statement (`Invoke-Item`/`Start-Process`/`&` against a
   quoted path), a PNG `tEXt`/`iTXt` chunk's `Comment` field (`next=<id>`, with `done <hex>`
   marking a terminal), a SQLite `launch_queue.target` cell that is exactly an id, a base64-decoded
   EML attachment's `route.txt` lines (`open: <id>` / `forwarded to: <id>`), and an inner ZIP's
   `run.txt` line (`execute <id>`).
3. **Two traps in the naming.** Two PowerShell launchers had mismatched or unterminated quotes
   (`& ".\summit-d467'`, `Start-Process '.\birch-41eb"`) — a real parse error, so that edge doesn't
   actually run. Member name words also repeated across unrelated ids (`birch-bd18` vs
   `birch-41eb`, `meadow-*`, `summit-*`), and free-text notes/comments/Subject lines mentioned bare
   words that were not edges at all (`nectar a78687`). The reader matched exact ids only.
4. **Build the graph and walk it.** Computing in-degree over the well-formed edges found exactly
   one zero-in-degree member (`birch-bd18`); walking forward gave a 4-node chain ending at an
   explicit `done` marker on a PNG (`orchard-e5bc`). The other 18 members formed a ring with no
   entry point, broken only by the two malformed launchers — the "tempting longer chain" the prompt
   warns about, since the longest valid segment of that ring has 14 nodes.
5. **Candidate, never graded.** The Single Solve submission was
   `["birch-bd18","nectar-1767","bramble-e622","orchard-e5bc"]`. The submit call returned
   `already_solved`: a teammate had scored this Single Solve already, so nothing was graded for us
   and this candidate stayed structurally checked only, never validator-confirmed on its own case.

## Time Trial and Mastery

The Time Trial case added a fifth hop format — JSON records (`{"task":..,"next":"<id>","owner":..}`,
terminal `{"done":true,"ref":..}`) — which the Single Solve's reader didn't yet parse; it fell
through to a generic exact-id scan that missed the terminal. A `json_doc` reader was added, with no
change to any already-chosen answer value.

The real story in Time Trial was a race in the release automation, not in the data:

- The first regular Time Trial attempt (`ea1cbcb3`) submitted 35 recap fields plus the graph field
  as two parallel group calls. The graph call came back **CORRECT**. A sibling call for the Day 30
  cluster field was graded a fraction of a second later as `already_solved`, because our own
  CORRECT had just closed the attempt — and the submission code collected results with
  `list(ex.map(...))`, which re-raised that sibling's exception and discarded the CORRECT entirely.
  The round was logged as `already_solved`, which read as "nothing graded," so the orchestrator
  launched fix rounds against an attempt that was in fact already solved.
- Fix round 2 escalated to a stronger model and raised the per-ID submission cap from 30 to 70. Fix
  round 3 resubmitted the identical answer map and got `incorrect` — by then a fresh Time Trial
  attempt (`d97cb5cc`) had been issued elsewhere on the team, and submissions to the stale attempt
  id were being graded against the new one's fixtures. 28 submission calls were spent this way
  before the loop was recognized as chasing a closed attempt.
- A first resume attempt then hit `cap_reached` (34/30): the resume path restored the fix-round
  counter but not the lane's escalated-cap flag, so it fell back to the un-escalated cap of 30
  with no fix round left to re-trigger the escalation.
- With both bugs fixed (`tools/release/rl.py` now collects every parallel future and lets a CORRECT
  win over a sibling's `already_solved`; `tools/release/orchestrate.py` re-marks a resumed lane as
  escalated and issues one fresh attempt on `already_solved` instead of spawning fix workers), a
  second resume went straight to a clean result: Time Trial CORRECT on attempt `d97cb5cc` in 2
  calls (`base[35]` accepted, then the cluster field's variant 0 CORRECT), and Mastery CORRECT on
  attempt `9a3f580f` in a single 540-answer call.

A Mastery pre-check also caught a second, unrelated bug before any submission: the candidate solver
looked up `case-001.zip` by bare name inside the outer bundle, so every one of the 15 sets resolved
to set 1's case. It was fixed to follow each question's own `artifact` chain
(`evidence-NN.zip -> case-001.zip`), with per-set caching.

That same pre-check settled the malformed-launcher question for real: across all 17 cases seen (the
Single Solve case, the Time Trial case, and all 15 Mastery sets), the graph is always exactly one
entry-to-sink walk over *every* parsed edge plus pure decoy cycles — malformed PowerShell launchers
sit on the real chain about as often as they sit on the decoy ring (5 of 11 vs. 30 of 56). In 5 of
the 15 Mastery sets (3, 9, 10, 11, 12) the valid-edges-only reading disagreed with that whole-walk
reading; the whole-walk primary was submitted and accepted in Mastery with no alternate spent,
settling the question the Time Trial case alone could not (it had only one malformed launcher, so
both readings happened to coincide there).

## Dead ends and gotchas

- **`already_solved` right after our own CORRECT is not a team solve — it can be our own pipeline
  discarding its own result.** The tell was in the lane log: the exact same attempt id showed a
  `CORRECT` line followed by `already_solved` a fraction of a second later. Treating that as a
  failure and running fix rounds against a now-closed (or superseded) attempt wasted 28 submission
  calls and an unnecessary model escalation before the race was found and fixed.
- **Dropping malformed edges looks reasonable and is wrong on this challenge.** A PowerShell
  launcher with a parse error genuinely wouldn't execute, but the validator's chain runs through
  it anyway in a third of the Mastery sets. The "tempting conclusion the corpus does not support"
  in the prompt is exactly this kind of plausible-but-wrong inference, not a second field to answer.
- **A bare-name lookup for `case-001.zip` silently grabbed the wrong set in Mastery.** Every nested
  `evidence-NN.zip` contains a file with the same name; resolving it correctly requires following
  the question's own `artifact` chain rather than searching the whole bundle for a name match.
- **Name collisions and bare-word mentions in notes are not edges.** Several unrelated members
  shared a name prefix (`birch-*`, `meadow-*`, `summit-*`), and SQLite notes, EML subjects, ZIP
  readmes and PowerShell comments all mentioned artifact words in passing; only an exact id in the
  correct primary field counts as a hop.
- Other `graph.*` fields (root, terminal, malformed edges, decoys, chain length) were never asked in
  any mode this day; any mapping for them stayed an untested guess.

## Takeaways

Name the technique: **execution-chain graph reconstruction over heterogeneous artifact formats**,
where each format exposes a "next hop" in its own idiom and the answer is the entry-to-sink walk
over *all* parsed edges (not just the well-formed ones), with decoys forming an unentered ring.
Validate the "keep or drop malformed edges" choice against several independent cases before trusting
it — the Time Trial case alone couldn't distinguish the two readings, and only the Mastery's 15 sets
did. Promoted to `library/metadata-forensics/artifact-chain-graph/` (META-CHAIN-GRAPH-001) with a
reusable `chain_graph.py`. What we would do faster: run the parallel-submit race fix and resume-time
escalation-flag fix before a finale day rather than after losing a chain of calls to it, and check a
multi-set Mastery bundle's structure (all cases, not just one) before locking in which edge reading
is the primary.

## Evidence

Leaf base: `challenges/day31/metadata-forensics/metadata-forensics-day-31-no-artifact-left-behind/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/team_solved.json` (`reason: "start_already_correct"`),
  `single-solve/artifacts/cand-solo/solve.py`
- `time-trial/regular/result.json` (CORRECT, attempt `d97cb5cc`), `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`
- `mastery/regular/result.json` (CORRECT, attempt `9a3f580f`, 540 answers), `mastery/regular/accepted_plan.json`,
  `mastery/ranked-1/result.json`
- `challenges/day31/_run/lane-metadata-forensics-day-31-no-artifact-left-behind.log`
- `challenges/day31/_run/resume-metadata-forensics-20261031T062333.log` (cap_reached, before the
  escalation-flag fix) and `...T062508.log` (clean TT + Mastery CORRECT after the fix)
- `challenges/day31/_run/supervisor/metadata-forensics-day-31-no-artifact-left-behind.r1.md` (root
  cause and the exact code fixes in `tools/release/rl.py` and `tools/release/orchestrate.py`)
- `challenges/day31/_run/librarian/metadata-forensics-day-31-no-artifact-left-behind.md`
- `library/metadata-forensics/artifact-chain-graph/README.md` (META-CHAIN-GRAPH-001)
- `progress.md` (grep `^| 31 .*no-artifact-left-behind`)
