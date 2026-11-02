# Packet Loss Prevention

| | |
|---|---|
| Day / Lane | 27 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-27-packet-loss-prevention` |
| Technique | classic pcap record walk by `incl_len`, counting snaplen-cut ("lost") packets |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 27 added exactly one new field to the metadata bundle: `pcap.packet_count`, the number of
packets in a `capture-001.pcap` member. Every capture in the day was built so that some records
were snaplen-cut (`incl_len < orig_len`), which is the "packet loss" of the title. The accepted
reading was the plain record count, cut packets included, walked by `incl_len`. Time Trial and
Mastery went CORRECT on the first variant we tried. The Single Solve was scored by a teammate
within seconds of unlock, so ours was never graded.

## The challenge

The prompt, identical in all three modes: "Packet captures, exports, and host artifacts describe
the same incident from different angles. Count traffic, reject malformed captures, and find the
host narrative missing network evidence." Despite that wording, only the count was ever asked.
The "reject malformed captures" and "host narrative" clauses never produced a question.

- **Single Solve:** one question, `pcap.packet_count` for `capture-001.pcap` inside
  `metadata-day-27-evidence.zip`. Format: base-10 integer, `trim` normalization.
- **Time Trial:** a 32-field recap bundle (`evidence.zip`). Of the 32, 31 were fields from earlier
  days: roster/evidence-note hashes and sizes, JPEG/PNG/PDF/ZIP, PE, .NET MVID, MSI, LNK, OOXML,
  EML, WAV/MP4, ELF, Mach-O, APK, JAR, SQLite, EVTX, registry hive, prefetch, and a SHA-256
  ordering synthesis. The 32nd was the new pcap count.
- **Mastery:** 11 evidence sets, 32 fields each (352 answer ids, `setN.` prefixed), delivered as an
  outer ZIP of `evidence-NN.zip` members.

## Solution

1. **Recon.** The Single Solve capture was 119 bytes: a classic little-endian, microsecond pcap
   (version 2.4, snaplen 65535, linktype 147/USER0) with three records. Record headers read
   `incl/orig` = 16/16, 21/21 and 10/64. The file ended exactly after record 3.
2. **The twist.** Record 3 is a packet cut short by the capture's snaplen (10 bytes kept of 64).
   It is still one packet. The risk was in the reading: counting only untruncated records
   (`incl == orig`) would give 2, while a plain record count gives 3.
3. **The walk.** Magic picks byte order, timestamp precision and header size, then records are
   walked from offset 24 by `incl_len`, never by `orig_len` (which overshoots the file on a cut
   record). We hardened the generic reader to handle all six classic magics, and to stop at a
   trailing header fragment, a record whose payload runs past EOF, or an absurd `incl_len`:

   ```python
   ts, _frac, incl, orig = struct.unpack_from(e + "IIII", d, p)
   step = orig if by_orig else incl
   if incl > MAX_REC or (by_orig and orig > MAX_REC):
       stop = "bogus length at record %d" % (n + 1); break
   if p + rh + step > len(d):
       partial, stop = True, "record %d cut short" % (n + 1); break
   n += 1
   p += rh + step
   ```

4. **Submission.** Variant 0 was the complete-record count. For the Single Solve that gave `3`, with
   `2` (untruncated only) as the alternate group variant. That submission was never graded; see below.

## Time Trial and Mastery

The Time Trial and Mastery solves reused the recap-bundle resolver from earlier days
unchanged for 31 fields, so the work was only the pcap field.

- **Time Trial.** The regular attempt took two calls: the 31 base fields were accepted in one
  call, and the pcap count (variant 0) was CORRECT on the next. A later ranked attempt (50 s
  window) was CORRECT in one call, 4.78 s total locally.
- **Mastery.** The regular attempt grouped the pcap alternates and the 9 sets that offered them
  were accepted on variant 0, then the remaining 343 fields came back CORRECT. The ranked attempt
  (355 s window) was CORRECT in one call, 7.73 s locally. Across the 22 Mastery sets seen
  (regular plus ranked) the captures were little- and big-endian, microsecond and nanosecond,
  linktypes 1, 101 and 105, with 0 to 5 snaplen-cut records each, and the plain complete-record
  count was accepted in every graded run.
- Mastery also mixed in per-set manifests where the evidence-note member names differ by set,
  so resolving the file named by each question ID (not a hard-coded name) mattered; this was
  already handled by the existing bundle toolkit.

## Dead ends and gotchas

- **No malformed capture was ever delivered.** The "reject malformed captures" clause is
  untested: we never saw a trailing partial record, a bad magic or a bogus length in a graded
  fixture, so the right answer for such a file is unknown. The solver's ladder of alternates
  (count including the partial record, `orig_len` walk, untruncated count, `0`, `none`) is guesswork.
- **Cut records count.** The tempting "lost packets do not count" reading (untruncated only)
  would have been wrong: the validator took the plain record count every time.
- **The Single Solve was never graded for us.** Our submissions returned HTTP 200
  `already_solved`, so the value `3` is a candidate that nothing validated. Its correctness
  is inferred only from the same rule being accepted in Time Trial and Mastery. Retry and
  practice for this Single Solve were refused (405 and 409), so it could not be reissued.
- The first Mastery `start` returned the team's CORRECT readout with no `artifacts[]`; a fresh
  `retry` was needed to get a real bundle. Automation also mistook the `already_solved` replies
  for rejections and ran needless fix rounds; see the Day 27 overview.

## Takeaways

Technique to search for: **pcap record walk by `incl_len`**. Count every record whose full
`incl_len` payload is present, including snaplen-cut records, and stop at EOF. Check the magic
rather than trusting the `.pcap` extension, since byte order and timestamp precision varied across
fixtures. Do not walk by `orig_len`. When a challenge prompt lists several tasks ("count, reject,
find"), the questions actually asked may cover only one, so do not over-build for the rest until
the question list says so.

## Evidence

Leaf base path: `challenges/day27/metadata-forensics/metadata-forensics-day-27-packet-loss-prevention/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-adapter/solve.py` (hardened pcap walker, ungraded candidate)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`,
  `mastery/ranked-1/result.json`
- `challenges/day27/_run/librarian/metadata-forensics-day-27-packet-loss-prevention.md`,
  `challenges/day27/_run/lane-metadata-forensics-day-27-packet-loss-prevention.log`
- `library/metadata-forensics/pcap-packet-count/README.md` (META-PCAP-PACKET-COUNT-001)
- `progress.md` (grep `^| 27 .*packet-loss-prevention`)
