# Trust Issues

| | |
|---|---|
| Day / Lane | 23 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-23-anti-analysis-recognition` |
| Technique | cumulative recap bundle: mechanical container, symbol and embedded-config extraction across 23 earlier specimens |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (ranked rerun `result.json`; original attempt journal-recorded) |
| Mastery | Independent (ranked rerun `result.json`; original attempt journal-recorded) |

## Summary

The prompt describes a binary that checks the clock, sniffs its environment and branches when it
thinks it is being watched. None of that matters: the delivery is a cumulative recap bundle of the
Day 1 to Day 23 specimens, and all 276 questions are mechanical facts about those files (hashes, ELF
or PE headers, symbol names, an embedded `HCFG` config record). We reused the recap tooling and
mapper lineage from earlier days, and the work was spelling traps in the validator. Single Solve
needed 16 submission calls, Time Trial and Mastery one call each.

## The challenge

Prompt: "This binary doesn't trust you. It checks the clock, sniffs the environment, and takes
different branches when it thinks it's being watched. Spot every predicate and force the truth
out." The Game Master listed it as "Trust Issues".

Single Solve delivered one AES-encrypted `evidence.zip` (password `infected`) with 30 members
covering Days 1 to 23: ELF64 x86-64 for most days, managed PE32 (.NET) for Days 3, 22 and 23, JARs
for Days 6 and 7, a WebAssembly pair for Day 16, a stripped/symbolized ELF pair for Day 15, and an
OCaml `.cmxs` plugin for Day 21 that is still ELF underneath. There were 276 answers
(100 `pattern`, 100 `exact_text`, 36 `integer`, 32 `enum`, 8 `json`), IDs of the form
`dNN_artifact_sha256_N` and `dNN_analysis_<field>`. Every question opens with "Use all dNN-- members
in ZIP central-directory order" or a rule for choosing the member.

Time Trial was the same shape on a fresh variant. Mastery was seven independent deliveries of that
bundle, 1,932 answers with `setN.` prefixes.

## Solution

1. **Recon.** We verified `evidence.zip` against the start response's SHA-256, extracted it, and
   classified every member by magic bytes, never by extension. Day 21's `.cmxs` is ELF, and Days 3,
   22 and 23 are PE with CLR metadata.
2. **Facts in one pass.** We ran the library's binary inspector (`bre_inspect`) with `--json
   --symbols --group-by '^(d[0-9]{2})'` to get container facts, `.symtab` names, section names,
   overlay fields and HCFG records per day. A short `answer_map.py` turned those facts and the
   per-member manifest into the 276-answer map. The only per-question logic was picking the right
   member (the `stripped`, `symbolized`, `compact`, `direct`, `loader` or `plugin` suffix in the
   question ID, or the sole `--sample*` member).
3. **New in Day 23.** Day 23 adds a managed PE with CLR metadata: `metadata_version` is the string at
   `BSJB + 16` (`v4.0.30319`), and the config fields come from the first `HCFG` record in the
   assembly whose CRC32 validates.
4. **Submission.** The full 276-answer map came back `incorrect`. `incorrect` is batch-level only,
   so we submitted by field family and read `accepted` and `remaining` as the signal (below). The
   last 18 odd singleton fields (Day 1 `func`/`marker`, Day 2 symbol regexes, Days 4/5 predicates,
   Days 6/7 classfile fields) returned CORRECT on call 16.

   | Calls | Group | Result |
   |---|---|---|
   | 1 | all 276 | incorrect |
   | 2-3 | `artifact_sha256_*` (30) | accepted |
   | 4-6 | `format` (23); calls 4 and 5 were the same payload | incorrect, incorrect, then accepted after the JAR fix |
   | 7-11 | `machine` (19), split by answer kind | all-19 incorrect; integer subset accepted; `I386` subset incorrect; `i386` accepted; remaining 12 accepted |
   | 12-15 | architecture, HCFG version, structural (55); offsets, XOR keys, CRCs (37); campaign and domain (48); mutex and URL (46) | accepted |
   | 16 | remaining singletons (18) | CORRECT |

## Time Trial and Mastery

Time Trial: we ran the Single Solve `answer_map.py` unchanged on the new variant and submitted all
276 answers in one call. CORRECT on call 1. The one difference we noted was that the Day 10 member
was a PE32+ (AMD64) here where Single Solve's was ELF, which classifying by magic bytes already
handled.

Mastery: seven deliveries, each verified against its own declared SHA-256. The member container varied
by set: Day 2 was PE in six sets and ELF in set 6, Day 4 differed across sets, and Day 10 was PE in
sets 1, 2, 3, 5, 6 and 7 and ELF in set 4. This broke the mapper once, offline, before any submission: the Day 2 C++/Rust
symbol predicates (`has_cpp_typeinfo`, `has_cpp_vtable`, `mangled_score_symbol`, `typeinfo_symbol`)
read the ELF `symtab` key and raised `KeyError` when Day 2 was a PE. The fix was a helper that reads
the symbol list for whichever container is present, keeping COFF names as stored:

```python
def symbol_names(entry):
    facts = entry.get("facts", {})
    if entry["kind"] == "elf":
        return facts.get("symtab", [])      # SHT_SYMTAB, table order
    if entry["kind"] == "pe":
        return facts.get("symbols", [])     # primary COFF names, prefixes kept
    return []
```

All 1,932 answers went in one call and were CORRECT.

Later ranked reruns (window 80 s for Time Trial, 465 s for Mastery) used a thin solver over the
library's generalized fact resolver, validated offline against the accepted maps first (0 of 276 and
0 of 1,932 mismatches). Both were CORRECT on call 1, in 10.5 s and 30.4 s locally. One extra respelling was
needed for the Time Trial map: on a PE member, the Day 4 and Day 10 machine/architecture fields take
`AMD64` rather than the resolver's generic `x86-64`. These ranked runs were for the speed
scoreboard, not the prize ranking.

## Dead ends and gotchas

- **The anti-analysis theme is flavor.** There is nothing to defeat: no clock checks, no
  environment predicates, no debugger traps. Every one of the 276 questions is static extraction.
  A long reverse-engineering session would have been wasted effort.
- **JAR `format` wants the literal word `JAR`.** The first `format` attempt gave `zip/jar`, the
  inspector's internal container label, for the Day 6 and Day 7 Java specimens and the group was rejected.
  The question's own syntax text lists `JAR`, `Java archive`, `Java Archive (JAR)` and
  `application/java-archive`; submitting `JAR` was accepted.
- **PE `Machine` for the .NET days is lowercase `i386`.** The COFF spelling is `I386` and the inspector
  emits that, but the validator rejected it for Days 3, 22 and 23 (three fields) and accepted `i386`. Other
  machine labels (`x86-64`, and integer codes for the fields that asked for integers) were accepted as the
  inspector produced them. A pattern-kind `machine` field's syntax text says labels are "common file,
  objdump ..." forms, which does not tell you the case; we found it by splitting the group.
- **Compound container tokens.** Day 3's `format` is a combined container-plus-runtime token. We
  submitted `PE32_CLI` (container `PE32`, plus `_CLI` when managed) and it was accepted. Day 22
  and 23's `format` are `enum` fields (`pe_cli` etc.), a different shape for the same fact, so the
  answer has to follow each question's own `answer_format`.
- **Per-set container drift in Mastery** (above): a specimen that was ELF in Single Solve can be PE in a
  Mastery set. Read each member's kind before choosing which symbol facts to use.
- **Process slip (our own):** a shell fallback chain around `gm.py submit` sent the same `format`
  submission twice, burning one attempt with identical `incorrect` responses; see the day overview.

## Takeaways

Technique to search for: **cumulative recap bundle** (23 days of specimens, `dNN_analysis_*`
question IDs, "central-directory order" preamble). When a theme-heavy prompt sits on a question
list like this, read the question IDs and formats first; they say what the challenge really is.
For the answers, classify by magic bytes, pull every fact in a single inspector pass, and, since
`incorrect` is batch-level only, bisect by field family (`suffix` groups) rather than guessing. Check
spelling traps (`JAR`, `i386`, `AMD64` on PE) per field kind. What we would do faster: start from the
previous day's validator-confirmed mapper and add only the new day's block, and submit Time Trial
and Mastery in one call once that mapper has gone CORRECT, which is how both of them went.

## Evidence

Leaf: `challenges/day23/binary-reverse-engineering/binary-reverse-engineering-day-23-anti-analysis-recognition/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/accepted_map.json`, `single-solve/artifacts/submission-attempt-log.json`
  (16 calls, last CORRECT; no `result.json` was saved for this mode)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json` (1 call, CORRECT),
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/artifacts/submission-attempt-log.json`
  (1 call, CORRECT), `mastery/ranked-1/result.json`
- `tools/ranked/solvers/binary-reverse-engineering-day-23-anti-analysis-recognition/solve.py`
- Library entries: BRE-INSPECT-B101 (binary inspector), BRE-RECAP-B105 (cumulative recap bundles),
  BRE-QFACTS-B107 (question-fact resolver)
- `progress.md` (grep `^| 23 .*anti-analysis-recognition`)
