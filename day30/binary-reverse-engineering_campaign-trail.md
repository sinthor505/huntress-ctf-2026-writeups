# Campaign Trail

| | |
|---|---|
| Day / Lane | 30 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-30-multi-binary-campaign` |
| Technique | static container classification plus a trailing XOR-coded JSON record (HGRF) in Neko bytecode files |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Campaign Trail" sounds like a multi-binary graph reconstruction, but nothing had to be reversed.
The new Day 30 block is three Neko bytecode files (`.n`, magic `NEKO`) that each end in a
CRC32-protected, XOR-coded JSON record. Every new field is either a container classification or one
property read out of that JSON. We answered all 420 fields offline with an extended library reader.
Single Solve was scored by a teammate before our first submission, so we have no graded result for
it. Time Trial and Mastery went CORRECT on our own attempts.

## The challenge

Prompt: "One binary is a clue; a campaign is a story. Reconstruct the loader, library, and config
graph across the whole set and map the operation end to end."

Delivery was one `evidence.zip` (Single Solve, Time Trial) and 420 answer IDs. Mastery delivered 14
ZIPs (14 sets, 5,880 IDs). The 420 IDs are the recap blocks `d01`-`d29` (14 fields per day,
re-asked from earlier days) plus a `d30` block of 14 fields:

- `d30_artifact_sha256_1..3`: SHA-256 of the three `d30--` members in ZIP central-directory order.
- `d30_analysis_{loader,library,config_capsule}_format`: enum
  `elf | pe | pe_cli | jar | wasm | neko_bytecode`.
- `d30_analysis_config_{campaign,domain,mutex,url}`: "use `loader.n`'s decoded HGRF
  `graph.config` object after confirming all three canonical campaign graphs match when
  `this_role` is omitted".
- `d30_analysis_config_capsule_graph_config_{campaign,domain,mutex,url}`: the same object, read from
  `config.n`.

Everything is `exact` normalization; the text fields are `exact_text`, case-sensitive, "without JSON
quotes".

## Solution

1. **Recon.** The `d30--` members are `d30--sample-{loader,library,config}.n`. Each starts with the
   magic `NEKO`. `bre_inspect` reports no known container for them, so they fell through to
   "unknown", and the enum answer is `neko_bytecode`. That token had been in the answer-format list
   since Day 24 without ever being the right answer.
2. **Finding the graph.** The prompt describes the carrier: scan backward from the end of the member
   for the unique CRC32-valid HGRF record. It uses the same 12-byte layout as the HCFG records from
   earlier days (magic, version byte `1`, XOR key byte, little-endian u16 length, little-endian CRC32
   of the decoded payload), followed by the XOR-coded UTF-8 JSON payload. The JSON has schema
   `huntress.binary.campaign_graph.v1`.
3. **The "graph".** The three members' graphs differ only in `this_role`. The `graph.config` object
   (`campaign`, `domain`, `mutex`, `url`) is identical across them, which is what the "confirm all
   three canonical graphs match" sentence asks you to check. So the `loader.n` and `config.n`
   families carry the same four values, and a reader that checks equality after dropping `this_role`
   is enough. No Neko bytecode is ever disassembled.
4. **Implementation.** We extended the library's `question_facts.py` with a `neko` member kind,
   an `hgrf_graph()` / `h_hgrf_property()` pair that picks the member from "basename ends `X.n`" or
   "`X.n`'s decoded HGRF" and walks a dotted property path, and a `config_capsule_format` ->
   generic `format` mapping. The solver (`cand-solo/solve.py`) is a thin wrapper that loads the start
   response and ZIP(s), calls the library resolver, lints the values against each question's
   answer format, and emits alternate-reading groups. No per-ID tables. 420/420 answers, key set equal
   to the answer IDs.
5. **Recap blocks.** The `d01`-`d29` fields were already covered by the library reader. A regression
   run over Days 27-29 against saved accepted maps (30,458 exact, 0 mismatch) was the check before
   the live attempts.

## Time Trial and Mastery

Single Solve returned `already_solved` on its first submission (the start response already carried a
teammate's solved-by readout), so we stopped submitting there and went straight to Time Trial and
Mastery via `retry`, which issues fresh attempts regardless.

- **Time Trial** (untimed regular attempt): CORRECT in 4 calls. First the 417 non-alternate answers,
  then one call per alternate group (three singleton groups), all at variant 0. Total 6.7 s from
  issue to CORRECT. A later ranked attempt (80 s window, hall-of-fame eligible) was also CORRECT in 4
  calls, 9.7 s; the lane log shows these as the same mapper and the same variant-0 choices.
- **Mastery** (untimed regular attempt): 14 sets, 5,880 IDs, CORRECT in one call with every group at
  variant 0, 16.6 s total. The ranked attempt (665 s window) was also one call, 16.2 s. Per-set solve
  time was 2.4-2.8 s.

The Mastery fixtures needed no generalization: the Neko triple had the same layout in every set, and
the Single Solve mapper was used unchanged.

## Dead ends and gotchas

- **The title and prompt are flavour.** "Reconstruct the graph across the whole set" invites building
  something. The fields only ask for container tokens and one property of one JSON object.
- **Mutex backslash.** The `mutex` value looks like `Global\Huntress<hex>`. The decoded JSON text
  has one backslash, and that single-backslash text was accepted at variant 0 in Time Trial and in
  every Mastery set. The JSON-escaped (doubled) spelling was queued as an alternate and was never
  needed; submit the single-backslash form first.
- **Crystal symbol count (recap field `d13_analysis_crystal_symbol_count`).** `SHT_SYMTAB` only versus
  `SHT_SYMTAB` plus dynamic symbols was an open question from Days 28 and 29. Both readings were
  queued as an alternate group; the symtab-only primary was accepted. (Our regression tool had assumed
  a fixed alternate index per day and showed false mismatches until it was changed to read the
  accepted choice from each leaf's `result.json`; that is a tooling note, see the Day 30 overview.)
- **Single Solve could not be graded for us.** The submission returned `already_solved`, and the
  offline map for that mode was never validated by the Game Master. Its correctness is inferred from
  the identical Time Trial and Mastery results, not from a graded Single Solve.
- **Not exercised.** A graph whose members disagree beyond `this_role`, a `.n` with several
  CRC-valid HGRF records (the backward scan picks the last), and a member with no HGRF at all
  (raises an unresolved error in our reader) never appeared. Treat those branches as untested.

## Takeaways

- Technique names to search for: **Neko bytecode (`NEKO` magic) classification**, **HGRF trailer**
  (HCFG-layout XOR+CRC32 JSON record, found by scanning backward for the last CRC-valid record).
  A new carrier tag with the same 12-byte layout needs no new code; only a different layout does.
- For a "campaign / graph / map the operation" prompt, read the question wording first. If every
  field is a container token or a single property path, it is a parser job, not a reversing job.
- Build the reader once from the prompt's own description of the record layout and a dotted JSON path
  walker, then answer by field family instead of per ID. That is why one solver covered 420 IDs and
  all 14 Mastery sets without edits.
- After the base answers are accepted, submit alternate-reading groups one call each so a rejection
  names the exact field.

## Evidence

Leaf: `challenges/day30/binary-reverse-engineering/binary-reverse-engineering-day-30-multi-binary-campaign/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
  (reason `start_already_correct`)
- `single-solve/artifacts/cand-solo/solve.py` (thin wrapper over the library reader)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`, `mastery/ranked-1/result.json`
- `challenges/day30/_run/librarian/binary-reverse-engineering-day-30-multi-binary-campaign.md`
- `challenges/day30/_run/lane-binary-reverse-engineering-day-30-multi-binary-campaign.log`
- `library/binary-reverse-engineering/` entries BRE-QFACTS-B107 (question facts, Day 30 row),
  BRE-PLAYBOOK-001 ("Day 30 evidence"), BRE-ROLEPAIR-B108 (Day 30 addendum)
- `progress.md` (grep `^| 30 .*multi-binary-campaign`)
