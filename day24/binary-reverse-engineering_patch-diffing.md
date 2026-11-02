# Patch Diffing

| | |
|---|---|
| Day / Lane | 24 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-24-patch-diffing` |
| Technique | cumulative recap bundle; container classification, HCFG config decode from JAR classfile constant pools, role-member pairs |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The title and prompt promise a before/after diff of a vulnerable function and its fixed twin. No
question asks for one. The day's new block is a pair of JARs (`...vulnerable.jar`, `...corrected.jar`)
with 12 static fields (hashes, container type, an embedded config record), added to the 23
earlier days' blocks in the same recap bundle. We ran the existing consolidated resolver, extended it
for three new rules, and all three modes went CORRECT. Single Solve took 6 submission calls (one base
set plus one per risky answer group); Time Trial and Mastery were each accepted in a single `full-prefer`
call on the first run.

## The challenge

Prompt: "Two builds, one fix. Diff the vulnerable function against its corrected twin and pinpoint
exactly what changed - then prove the old one was really broken."

Delivered: one `evidence.zip` (SHA-256 verified against the start response). The Single Solve asked 288
answers: 24 blocks (`d01`..`d24`) of 12 fields each. The `d01`-`d23` blocks were earlier days' formats
(ELF/PE/Mach-O/JAR/Neko/WASM facts, hashes, configs) and the resolver already handled them. The `d24`
block:

- `d24_artifact_sha256_1/_2`: SHA-256 of member 1/2 in ZIP central-directory order.
- `d24_analysis_{corrected,vulnerable}_format`: classify the member whose basename ends
  `corrected.jar` / `vulnerable.jar` (enum: `elf`, `pe`, `pe_cli`, `jar`, `wasm`, `neko_bytecode`).
- `d24_analysis_config_{campaign,domain,mutex,url}`: "decode **both** role members and require their
  config objects to be byte-for-byte equal before using the common value".
- `d24_analysis_corrected_config_{campaign,domain,mutex,url}`: the same four fields from
  `corrected.jar` alone.

The config is an HCFG record, specified in the question: bytes 0..3 `HCFG`, byte 4 version, byte 5 XOR
key, bytes 6..7 little-endian payload length, bytes 8..11 little-endian CRC32 of the decoded payload,
payload from byte 12; XOR with the key, check the CRC, parse the JSON. In a JAR the record sits in a
Base64 constant of at least 80 characters in a classfile's constant pool.

## Solution

1. **Read the questions before the prompt.** The prompt suggests decompilation and a diff, but every
   `d24` field is a static container fact or a decoded config value. Nothing needs the changed function
   or an exploit proof.
2. **Run the existing resolver.** The solver is a thin wrapper around the library's `question_facts.py`
   (a per-field rule resolver driven by the question text). 280 of the 288 IDs resolved unchanged from
   the `d01`-`d23` rules.
3. **Add the three Day 24 rules** to the resolver:
   - *Role hints.* `corrected` and `vulnerable` in the question select the member by basename suffix,
     then the container is classified by magic and parsed structure (both are `jar`).
   - *"Both role members equal" configs.* Decode the HCFG record of every non-`.json` member in the
     block and require the values to agree; the common value is the answer. On the real bytes the two
     JARs carried the same config, so the `config_*` and `corrected_config_*` fields are equal.
   - *Any-classfile constant scan.* The question says "parse every JAR classfile constant pool". Earlier
     JAR days kept the Base64 HCFG constant in the `Main-Class` classfile. Here `Main-Class` is
     `Day24Corrected000` / `Day24Vulnerable000`, but the 508-character constant is in the companion
     `...$.class` classfile (Scala/Kotlin object style). A scan limited to `Main-Class` finds no
     CRC32-valid record, so the filter was dropped for this sentence.
4. **Submit by group.** The base set of 277 fields went in first, then each risky group at its first
   variant: `machine`, `format`, and the two member-order `artifact_sha256` groups (`1`, `2`) and `3`.
   Every group was accepted at variant 0 and the last call returned `CORRECT` (6 calls total). The
   journal's "285 of 288 accepted" is the running count one group before `CORRECT`, not three fields
   that needed an alternate reading.

Values for this fixture, for reference: campaign `BR24-000`, domain `node-000.day24.example.invalid`,
mutex `Global\HuntressBR24_000`, URL `https://node-000.day24.example.invalid/api/BR24-000`, both
formats `jar`.

## Time Trial and Mastery

- **Time Trial:** same 288 questions, a fresh `evidence.zip`. The regular (untimed) run and the first
  ranked run (80 s window) were each CORRECT in one `full-prefer` call, ~11-12 s end to end. Two later
  ranked re-runs went CORRECT in 7 calls (base set, then one call per group, all at variant 0).
- **Mastery:** 8 sets x 288 = 2,304 answers, IDs prefixed `setK.`, 8 evidence ZIPs, ranked window 530 s.
  One call per run, ~20-28 s end to end. Fixtures differ per set, so the extractor is manifest-driven
  rather than keyed to one member name.
- The same resolver and plan served all three modes. The planner also carried alternates (`I386`,
  `0x014c`, `332`, `x86`, `JAR`) that were never submitted.

## Dead ends and gotchas

- **The title is not literal.** No diff, changed function or proof is requested. This is the lane's
  recurring pattern: new-block questions on these days are static facts about the delivered members.
- **PE `AMD64` vs `x86-64`.** The resolver emits `x86-64` for the machine fields; the accepted Mastery
  maps carried `AMD64` for 12 and 10 of those fields in the two ranked runs (1 in a Time Trial). Both are
  the same fact under the field's pattern syntax. Lowercase `i386` for the executable-header machine
  fields on `d22`/`d23` was also accepted.
- **Enum tokens are lowercase.** `jar` was accepted; the `JAR` alternate was never needed.
- **Per-set pairing.** A Mastery delivery was seen as one 288-ID start (unprefixed IDs) listing all 8
  artifacts, and the first resolver version put set 2's ZIP in slot 2, so IDs resolving to set 1 failed in
  7 of 8 sets. Fixed in the library; the self-contained second solver covered Mastery on that first run.
  This was a bug in our own tooling, not a property of the challenge (see the day overview).

## Takeaways

- Read the question text before the prompt; in a cumulative recap bundle the prompt flavor and the
  asked fields can be unrelated.
- **HCFG record in a JAR constant pool:** scan every classfile (including `$.class` companions), not just
  `Main-Class`; Base64 constants of >= 80 characters, XOR key from byte 5, verify the CRC32.
- **Role-member pairs:** pick members by the basename suffix the question names, and check both members
  decode to equal configs before taking the common value. Day 25 reused this shape, as did later pairs.
- Submit risky groups separately from the base set so an `accepted` response localizes the doubt.

## Evidence

Leaf base: `challenges/day24/binary-reverse-engineering/binary-reverse-engineering-day-24-patch-diffing/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
  (the leaf holds no `result.json`; the CORRECT is in the lane log)
- `single-solve/artifacts/cand-adapter/solve.py` (wrapper over `question_facts.py`)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`,
  `time-trial/ranked-3/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `challenges/day24/_run/lane-binary-reverse-engineering-day-24-patch-diffing.log` (submission rounds)
- Library entry BRE-ROLEPAIR-B108 (`library/binary-reverse-engineering/role-member-pairs.md`) and the
  resolver `library/binary-reverse-engineering/question_facts.py` (BRE-QFACTS-B107)
- `progress.md` (grep `^| 24 .*patch-diffing`)
