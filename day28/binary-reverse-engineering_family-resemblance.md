# Family Resemblance

| | |
|---|---|
| Day / Lane | 28 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-28-family-clustering` |
| Technique | cumulative recap bundle with role-named members (`family-a`, `family-b`, `false-friend`); static container and HCFG-record reads |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt asks for behavioral clustering of related binaries and a spot-the-false-friend judgement. No question needs any of that. All 392 answers are static reads: SHA-256s, container classification, ELF `e_machine`, and one JSON property out of an XOR-obfuscated `HCFG` config record. The "family" idea shows up only as three member names in the new `d28` block. The Single Solve was scored by a teammate before we submitted, so we have no graded result for it. Time Trial and Mastery were ours, and each went CORRECT.

## The challenge

Prompt: "These binaries might be related — or they might just look alike. Cluster them by behavior, watch out for false friends, and sort the family from the strangers."

- **Delivery:** one `evidence.zip` (Single Solve, Time Trial) or twelve of them (Mastery, one per set). Members are grouped in blocks `d01--` to `d28--`.
- **Questions:** 392 answer IDs per bundle, 28 blocks of 14 fields. Mastery was 12 sets, 4,704 IDs. Blocks d01-d27 re-ask earlier days' fields. Only block `d28` is new.
- **Formats:** answers are SHA-256 hex, base-10 integers, enum tokens (`elf`, `pe`, `pe_cli`, `jar`, `wasm`, `neko_bytecode`) or `exact_text`.
- **Timers:** Time Trial ranked window 80 s, Mastery 570 s. Single Solve was untimed.

The `d28` block has 14 fields over three ELF members whose basenames end `family-a.elf`, `family-b.elf` and `false-friend.elf`:

- `d28_artifact_sha256_1..3`: SHA-256 of members 1-3 in ZIP central-directory order.
- `family_a_*`, `family_b_*`, `false_friend_*`: `format` (container enum) and `machine` (the ELF `e_machine` as an integer).
- `config_{campaign,domain,mutex,url}`: "decode `family-a.elf` and `family-b.elf`, require their config objects to be byte-for-byte equal, then use the common value; do not use `false-friend.elf`".
- `false_friend_config_campaign`: the same read, but on the false friend's own record.

## Solution

1. **Read the questions, not the prompt.** Nothing asks for a similarity score, a cluster or a behavioral comparison. Each field names a member by basename suffix and asks for a container fact or one JSON property. The earlier role-pair days (JAR pairs on Days 24-25) had the same shape, so we did not analyze the binaries' behavior.
2. **Reuse the per-field resolver.** The library's `question_facts` resolver already answered 358 of the 392 IDs unchanged (d01-d27 recap blocks and the d28 SHA-256s). We wrapped it in an adapter that handled the remaining 34 fields: structural counts and ordered-set hashes, symbol and class counts, runtime predicates, PK19 overlay keys, and the new `d28` role fields.
3. **Role members.** For `d28` the member is selected from the question text (`ends family-a.elf`, and so on), not from file order. The pair-config question takes both `family-a` and `family-b`, reads each one's HCFG record, and flags a review if the two differ. They agreed. The false friend's config was read from its own member.
4. **HCFG record.** Scan backward from the end of the member for ASCII `HCFG`. The layout is: byte 4 version, byte 5 XOR key, bytes 6-7 payload length (LE), bytes 8-11 CRC32 of the decoded payload (LE), payload from byte 12. XOR the payload with the key, check CRC32, parse the UTF-8 JSON, and return the requested top-level string. The question's "first CRC32-valid record scanning backward" rule is there to disambiguate extra or decoy records; we did not see one here.
5. **False friend.** It is also an x86-64 ELF (machine 62), so the `format` and `machine` fields do not separate it from the family. Only its config differs.
6. **Submission.** The map is split into a base set plus two small alternate-spelling groups, submitted as one full call per attempt.

## Time Trial and Mastery

The same two solvers, independently written, produced identical maps for each delivery, so nothing had to change.

- **Time Trial, regular (untimed) attempt:** four calls. The 2-ID `hex` group (`d19_analysis_packing_key_1_hex` / `_2_hex`) was accepted at variant 0, and the 389-ID base set was accepted. The single `d13_analysis_crystal_symbol_count` field then came back `incorrect` at the first variant and `CORRECT` at the second (see below).
- **Time Trial, ranked attempt (80 s window):** one call, all 392 answers, `CORRECT`, 9.45 s from issue to result locally.
- **Mastery, regular and ranked:** twelve `evidence.zip` sets, 4,704 IDs. Each solved set by set with the same adapter, merged into one submission. Both attempts went CORRECT in one call (ranked: 20.31 s locally in a 570 s window).

## Dead ends and gotchas

- **Symbol count: `.symtab` only versus `.symtab` + `.dynsym`.** `d13_analysis_crystal_symbol_count` counts symbols matching a Crystal-runtime pattern. Our primary reading counted `.symtab` plus `.dynsym` (1255), which the validator rejected. The second variant, `.symtab` only (771), was accepted. It is the only field where the two readings differ on these members, so the other count fields could not tell us which rule the validator uses. Later days re-confirmed the `.symtab`-only reading (see Takeaways).
- **Packing-key order (`d19_analysis_packing_key_1_hex` / `_2_hex`).** The adapter carried a swapped-order alternate in case key 1 and key 2 were reversed. It was never needed. The first reading (`k1` at overlay offset +5, `k2` at +6) was accepted.
- **Single Solve gave no validation.** The map we built offline for the Single Solve was never graded for us, because the submission returned `already_solved`. The reading choices above were confirmed by Time Trial and Mastery, not Single Solve.
- Our own automation hit nothing worth noting on this challenge.

## Takeaways

- **Titles and prompts in this lane are flavor.** "Cluster by behavior" produced no behavioral question. Open `start_response.json`, group the IDs by `dNN_` prefix, and find the new block. Here that was 14 fields on three role-named members.
- **Role-named members** (`family-a` / `family-b` / `false-friend`) are resolved by basename suffix. This recipe is the same one used for the vulnerable/corrected JAR pair earlier and for the O3/O0 pair on Day 29. Build it once and each new day is a rename.
- **Pair-config questions** (decode both, require equal) are safe to answer from either member once you check they agree. Keep an "unequal" guard because it flags a modelling gap.
- **Contested readings need an alternate group.** The `.symtab`-only versus `.symtab`+`.dynsym` choice cost one extra call, because only a small, isolated group tells you which spelling the validator wants. Later days adopted `.symtab`-only as the default.

## Evidence

Leaf base: `challenges/day28/binary-reverse-engineering/binary-reverse-engineering-day-28-family-clustering/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json` (the `already_solved` marker)
- `single-solve/artifacts/cand-adapter/solve.py` (adapter over `question_facts.py`), `single-solve/artifacts/cand-rubric/solve.py` (second, independent solver)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json` (the `count` and `hex` groups), `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`
- Challenge-level: `challenges/day28/_run/lane-binary-reverse-engineering-day-28-family-clustering.log`
- `library/binary-reverse-engineering/question-facts.md` and `lane-evolution-and-playbook.md` (Day 28/29 evidence), `role-member-pairs.md`
- `progress.md` (grep `^| 28 .*family-clustering`)
