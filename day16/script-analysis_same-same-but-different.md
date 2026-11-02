# Same Same, But Different

| | |
|---|---|
| Day / Lane | 16 / Script Analysis |
| Challenge ID | `script-analysis-day-16-obfuscation-equivalence` |
| Technique | static reconstruction of inert descriptor markers; descriptor-bank reuse across days |
| Single Solve | Independent (saved submission response reads CORRECT) |
| Time Trial | Independent (saved submission response; later ranked rerun also CORRECT) |
| Mastery | Independent (saved submission response; later ranked rerun also CORRECT) |

## Summary

The prompt promises an obfuscation-equivalence puzzle: strip several scrambled versions of the same
logic down to their semantics, prove which are equivalent, and find the one that changed the plot.
Nothing in the question set asks for that. The 1,095 questions are the Day 15 descriptor bank plus
69 new Ruby and "advanced Bash" questions, so the work was reconstructing each file's embedded
descriptor and mapping its fields to answer IDs. We reused the Day 15 structural mapper unchanged
and all three modes went CORRECT on the first submission.

## The challenge

Prompt: "Several obfuscation families have scrambled the same logic independently, or so they claim.
Strip each version down to its semantics, prove which ones are truly equivalent, and expose the
variant that quietly changed the plot."

Delivery was `evidence.zip` (AES-encrypted, password `infected`) with 123 members across many
families: PowerShell samples, Batch, VBScript, JScript/JavaScript, Node.js, Bash, Python recovered
from packaged forms (py2exe, PyInstaller, cx_Freeze, pyarmor, wheel, zipapp), and "polyglot" and
"host" containers (PHP, Lua, Perl, Ruby, WMI MOF, MSBuild, AutoIt, AppleScript, TypeScript, HTA,
WSF, SCT). Single Solve and Time Trial each had 1,095 answers; Mastery was an outer ZIP of five such
archives (5 x 1,095 = 5,475, ids prefixed `setN.`). Answer kinds: 650 `exact_text`, 167 `enum`,
120 `pattern`, 78 `integer`, 78 `ordered_tokens`, 2 `mitre_attack_id`. All values are submitted as
strings. Every mode started untimed (`deadline: null`).

## Solution

1. **Check the question IDs before reading the prose.** Diffing the ID set against the previous day
   showed Single Solve was a strict superset of Day 15: 1,026 existing IDs plus 69 new ones (44
   `polyglot-ruby-*`, 11 `host-ruby-rack-*`, 14 `bash-advanced-*`). There is no equivalence, diff or
   "which variant changed" question. The same questions per file (`role`, `attack`, `adapter`,
   `campaign`, `url`, `steps`, `stage-count`, `host-profile`, `obfuscation`, `stage-profile`,
   `next-language`) apply to Ruby as to the older languages.
2. **Recover the descriptor in every file.** Each specimen carries an inert marker such as
   `HUNTRESS_ANALYSIS_V1:HB64-<hex>`. For the Ruby launcher we checked, the hex decodes to base64,
   which decodes to a pipe-delimited record:
   `v=3|language=ruby|role=launcher|verdict=suspicious|attack=T1059|adapter=process_builder|host_profile=standalone|obfuscation_profile=hex_base64|stage_profile=three_stage_decode_write_launch|...|steps=...|campaign_id=...|url=...`.
   Other files use other representations and transforms (reversed base64, URL-safe base64,
   `\u00XX` escapes, `%XX` hex, JSON number arrays, XOR/rolling-XOR/add-mod-256 and
   reverse). `static_decode.py` handles these offline and wrote `decoded-descriptors.json`
   (123 members, 0 errors, 0 missing IDs).
3. **Map descriptor fields to answers.** `answer_map.py` mostly copies a field (`campaign_id`,
   `url`, `adapter`, `steps` joined with `>`, and so on). The interesting part is `attack`:

   ```python
   elif suffix=='attack' and (aid.startswith('polyglot-') or aid.startswith('host-')) \
        and f.get('attack') is not None: value=f['attack']
   elif suffix=='attack': value=attack_for(f,family,prefix)
   ```

   Polyglot and host specimens carry their own `attack` field, and that declared value is what we
   submitted. Families with no declared field use the precedence in the question text, via
   `attack_for` (for example `acquire_remote` -> `T1105`, `decode_embedded` -> `T1140`,
   `dispatch_process` -> the language's `T1059.00x` sub-technique). The new `bash-advanced-*` files
   are fixed at `role=advanced_syntax`, `attack=T1059.004`.
4. **Batch and VBScript controls come from structure, not from the descriptor.** The nine
   `batch-*-control` answers were derived by classifying the wrapper around the decode statement
   (one-item `FOR /L ... DO CALL` -> `call_subroutine`, `if defined` guard -> `if_guard`,
   plain `goto` into the decode label -> `goto_dispatch`, and so on). We cross-checked the mapper
   against an independent signature classifier; all nine agreed (5 `call_subroutine`, 3 `if_guard`,
   1 `goto_dispatch`). The mapper carries an empty per-ID override table on purpose.
5. **Submit.** One call with all 1,095 fields, graded CORRECT on attempt 1. The extractor and mapper
   are offline; submission was a separate step.

## Time Trial and Mastery

- **Time Trial** was a fresh archive (different SHA-256, 123 members) with the same IDs, but 318 of the
  1,095 values differed from Single Solve, so nothing carried over as a literal answer map. The same
  unmodified scripts decoded it, no per-ID overrides, CORRECT on attempt 1.
- **Mastery** delivered five AES evidence archives. We built a per-set working directory that mimics
  the single-solve layout (strip the `setN.` prefix, run decode and map per set, merge with the prefix
  restored) and submitted all 5,475 fields in one call, CORRECT on attempt 1.
- Both modes were untimed when first solved. Later ranked reruns on the same solver logic were also
  CORRECT on attempt 1 (Time Trial 8.5 s total, Mastery 7.1 s total; ranked runs were bragging rights
  only).

## Dead ends and gotchas

- **The prompt is misleading.** "Prove which ones are truly equivalent" and "expose the variant that
  quietly changed the plot" describe nothing that is graded. Trusting the question IDs and the
  artifact list over the narrative saved the day.
- **The `attack` rubric is generous but the descriptor wins.** The question text lists alternates
  (the parent `T1059` is accepted where a sub-technique is canonical; a staged chain or relay also
  accepts `T1105`; `none` is accepted for some relays). We did not need any of them: declared
  descriptor values for polyglot/host files and the stated precedence elsewhere were accepted on the
  first call.
- **Batch control answers are not constant across days or modes.** The structural rule held here, but
  the same shape had been `for_once` on another day; classify each file from its own source.
- **Antivirus quarantine on fixtures.** In the later ranked Mastery rerun, Windows Defender
  real-time protection deterministically quarantined at least one extracted script
  (`staged-handoff.js`) the instant its bytes were written, so the first ranked attempt failed
  before submitting. The fix was to decode the archive entirely in memory. This did not affect the
  original solves, which extracted to disk.

## Takeaways

- **Descriptor-bank reuse.** Script-analysis banks in this period were cumulative: before analyzing,
  diff the new question IDs against the previous day's. Here a day-sized "new" challenge was 69 new
  questions on top of a mapper we already had.
- **Declared descriptor beats derived graph rule** for families that carry their own `attack`
  field; derive from the stated precedence only where it is absent.
- **Control-flow wrapper fields are structural.** Classify them from the source with a signature
  classifier and cross-check with the mapper; never reuse another mode's per-ID overrides.
- **Run fixtures that are intentionally obfuscated malware lookalikes in memory** if the host has
  real-time antivirus.

## Evidence

Leaf base: `challenges/day16/script-analysis/script-analysis-day-16-obfuscation-equivalence/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/static_decode.py`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/controls.json`,
  `single-solve/artifacts/submission-attempt-log.json`, `single-solve/artifacts/submission-attempt-01-response.txt`
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json`,
  `time-trial/artifacts/submission-attempt-01-response.txt`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/prep_mastery.py`, `mastery/artifacts/submission-attempt-log.json`,
  `mastery/artifacts/submission-attempt-01-response.txt`, `mastery/ranked-2/result.json`
- Ranked-run solver (in-memory variant): `tools/ranked/solvers/script-analysis-day-16-obfuscation-equivalence/solve.py`
- Library: descriptor-bank reuse (SCRIPT-TECHNIQUE-005), Batch control signature
  (SCRIPT-TECHNIQUE-004), `library/script-analysis/`
- `progress.md` (grep `^| 16 .*obfuscation-equivalence`)
