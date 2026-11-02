# Where It All Began

| | |
|---|---|
| Day / Lane | 28 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-28-where-it-all-began` |
| Technique | raw disk image partition table parsing (GPT, with MBR/EBR fallback) |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 28 added a new metadata family, disk images, and one new field: `disk.partition_start_lba_sequence`,
the start sector of every partition as a compact JSON array. The library had no disk reader, so we wrote
a stdlib GPT/MBR parser. The Single Solve was scored by a teammate within seconds of unlock, so our own
submission came back `already_solved` and graded nothing. Time Trial and Mastery (fresh `retry` attempts)
went CORRECT with that reader, the Mastery in a single 396-field call.

## The challenge

The prompt: "Disk images hold every previous artifact family in context. Recover partition layouts,
locate evidence by offset, and prove which image was rebuilt from an incomplete source." Nothing in the
graded questions actually asked for an offset search or for a "rebuilt" image; those phrases were flavor
(see Dead ends).

- **Single Solve:** one question, `d28-disk-001-disk-partition-start-lba-sequence`, "Determine
  disk.partition_start_lba_sequence for disk-001.img", format `json` (compact array, exact values and
  order), a bundle ZIP (`metadata-day-28-evidence.zip`) holding `disk-001.img`.
- **Time Trial:** `evidence.zip` plus 33 questions: the cumulative recap of every earlier day's field
  (roster/evidence-note hashes and sizes, JPEG/PNG/PDF/ZIP/PE/.NET/MSI/LNK/OOXML/email/audio/video/ELF/
  Mach-O/APK/JAR/SQLite/EVTX/registry/prefetch/pcap facts), the new disk field, and one synthesis question
  (order six named files by SHA-256 digest). Ranked window 50 s.
- **Mastery:** 12 sets of the same 33 questions (396 answer ids, `setN.` prefixed), ranked window 385 s.

## Solution

1. **Read the disk field's shape.** The answer is the start LBA of each partition as `[a,b,c]`, ints
   only. Sector size 512.
2. **GPT first.** A protective-MBR entry of type `0xEE` (or `EFI PART` at LBA 1) means GPT. The header at
   LBA 1 gives the entry-array LBA (offset 72), entry count (80) and entry size (84); each entry's first
   LBA is the u64 at offset 32 of the entry. All-zero type-GUID slots are skipped.
3. **Fallbacks.** If the primary header or array is unusable, use the backup header in the last sector.
   If there is no GPT, parse the four MBR entries (start LBA = u32 at +8) and follow an extended
   container (`0x05/0x0F/0x85`) through the EBR chain for logical partitions.
4. **Ordering.** The accepted answer is the starts in partition-table order. We also registered the
   ascending-sorted order (and, for MBR, a variant without the extended container) as alternates in case
   table order was wrong; none was needed, because in every graded image table order was already
   ascending.
5. **Wire it into the existing solver.** The reader was bolted onto the metadata toolkit's
   `question_facts` contract (a `disk` kind registered with `metaread.identify`/`READERS`), so every
   earlier day's field went through unchanged code. The Single Solve candidate produced `[34]` (a
   protective MBR plus one GPT partition at LBA 34); that map was never graded for us.
6. **Submit.** Time Trial and Mastery each took the plain primary variant for every field.

```python
# GPT entry: start LBA is the u64 at +32; skip all-zero (unused) slots
for i in range(h["entries"]):
    e = d[base + i * h["entry_size"]: base + (i + 1) * h["entry_size"]]
    if not any(e[:16]):
        continue
    start, end = struct.unpack_from("<QQ", e, 32)
    out.append({"index": i, "start": start, "end": end})
```

## Time Trial and Mastery

- **Fixtures were bigger than the Single Solve.** The Single Solve had one image with one partition;
  the Time Trial and Mastery bundles carried a disk with 1-6 partitions per set (for example
  `[72,84,133]`, `[40,147,294,336,384,436]`), plus the full cumulative recap of Days 4-27 fields. The
  Day 27 pcap packet-count field was one of them, so the earlier day's reader had to still resolve.
- **Time Trial (regular rehearsal):** 32 fields accepted in the first call, then the pcap packet-count
  field was submitted as its own alternate group and returned CORRECT, 2 calls. The ranked attempt
  (`run_for_time`, 50 s window) took one call, 3.89 s end to end locally.
- **Mastery:** one call of 396 answers CORRECT for both the regular (untimed) and the ranked attempt
  (385 s window, 5.15 s locally). The Mastery set manifests varied (the file named by the
  `evidence-note-filename` / `roster-filename` / `evidence-note-copy-filename` questions differs per
  set), so extraction resolved files from each set's own manifest.
- Time Trial and Mastery each also issued a fresh attempt via `retry` after the first `start` showed the
  team's CORRECT readout; both were graded on their own merits.

## Dead ends and gotchas

- **Single Solve was un-gradable for us.** The one submission returned `already_solved`; nothing was
  scored. The offline `[34]` is therefore unvalidated by the Game Master, and the validator's preference
  between table order and sorted order was never separated by an out-of-order table (every graded
  image was ascending). If a fixture ever carries an out-of-order GPT, table order is an unconfirmed guess.
- **The prompt's "locate evidence by offset" and "rebuilt from an incomplete source" never became
  questions.** The only new field was the partition start sequence. We could not determine from the
  files what the "rebuilt" wording referred to; the backup-header fallback it hints at was never needed
  (every graded image had an intact primary GPT).
- **A second independent reader failed on file naming, not on the disk.** The other candidate solver
  raised `unresolved question roster-filename` / `evidence-note-filename` on the Time Trial and all 12
  Mastery sets (member names are hex-tagged like `roster-e00e66.csv`; the log does not state the root cause, but it
  matches the hex-stem naming trap seen on Day 24), so only the first candidate was submitted. See the day
  overview for the automation side.
- The Time Trial pcap packet-count field was split into its own group by the submission planner; we did
  not determine why from the logs, and its primary variant was accepted.
- No answer-format surprises on the disk field: a compact JSON array of ints with no spaces.

## Takeaways

- **Technique: GPT/MBR partition-table parsing from raw bytes**, search term "partition_start_lba",
  `EFI PART`, protective MBR `0xEE`. GPT entry start LBA is the u64 at +32; MBR start is the u32 at +8;
  follow EBRs for logical partitions, with the link entry relative to the container start.
- Keep the metadata reader set cumulative: a new day's Time Trial/Mastery re-asks every earlier field,
  so every reader must still resolve (check with a regression over saved accepted fields).
- A new field that arrives alone in the Single Solve (here one disk question) is a cheap place to build a
  reader before the larger Time Trial/Mastery bundles; when the Single Solve is team-scored, the first
  real grading comes from the Time Trial, so the reader needs alternates ready.
- Faster next time: register sorted-order and primary-vs-backup alternates up front, which we did, and read
  the manifest per set instead of any hard-coded member name.

## Evidence

Leaf base: `challenges/day28/metadata-forensics/metadata-forensics-day-28-where-it-all-began/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-adapter/solve.py` (disk reader wrapper), `single-solve/artifacts/cand-adapter/OUT.json`
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`
- `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- `challenges/day28/_run/lane-metadata-forensics-day-28-where-it-all-began.log`
- `library/metadata-forensics/disk-partition-lba/README.md` (META-DISK-PARTITION-LBA-001)
- `progress.md` (grep `^| 28 .*where-it-all-began`; rows for Time Trial and Mastery only)
