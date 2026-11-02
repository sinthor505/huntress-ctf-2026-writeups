# Mixed Signals

| | |
|---|---|
| Day / Lane | 26 / Script Analysis |
| Challenge ID | `script-analysis-day-26-beam-message-flow` |
| Technique | cumulative inert-script bank; decode the embedded behavior descriptor and answer from its declared fields (new family: Erlang `.escript`) |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 26 of the Script Analysis lane was the previous day's specimen bank plus one new language,
Erlang `escript` files. The new files carried the same inert behavior-descriptor marker as every
other recent family, so the existing decoder read them without changes. The one real decision was
which reading to trust for each Erlang specimen's `role` and `attack` fields: the descriptor's own
declared value or the one derived from its step graph. The declared value was right every time.
All three modes went CORRECT.

## The challenge

Prompt: "An Erlang escript passes messages between processes, and some of its indicators look
suspiciously familiar. Trace the message flow, and separate the true cross-language indicators from
deterministic collision decoys planted to look like old friends."

The prompt is about Erlang, but the delivered data is a mixed bank. Single Solve was one
`evidence.zip` (171 members across 27 file extensions: PowerShell, Batch, Bash, Python, JS/Node,
PHP, VBScript, Lua, Perl, Ruby, JSP, MOF, AutoIt, Elixir and others, plus 4 new `.escript` files)
and **1,615 answer ids**. Fields per specimen included `role`, `attack`, `adapter`, `campaign`,
`url`/`file-path`, `stage-count`, `steps`, `host-profile`, `obfuscation`, `stage-profile` and
`next-language`. Formats were mostly `exact_text` (978), then `enum` (215), `pattern` (168),
`integer` and `ordered_tokens` (126 each). Single Solve was untimed. Against Day 25's Single Solve
bank, the diff was 44 added ids (all `polyglot-erlang-*`: 4 specimens x 11 fields), none removed
or changed.

## Solution

1. **Diff the bank first.** We compared the question set against the previous day's. The 44 new ids
   were the whole delta, so the previous day's solver could be reused as-is and only the Erlang
   family needed thought.
2. **Decode the marker.** Each `.escript` starts with a `% HUNTRESS_ANALYSIS_V1:KIND-payload` comment
   (hex, Base64, hex-of-Base64 or dot-fragmented hex). The module body repeats the descriptor as a
   string returned by `build_plan/0`, which lets the decoder cross-check the marker against the
   in-body copy. The existing descriptor decoder handled both unchanged; only a suffix-to-family
   entry (`.escript` -> Erlang) and a language attack value were added for the derived view.
3. **Answer from the declared fields.** For this specimen schema each descriptor carries its own
   `role`, `attack` and `adapter`. Erlang has no ATT&CK sub-technique, so the declared `attack`
   values were `none` (benign utility), `T1105` (downloader) and the bare parent `T1059`
   (launcher, staged chain). The ordered step graph often disagreed with the declared role
   (for example a declared `T1059` on a staged chain whose graph reads acquire-then-dispatch); the
   validator sided with the declared field.
4. **Submit in groups.** The solver's output had 1,615 ids in 7 groups (attack, role, Batch control,
   and the rest as a base set). Round 0 went out as four calls: the 4-id Erlang `attack` group
   (accepted), the 1,607-id base set (accepted), the 2-id Batch `control` group (accepted), and the
   2-id Erlang `role` group, which returned CORRECT. Every group went in at its first variant; none
   of the queued alternates was used. The lane log puts Single Solve CORRECT about 1 min 43 s after
   recon.

One decision worth copying: the Erlang `attack` and `role` answers (the fields we were least sure
of) were put in their own small groups with alternates (graph reading, bare `T1059`, `T1105`)
rather than in the 1,607-id base set, so a wrong guess would show up as an isolated rejection
instead of sinking everything.

## Time Trial and Mastery

Time Trial delivered 1,677 ids in one `evidence.zip`; Mastery delivered 10 sets of 1,677
(16,770 ids, `setN.` prefixes, one nested `evidence-NN.zip` per set). Both used the hash-style id
schema (`polyglot-erlang-2d7c2a-role`, `powershell-d8219f-url`, layered `LYR` markers) instead of
Single Solve's named ids, so the solver dispatched on the schema in the start response, not on the
day. Each Erlang specimen contributed 13 ids per set (52 new ids in all). The Erlang fields matched
our accepted plans in every saved delivery (librarian check, 0 mismatches).

Results from the leaf's own result files, all with every group at variant 0:

- Regular (untimed) Time Trial and Mastery: CORRECT in 1 call each.
- Ranked Time Trial attempt 1 (50 s window): CORRECT in 1 call, 5.18 s locally. Ranked Mastery
  attempt 1 (320 s window): CORRECT in 1 call, 8.98 s locally.
- Later ranked reruns (Time Trial 2 and 3, Mastery 2): CORRECT. The two Time Trial reruns went out as
  4 grouped calls (role, attack, style, base) instead of 1, with variant 0 in every group. The lane
  log does not say why they were split.

Batch `style`/`control` answers (`call_subroutine`, e.g. `diamond|call_subroutine|call_double`)
were variant 0 as well; the `for_once` alternate was never needed in this challenge.

## Dead ends and gotchas

- **"Collision decoys" never showed up.** The prompt promises deterministic decoys that mimic
  indicators from other languages. In the Erlang specimens we saw only one descriptor (marker
  comment plus the identical `build_plan/0` string) and no second, conflicting indicator. We could
  not determine what the prompt's decoys referred to, and the validator accepted the plain decoded
  values anyway.
- **Graph reading vs declared field.** The queued insurance was the graph-derived reading of `role`
  and `attack`. In the saved Time Trial, 3 of 4 Erlang members had a declared reading that did not
  match the graph reading, and for some roles the graph reading was not even a valid role (it
  produced strings like `UNKNOWN:build_schedule,dispatch_memory`). Declared fields won.
- **Do not generalize one day's `attack` rule.** The bare `T1059` for a language with no
  sub-technique was confirmed by this challenge's own accepted plan, but staged-chain `attack`
  values varied by language and day elsewhere in this lane. Check each new family against its own
  validator result.
- The challenge ID says "beam-message-flow"; the display title is "Mixed Signals".

## Takeaways

- **Bank-diff first.** In a cumulative generated bank, diffing ids against the previous day tells you
  within seconds whether you face a new schema or just a new language; here it was 44 ids and one
  new file extension.
- **Declared descriptor field beats graph derivation** for these specimens. It held for Erlang and
  again for the next new languages (Dockerfile, Makefile).
- **Group uncertain fields separately** and submit them with alternates; it costs a few calls and
  keeps any rejection attributable.
- Dispatch the solver on the start response's schema (named vs hash ids, `LYR` vs plain markers),
  never on the day number.

## Evidence

Leaf base: `challenges/day26/script-analysis/script-analysis-day-26-beam-message-flow/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (schema-dispatching solver, with `legacy_solve.py`
  for the Single Solve schema)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (also `ranked-2`, `ranked-3`)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` (also `ranked-2`)
- `challenges/day26/_run/librarian/script-analysis-day-26-beam-message-flow.md`
- `challenges/day26/_run/lane-script-analysis-day-26-beam-message-flow.log`
- Library entries: descriptor decoder (Erlang placement row), descriptor schema and rubrics
  (section 4.1d, Erlang/BEAM), bank workflow (Day 26 repeat)
- `progress.md` (grep `^| 26 .*beam-message-flow`)
