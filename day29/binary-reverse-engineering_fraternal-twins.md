# Fraternal Twins

| | |
|---|---|
| Day / Lane | 29 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-29-variant-diffing` |
| Technique | Static ELF parsing of an O3/O0 build pair; reuse of a cumulative "recap" question bank |
| Single Solve | Independent (2 submission calls) |
| Time Trial | Independent (regular rehearsal 11 calls, ranked attempt 1 call) |
| Mastery | Independent (regular and ranked, 1 call each) |

## Summary
The prompt promised a diff of two builds of the same source, but the new part of the challenge was
tiny: eight answer IDs about an `-O3` and an `-O0` ELF. The other 398 IDs per set were recap blocks
re-asking every earlier day's questions. A parser library built for earlier days answered all of it
offline, and the only real ambiguity was a single symbol-count field.

## The challenge
Prompt: "Two binaries, same source, different compiler moods. Diff the variants, account for every
optimization, and prove they're semantically the same program."

Single Solve delivered one `evidence.zip` and 406 answer IDs. They fall into 29 blocks (`d01` to
`d29`) of 14 questions each, and each block is scoped to the ZIP members tagged with its own day
prefix. The `d01` to `d28` blocks repeat earlier days' questions about their old artifact families:
hashes, ELF/PE header fields, symbol counts, HCFG configuration records, structural counts. Only the
`d29` block was new:

- `d29_artifact_sha256_1` and `_2`: SHA-256 of the two members in ZIP central-directory order.
- `d29_analysis_{optimized,unoptimized}_format` and `_machine`: container class and `e_machine`
  of the member whose name ends `O3.elf` or `O0.elf`.
- `d29_analysis_{optimized,unoptimized}_config_{campaign,domain,mutex,url}` and the un-prefixed
  `d29_analysis_config_*` forms. Both role members carry an HCFG record (magic, version, XOR key,
  length, CRC32, XOR-encoded JSON payload). The un-prefixed questions say to decode both members
  and require the configs to be byte-for-byte equal before answering.

Time Trial used the same 406 IDs on a fresh artifact (80 s ranked window). Mastery was a bundle of
13 sets (13 ZIPs, 5,278 IDs, set-prefixed as `setN.d01_...`, 620 s ranked window).

## Solution
1. Recon: check the ZIP against the start response's SHA-256, count the IDs per `dNN` block, and
   note that 28 of 29 blocks are old questions.
2. Run the library's `question_facts` reader (a field-name-to-handler resolver over an
   ELF/PE/JAR/WASM parser) on the whole question list. It resolved 398 of 406 IDs unchanged. The
   unsupported eight were the new d29 role fields.
3. Extend the Day 28 adapter (a thin wrapper over `question_facts`) with the new roles. Day 28 had
   picked members by a suffix such as `family-a.elf`. The only change was to recognize
   `O3`/`O0` in the question text and look up the unique member whose basename ends that way:

   ```python
   m = re.search(r"ends (family-a|family-b|false-friend|O3|O0)\.elf", text)
   if m:
       hits = [x for x in mem if x.name.lower().endswith(m.group(1).lower() + ".elf")]
   ...
   for role in ("false_friend", "family_a", "family_b", "optimized", "unoptimized"):
       H[f"{role}_format"] = qf.h_format
       H[f"{role}_machine"] = qf.h_machine
       for k in ("campaign", "domain", "mutex", "url"):
           H[f"{role}_config_{k}"] = qf.h_config(k)
   ```
4. The HCFG decode is the library's backward scan for `HCFG` magic, XOR with the key byte, CRC32
   check, then JSON parse. In the Single Solve artifact the O0 and O3 configs were equal, so the
   shared and per-role answers coincide (campaign `BR29-000`, domain
   `variant-000.day29.example.invalid`). Both builds are x86-64 ELF (`e_machine` 62), so format is
   `elf` and machine is `62` for both roles.
5. The adapter emitted 406 of 406 answers, passed the format lint, and declared one alternate group.
   Total solver time was about 3.5 s. Submission was a probe of the one alternate-group field alone
   followed by the remaining 405 IDs. Single Solve went CORRECT on the second call.

Despite the title, no function-level diff or decompilation was done. Every d29 question is a
container-level fact, a hash, or an embedded config record, and the "semantically the same program"
requirement is carried by the config-equality check.

## Time Trial and Mastery
- Time Trial: the same 406-ID shape on a new artifact. The regular (unranked) rehearsal took 11
  calls, almost all of it the d13 field below. The first full submission was `incorrect`, the
  call was split into eight chunks of 37 to 69 IDs, all `accepted`, and the leftover
  symbol-count field was flipped to its alternate reading. The ranked attempt then went CORRECT
  in one call (6.6 s locally, with the alternate already chosen).
- Mastery: 13 deliveries, `setN.`-prefixed IDs, 5,278 answers. The same adapter ran once per set and the
  merged map went CORRECT in a single 5,278-ID call in both the regular and ranked runs (about 14 to
  15 s local total, 620 s ranked window). No set needed new handling.

## Dead ends and gotchas
- **One contested field: `d13_analysis_crystal_symbol_count`.** The question counts names matching
  `/Crystal|crystal|__crystal|GC_/` "from ELF symbol-table entries with nonempty resolved names".
  The adapter's primary reading counted `.symtab` plus `.dynsym` (1255 in the Single Solve artifact).
  The alternate counted `.symtab` only (771). In Time Trial the symtab+dynsym reading was `incorrect`
  and the symtab-only reading was `CORRECT`. Day 28's Time Trial agreed. The library now counts
  `SHT_SYMTAB` only.
- **Single Solve oddity.** The Single Solve log shows the isolated d13 probe returning `incorrect`,
  yet the 405-ID submission without that field returned CORRECT, so the Single Solve grader evidently
  did not require it. We could not determine from the files why. The saved
  `accepted_plan.json` therefore still lists the rejected 1255 as that field's primary.
  Read the log's winning variant index, not the plan file, when reusing it.
- A composite `incorrect` carries no per-field signal here, so the Time Trial diagnosis had to
  bisect by chunk and then by single ID. Isolating the one uncertain field first would have saved
  most of the 11 calls.
- Hyphenated role names need underscore field prefixes (`false_friend_format`). The `dNN_structural_*`
  IDs were not in the reader's ID grammar and needed a parser patch.
- The d29 pair's configs were equal in every delivery seen. The branch for unequal configs (where
  "require equal" would have to fail somehow) was never exercised.

## Takeaways
- On a late-event day, count the new IDs before reading the prompt. A "diff the variants" title
  concealed an 8-field addition on top of a 398-field regression test of earlier work.
- Keep one cumulative parser with a field-name handler registry (`question_facts`), so a new day
  only adds role-selection logic. That turned a challenge nominally about compiler optimization into a few
  seconds of offline work and a 295-line adapter, most of it inherited.
- For count fields over symbol tables, `.symtab` only is the validator's reading (the Day 28 and 29
  Time Trials both agree). Put the alternate reading in a variant group and test that field alone.
- For multi-set Mastery bundles, resolve per set and merge. Sets here were uniform, but earlier days
  varied member manifests per set.

## Evidence
- `challenges/day29/binary-reverse-engineering/binary-reverse-engineering-day-29-variant-diffing/single-solve/journal.md`
- Solver: `.../single-solve/artifacts/cand-adapter/solve.py`; graded plan: `.../single-solve/accepted_plan.json`
- Time Trial: `.../time-trial/regular/result.json` (11 calls), `.../time-trial/ranked-1/result.json` (1 call)
- Mastery: `.../mastery/regular/result.json`, `.../mastery/ranked-1/result.json` (1 call each)
- Submission rounds: `challenges/day29/_run/lane-binary-reverse-engineering-day-29-variant-diffing.log`;
  post-solve notes: `challenges/day29/_run/librarian/binary-reverse-engineering-day-29-variant-diffing.md`
- `progress.md` rows for Day 29 `binary-reverse-engineering-day-29-variant-diffing` (3 rows)
