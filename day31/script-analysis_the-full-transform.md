# The Full Transform

| | |
|---|---|
| Day / Lane | 31 / Script Analysis |
| Challenge ID | `script-analysis-day-31-xslt-final-gauntlet` |
| Technique | descriptor-driven specimen analysis; one new language family (XSLT) folded into a finale recap of the whole lane |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The lane's final challenge was an explicit recap: the prompt said it tied together "all thirty-one
languages, every hosted-source profile, and every finding family" the lane had used across the event,
plus one new addition, XSLT. The only real novelty was four `.xsl` specimens whose embedded descriptor
marker appeared twice, once in an XML comment and once (without its kind prefix) inside a bare
`<descriptor>` element. Every other rule from the previous thirty days held unchanged, including
declared-descriptor-wins for the new language's ATT&CK mapping. A teammate scored the Single Solve
before our first submission landed; Time Trial and Mastery were both graded CORRECT for us on fresh
`retry` attempts, with Mastery needing one resume after a transient Game Master outage.

## The challenge

Prompt: "An unseen XSLT-led campaign ties together all thirty-one languages, every hosted-source
profile, and every finding family you've proven. Read all of it, run none of it, and deliver the final
analysis."

| Mode | Questions | Delivery |
|---|---|---|
| Single Solve | 1,846 | one `evidence.zip`, 192 members, named ids (`resource-scenario`, `host-jsp-webshell-jsp-role`, ...) |
| Time Trial | 1,950 | one `evidence.zip`, hash-style ids (e.g. `powershell-84522a-url`) |
| Mastery | 29,250 | one outer ZIP of 15 nested `evidence-NN.zip` sets, each 1,950 ids prefixed `setN.` |

Comparing the Single Solve question bank against Day 30's with `bank_inventory.py --against` showed a
pure superset: 44 new ids, all `polyglot-xslt` (4 `.xsl` members x 11 fields), nothing else added,
removed, or changed. The Time Trial/Mastery hash-id bank grew by the matching 40 xslt fields across 4
specimens (10 fields each in that schema). Everything else in the bank was a straight recap of the
other thirty languages the lane had already covered.

## Solution

1. **Diff the bank first, as on every new-language day.** `bank_inventory.py --against` the Day 30
   Single Solve isolated the whole day's novelty to `polyglot-xslt`, so no time went into re-checking
   the other 30 recapped families.
2. **Find the descriptor marker in the new file shape.** Each `.xsl` specimen carries the lane's usual
   `<!-- HUNTRESS_ANALYSIS_V1:<KIND>-... -->` marker, decodable unchanged by the library's existing
   XML-comment reader. XSLT added one wrinkle: the same payload, without its `KIND` prefix, also sits
   inside a plain `<descriptor>` element in the specimen body. The library decoder read the comment and
   never needed the element copy on any delivery that was actually graded.
3. **Answer from the declared descriptor, not a graph reading.** As with every new-language day since
   Elixir, Erlang, Dockerfile, and Makefile, `role`, `attack`, `adapter`, and `steps` came from the
   specimen's own descriptor fields. For XSLT's `attack` field specifically, the MITRE technique for XSL
   abuse, `T1220` (XSL Script Processing), was a plausible alternative to the declared value for a
   downloader/launcher/staged-chain specimen, so the solver queued it as a held-back alternate rather
   than submitting it first.
4. **Dispatch on the question-id schema.** The Day 30 solver (`solve_v2` for hash ids, `legacy_solve.py`
   for Single Solve's named ids) was reused unchanged; only the `attack`-alternate list grew one entry
   (`T1220`) for the `xslt` family.
5. **Submit by group.** Unambiguous fields went in one large call; small per-language `attack` groups
   (JavaScript, JScript, VBScript, AppleScript polyglots) and a Batch control-structure group were probed
   separately in case any needed a non-default variant.

## Time Trial and Mastery

Single Solve returned `already_solved` on our very first submission (6 calls, 0 of 1,846 accepted): the
team had already scored it, so that map was never validated by the Game Master. We moved straight to
Time Trial and Mastery, which grade our own attempts independently of that.

- **Time Trial** (1,950 ids): CORRECT in 7 calls. All six probed groups (the Batch control group and
  five per-language `attack` groups) were accepted at variant 0; the final AppleScript `attack` group
  came back CORRECT directly.
- **Mastery** (15 sets x 1,950 = 29,250 ids): the first `retry` hit a transient Game Master `503
  temporarily_unavailable` with nothing issued or consumed. Before relaunching, we read the latest
  (teammate's) attempt's status to confirm 31 fresh variants were still available, then ran the solver
  offline against that teammate's downloaded Mastery fixture as a dry run: 29,250/29,250 answered, 0
  missing, 0 lint problems, 0 decode errors. A fresh `retry` then succeeded, and the real submission went
  CORRECT in 7 calls, all six groups accepted at variant 0.
- Later ranked reruns of both modes (50 s Time Trial window, 610 s Mastery window) also went CORRECT on
  the first call each. Ranked runs were bragging-rights only under the Day 30+ prize rules; the untimed
  `retry` results above are what counted.

## Dead ends and gotchas

- **The Single Solve map was never validator-tested.** The team's prior solve meant our 1,846-id named
  map, including the new `polyglot-xslt` fields, is only "observed," never confirmed CORRECT. A follow-up
  `retry` (405 `operation_not_allowed`) and `practice` (409 `retry_variants_unavailable`) both refused to
  issue a fresh attempt, consuming nothing, so there was no path to an independently graded Single Solve.
- **`T1220` was never needed.** Every `xslt` `attack` group, including the ones where the descriptor
  declared a bare parent technique (`T1059`) or `T1105` for a non-remote-looking graph, was accepted at
  variant 0 with the declared value. The XSL-specific technique was a reasonable guess that the validator
  did not want.
- **A transient Game Master 503 is not a solver bug.** The first Mastery `retry` failed server-side with
  nothing issued; the right response was to check status and re-validate offline before simply retrying,
  not to start changing code.
- **No format surprises.** `exact_text`, `enum`, and `mitre_attack_id` answers were accepted as declared,
  as on every prior recap day.

## Takeaways

- **Declared descriptor wins** held for a sixth consecutive new-language day (after Elixir, Erlang,
  Dockerfile, Makefile, and BeanShell). `declared_vs_plan.py` against the accepted Time Trial and Mastery
  plans showed 0 mismatches for the new family: 40/40 xslt fields (Time Trial, 4 specimens) and 600/600
  (Mastery, 60 specimens across 15 sets).
- Diffing the question bank against the immediately prior day, before touching the artifact, continues
  to isolate exactly what is new on a "final recap" day with thousands of recycled questions.
- When a specimen's marker is duplicated in two places in the same file, decode whichever copy the
  existing reader already understands first; only add a fallback for the other copy if a real delivery
  needs it. Here the `<descriptor>`-element fallback was written for robustness but never exercised on
  any graded delivery.
- A Single Solve a teammate already scored cannot be validated by us; the independent Time Trial and
  Mastery results, including a clean recovery from a mid-run server 503, are what show the workflow
  still works end to end on the lane's last day.

## Evidence

Leaf base: `challenges/day31/script-analysis/script-analysis-day-31-xslt-final-gauntlet/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-solo/solve.py` (schema-dispatching solver; `legacy_solve.py` is the
  named-id path), `single-solve/artifacts/scratch-solo/xslt_probe.py` (marker-shape probes)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`, `mastery/ranked-1/result.json`
- `challenges/day31/_run/lane-script-analysis-day-31-xslt-final-gauntlet.log`
- `challenges/day31/_run/resume-script-analysis-20261031T060837.log` (the Mastery 503 and resume)
- `challenges/day31/_run/supervisor/script-analysis-day-31-xslt-final-gauntlet.r1.log`
- `challenges/day31/_run/supervisor-report.md`
- `challenges/day31/_run/librarian/script-analysis-day-31-xslt-final-gauntlet.md`
- `library/script-analysis/descriptor-schema-and-rubrics.md` (section 4.1h)
- `progress.md` (grep `^| 31 .*xslt`)
