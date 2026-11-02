# Final Boss

| | |
|---|---|
| Day / Lane | 31 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-31-reversing-gauntlet` |
| Technique | static container classification plus a new HCFG-layout trailer record (`HGAU`) on top of a 30-day cumulative recap |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent (regular rehearsal 7 calls; ranked attempt 1 call) |
| Mastery | Independent (regular 1 call; ranked 1 call) |

## Summary

Day 31 was the event's final 24-hour day, and the Binary lane's "Final Boss" lived up to its name
only in field count, not in difficulty: 434 answer IDs, of which 420 were the same `d01`-`d30` recap
blocks every cumulative BRE day had already answered, and only 14 were new. The new block was one
ELF member ending in a trailing JSON record under a fresh magic (`HGAU`) that reused the exact
12-byte layout of every earlier `HCFG`/`HGRF` record. We generalized the library's record reader
instead of writing a one-off parser, and answered all 434 (and all 6,510 Mastery fields) offline.
Single Solve was scored by a teammate before our first submission; Time Trial and Mastery went
CORRECT on our own attempts, in regular and ranked form.

## The challenge

Prompt: "The gauntlet: everything you've learned, across platforms and languages, in one campaign.
Reconstruct it all, end to end, offline. No hints. Good luck."

Single Solve and Time Trial delivered one `evidence.zip` and 434 answer IDs; Mastery delivered 15
ZIPs (15 sets, 6,510 IDs, `setN.`-prefixed). The IDs fall into 31 blocks of 14 questions each. The
`d01`-`d30` blocks repeat every earlier day's question shapes against their own day-prefixed ZIP
members (hashes, container/ELF/PE header fields, symbol counts, HCFG-family configuration records,
structural counts). Only the `d31` block was new, scoped to a single member `d31--sample.elf`
(ELF64, x86-64):

- `d31_artifact_sha256_1`: SHA-256 of the `d31--` member.
- `d31_analysis_format` (enum `elf | pe | pe_cli | jar | wasm | neko_bytecode`) and
  `d31_analysis_machine` (processor architecture from the executable header).
- `d31_analysis_config_{campaign,domain,mutex,url}` and the root-qualified
  `d31_analysis_gauntlet_config_{campaign,domain,mutex,url}`: the same four properties of the
  decoded record's `config` object, asked twice under different phrasing.
- `d31_analysis_gauntlet_crc32_hex` (header bytes 8..11, as `0x` + 8 lowercase hex digits) and
  `d31_analysis_xor_key_hex` (header byte 5, as `0x` + 2 lowercase hex digits).
- `d31_analysis_gauntlet_trace_branch` (enum `success | decoy | failure`, from the record's
  `trace.branch`).

The record itself: scanning backward from the end of `d31--sample.elf` for the unique CRC32-valid
record under magic `HGAU` gives a 12-byte header (magic, u8 version, u8 XOR key, u16 LE length, u32
LE CRC32 of the decoded payload) followed by XOR-coded UTF-8 JSON, schema
`huntress.binary.reversing_gauntlet.v1`. Its root carries `config{campaign,domain,mutex,url}` and
`trace{branch,...}` — the same shape as Day 30's `HGRF` "campaign graph" record, just under a new
four-letter magic.

## Solution

1. **Recon.** Verified `evidence.zip` against the start response's SHA-256, then counted IDs per
   `dNN` block: 30 blocks of 14 (420 fields, all previously-seen shapes) plus one new 14-field `d31`
   block, 434 total. One new member, `d31--sample.elf`.
2. **Run the cumulative reader unchanged.** `library/binary-reverse-engineering/question_facts.py`
   (built up across Days 24-30) resolved 420 of 434 IDs out of the box — every recap block. The 14
   unsupported IDs were exactly the new `d31` block.
3. **Generalize instead of hardcoding a new record type.** The `d31` record shares Day 30's `HGRF`
   layout byte-for-byte, just under a different magic, so rather than write another one-off decoder
   the library gained three general pieces:

   ```python
   def tagged_record(raw: bytes, magic: bytes = b"HGRF", any_version: bool = False) -> dict | None:
       """A member ends in an HCFG-layout record under another magic (HGRF, HGAU, ...): magic,
       u8 version, u8 XOR key, u16 LE length, u32 LE CRC32 of the decoded payload, then the
       XOR-coded UTF-8 JSON.  Scan backward; the last CRC32-valid record wins."""
       ...

   def h_tagged_record(dl, blk, q, magic: str):
       """Dispatched for any question naming a 'CRC32-valid XXXX record' where XXXX != HCFG.
       Member: from 'basename ends X.ext' or 'end of the ELF/PE/...' text.  Field: a JSON
       property path, header byte 5 (XOR key), header bytes 8..11 (CRC32), byte 4 (version)."""
       ...
   ```

   and a `json_path()` walker that drops a leading path segment naming the schema's own root object
   when that key is absent (so `gauntlet.config.url` resolves the same as `config.url`), and falls
   back to an underscore-joined nested key when a segment isn't present verbatim
   (`gauntlet.trace_branch` → `trace.branch`). `HGRF` questions kept their existing `h_hgrf_property`
   path unchanged; `h_tagged_record` only fires for a different, non-`HCFG` magic.
4. **Decoded values** (from the single CRC32-valid `HGAU` record in `d31--sample.elf`): format `elf`,
   machine `x86-64`, campaign `BR31-E84AAD11`, domain `core-f3dc07c7.day31.example.invalid`, mutex
   `Global\Huntress833FFE72E73B` (one literal backslash), url
   `https://sync-b8db3306.day31.example.invalid/vault/5db45dd33d`, XOR key `0x3b`, CRC32
   `0x9b8d9cd6`, `trace.branch` `success`. The root-qualified `gauntlet_config_*` fields read
   identically to the bare `config_*` fields, as `json_path()`'s root-drop was designed to do.
5. **Regression before any live attempt.** `regress_accepted.py --days 24-30 --discover` against
   saved accepted maps: 55,655 exact + 50 equivalent matches, 1 pre-existing mismatch unrelated to
   this change (a Day 26 field that only ever uses `HCFG`, never exercising the new code path). Day
   30's own Time Trial (420/420) and Mastery (5,880/5,880) still matched exactly.
6. **Submission.** The solo wrapper (`cand-solo/solve.py`, a thin loader over `question_facts.py`)
   produced 434/434 answers, passed the answer-format lint, and its key set matched `answer_ids`
   exactly, in 3.5 s. The first Single Solve submission came back `already_solved` — a teammate had
   already scored this Single Solve — so that offline map was never validator-tested there; we moved
   straight on to Time Trial and Mastery, which we grade ourselves.

## Time Trial and Mastery

- **Time Trial**, regular (untimed) attempt: CORRECT in 7 calls. The solver declared six alternate
  groups (the recap field `d13_analysis_crystal_symbol_count`, three HCFG-family mutex
  doubled-backslash alternates including the new `d31` one, and a machine-label spelling alternate).
  The log shows one alt-group probe, then the 428 non-alternate answers in one call, then one call
  per remaining alternate group, every one accepted at variant 0, total 6.87 s. A later **ranked**
  attempt (80 s window, hall-of-fame eligible) submitted the full 434-answer map "full-prefer" in a
  single call and was CORRECT in 1.46 s (`t_total` 6.46 s including issue/download/solve).
- **Mastery**, regular attempt: 15 sets solved independently (~2.5-2.9 s each), merged into 6,510
  answers, and submitted in one call — CORRECT, 1.81 s, `t_total` 13.78 s. The **ranked** attempt
  (715 s window) repeated this: 15 sets solved in 27.64 s, one 6,510-answer submission, CORRECT in
  2.5 s, `t_total` 37.71 s. No set needed new handling; `chosen` shows every alternate group at
  variant 0 in both regular and ranked Mastery, matching Time Trial.

## Dead ends and gotchas

- **The title oversold the task.** "Final Boss" / "the gauntlet" / "reconstruct it all" implied a
  large cross-binary puzzle, but 420 of 434 fields were a pure regression test of 30 earlier days,
  and the 14 new fields were one more record under the same 12-byte layout the library had already
  modeled twice. Count the new IDs before trusting a late-event title.
- **Mutex backslash.** Like Day 30's `HGRF` mutex, the `HGAU` mutex decodes to a string with one
  literal backslash (`Global\Huntress...`); the single-backslash reading was accepted at variant 0 in
  Time Trial and in all 15 Mastery sets, with the doubled/escaped spelling queued as an unused
  alternate.
- **Carried-over ambiguity.** `d13_analysis_crystal_symbol_count` (symtab-only vs. symtab+dynsym, an
  open question since Day 28) was queued as an alternate group again; the symtab-only primary was
  accepted, consistent with every day since.
- **Mastery's ranked delivery can omit the set prefix for set 1** (a pattern already known from Day
  24): an unprefixed answer-ID block means "this is set 1," not a malformed delivery. The existing
  `pair_deliveries()` helper already handled it, so the Day 31 ranked Mastery bundle needed no
  solver change, but a naive reader that treats unprefixed IDs as an error would have silently
  dropped that set.
- **Single Solve stayed unvalidated.** The `already_solved` submission means the Game Master never
  graded our offline map for that mode. Confidence that it was correct rests on the matching Time
  Trial and Mastery CORRECTs and the regression check against the prior six days' accepted maps, not
  on a graded Single Solve result.

## Takeaways

- Technique to search for: **HCFG-layout trailer record generalization** — `tagged_record()` /
  `h_tagged_record()` / `json_path()` in `question_facts.py` treat the whole family of
  "CRC32-valid XXXX record" trailers (`HCFG`, `HGRF`, `HGAU`, and any future magic sharing the same
  12-byte header) as one dispatch target, driven entirely by the question text's own field
  description. A new magic needs no new code; only a different byte layout would.
- On a cumulative recap day, check the new-field count first. Day 29 and Day 30 both added 8-14 new
  fields out of 400+; Day 31 followed the same ratio despite the "final boss" framing.
- `json_path()`'s root-name-dropping (`gauntlet.config.url` ≡ `config.url`) and underscore-joined
  fallback (`gauntlet.trace_branch` → `trace.branch`) let one reader answer both a bare property path
  and a root-qualified one without a day-specific special case.
- Run the regression tool against several prior days' accepted maps before the first live submission
  on a recap day — that is what let us trust the offline map when Single Solve itself could not be
  graded.

## Evidence

Leaf: `challenges/day31/binary-reverse-engineering/binary-reverse-engineering-day-31-reversing-gauntlet/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
  (reason `already_solved`)
- `single-solve/artifacts/cand-solo/solve.py` (thin wrapper over the library reader)
- `time-trial/regular/result.json` (7 calls), `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json` (1 call)
- `mastery/regular/result.json` (1 call), `mastery/regular/accepted_plan.json`,
  `mastery/ranked-1/result.json` (1 call)
- `library/binary-reverse-engineering/question_facts.py` (`tagged_record`, `h_tagged_record`,
  `json_path`); `library/binary-reverse-engineering/regress_accepted.py` (the Days 24-30 check)
- `challenges/day31/_run/lane-binary-reverse-engineering-day-31-reversing-gauntlet.log` (submission
  rounds); `challenges/day31/_run/librarian/binary-reverse-engineering-day-31-reversing-gauntlet.md`
  (post-solve note)
- `progress.md` (grep `^| 31 .*reversing-gauntlet`)
