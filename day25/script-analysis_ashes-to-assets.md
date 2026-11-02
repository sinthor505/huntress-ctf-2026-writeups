# Ashes to Assets

| | |
|---|---|
| Day / Lane | 25 / Script Analysis |
| Challenge ID | `script-analysis-day-25-phoenix-stage-chain` |
| Technique | declared-descriptor decoding of a new language family (Elixir / Phoenix); descriptor fields win over the step graph |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Ashes to Assets" is a script-analysis bank that, relative to Day 24, added exactly one language family:
Elixir (`.exs`), plus one Phoenix-controller host specimen. Every specimen carries an encoded descriptor
marker, and the answers are the descriptor's own fields. The Day 24 mapper decoded the new Elixir markers
unchanged, so there was no fix round: Single Solve went CORRECT in 7 calls (all of them submission
grouping, not corrections), Time Trial and Mastery in one call each.

## The challenge

Prompt: "Configuration starts out in Elixir source and has changed shape by the time an inert
Phoenix-hosted stage picks it up. Follow every transformation, reconcile each one with earlier campaign
artifacts, and find the setting that didn't survive the rebirth."

- **Single Solve:** one `evidence.zip` (167 members, SHA-256 checked against the start response) and 1,571
  answer ids with named ids (`polyglot-elixir-downloader-attack`, `jscript-stager-role`, ...). Formats were
  `exact_text` for artifact-derived values (URLs, campaign ids, paths) and `enum` for roles and scenarios.
- **Time Trial:** 1,625 ids, one `evidence.zip`, hash-style ids (`powershell-58112f-url`,
  `polyglot-jsp-588efb-url-host`-style stems) and `LYR`-layered markers.
- **Mastery:** nine nested sets, each exactly the Time Trial's 1,625 ids with a `setN.` prefix (14,625 total).

Day 25 added 55 Single Solve ids (44 `polyglot-elixir-*` over four roles, 11 `host-elixir-phoenix-elixir`) and
65 per Time Trial / Mastery set (five Elixir members x 13 ids). Everything else was the Day 24 bank.

## Solution

1. **Recon.** The bank was diffed against the previous day's with the library's bank-inventory tool: a pure
   superset (55 added, 0 removed, 0 changed), all of it Elixir. That decided the plan: reuse the Day 24
   schema-dispatching mapper rather than write a new one.
2. **Decode the markers.** Elixir specimens carry a `# HUNTRESS_ANALYSIS_V1:<KIND>-<payload>` line comment
   (`B64`, `HB64`, `FRAG`, `HEX` in Single Solve; `LYR` layers in Time Trial/Mastery), and the same payload is
   assigned to a `descriptor_<id>` string with a `_plan_<id> = %{descriptor: ...}` map. The host specimen adds
   a `defmodule Huntress.InertController<id>` whose `show/2` returns an inert fixture string. The library
   descriptor decoder already handled these marker kinds; the only gap was two table entries (`.exs` mapped to
   the `elixir` family, and `elixir` mapped to its language ATT&CK value).
3. **Answer from the declared fields.** For every v3-style specimen the accepted value is the descriptor's own
   `role`, `attack`, `adapter`, `campaign`, `url`/`file_path`, `stage_count`, `steps` (joined with `>`),
   `host_profile`, `obfuscation`, `stage_profile` and `next_language`. The decoded fields were compared
   against what the validator finally accepted: Single Solve 50 of 50 Elixir fields, Time Trial 65 of 65,
   Mastery 585 of 585.
4. **Elixir `attack` is the bare `T1059`.** Elixir has no ATT&CK sub-technique, so launcher and staged-chain
   members take the parent technique; the downloader takes `T1105` and the benign utility takes `none`.
   These came straight from the descriptor, not from any rule we derived.
5. **Submit.** Single Solve was submitted as one call with the 1,565 non-control fields, then the six Batch
   `*-control` ids one structural-signature group at a time (`C`, `direct`, `F`, `if_init`, `goto`,
   `if_same`); every call returned `accepted` and the last returned `CORRECT`.

Accepted Elixir values for the Single Solve (descriptor fields, per role):

| Specimen | role | attack | steps (as accepted) |
|---|---|---|---|
| `polyglot-elixir-benign-utility` | `benign_utility` | `none` | `acquire_remote>dispatch_process` |
| `polyglot-elixir-downloader` | `downloader` | `T1105` | `decode_embedded>write_file>dispatch_process` |
| `polyglot-elixir-launcher` | `launcher` | `T1059` | `profile_host>decode_configuration>dispatch_language` |
| `polyglot-elixir-staged-chain` | `staged_chain` | `T1059` | `decode_configuration>return_configuration` |
| `host-elixir-phoenix-elixir` | `staged_chain` | `T1059` | `decode_embedded>write_file>dispatch_process` |

## Time Trial and Mastery

Both modes returned to the hash-id schema with layered `LYR` markers, while Single Solve kept named ids, so
the solver dispatches on the id schema in the start response (the Day 24 mapper already did). No code change
was needed. Time Trial and Mastery (regular, untimed) were each CORRECT in one call, as were the first ranked
attempts (Time Trial 50 s window, Mastery 290 s window). Mastery's nine sets each decoded to 1,625 answers
from separate per-set candidate maps, then went in as a single 14,625-id submission.

Later reruns of the ranked Time Trial took 7 calls instead of 1 (a different submission grouping; all groups
were accepted at their first variant and the last returned CORRECT). We could not determine from the journal
or logs why the grouping differed; no answer was ever rejected. One further ranked attempt stopped at the
per-ID submission cap before finishing (see the day overview).

## Dead ends and gotchas

- **The descriptors are self-inconsistent on purpose.** A `benign_utility` whose steps are
  `acquire_remote>dispatch_process`; a `staged_chain` whose steps read like a benign decode-and-return. Reading
  the step graph to derive role/attack would have disagreed with the declared fields. The validator accepted the
  declared fields every time; the prompt's "reconcile each one" wording did not call for anything beyond that.
- **The "setting that didn't survive the rebirth" had no visible counterpart.** We found nothing in the
  specimens or answer formats that needed a separate cross-stage reconciliation beyond reading `next_language`
  and `stage_count` from the descriptor.
- **Mapper hedging was never used.** Alternate readings (graph-derived attack, a Batch `for_once` control) were
  queued as insurance and never submitted; every group was accepted at variant 0.
- **Do not trust one day's mapper blindly for a new family.** This one worked only because the diff showed the
  new bank was Elixir alone; the same check tells you when a day introduces a new suffix instead.

## Takeaways

- **Diff the bank first.** On a day that adds one language, a superset diff against the previous day (added /
  removed / changed ids) tells you the whole job: add the file-extension and language-value table entries and
  rerun the existing mapper.
- **Declared descriptor wins** for role, attack, adapter and steps on v3 specimens, even when the step list
  contradicts it. This held for Elixir here and for the following new-language days.
- **Dispatch on the question-id schema** (named vs hash-id) in the start response, not on the day number.
- **Submit big bank maps grouped:** all non-control fields in one call, then Batch controls by structural
  signature.

## Evidence

Leaf base: `challenges/day25/script-analysis/script-analysis-day-25-phoenix-stage-chain/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (schema dispatcher), `legacy_solve.py` (Single Solve named-id path)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`,
  `time-trial/ranked-3/result.json`, `time-trial/ranked-vm-1/result.json` (cap reached),
  `time-trial/ranked-vm-2/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` through `mastery/ranked-3/result.json`
- `challenges/day25/_run/librarian/script-analysis-day-25-phoenix-stage-chain.md`
- `challenges/day25/_run/lane-script-analysis-day-25-phoenix-stage-chain.log`
- `library/script-analysis/` entries: descriptor decoder (SCRIPT-TECHNIQUE-C101), descriptor schema and rubrics
  section 4.1c Elixir (SCRIPT-REFERENCE-C102), bank workflow section 7 (SCRIPT-TECHNIQUE-C104)
- `progress.md` (grep `^| 25 .*phoenix-stage-chain`)
