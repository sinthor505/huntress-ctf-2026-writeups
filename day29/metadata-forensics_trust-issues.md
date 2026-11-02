# Trust Issues

| | |
|---|---|
| Day / Lane | 29 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-29-trust-issues` |
| Technique | X.509 fingerprinting (SHA-256 of the DER), GPT partition-table parsing, cumulative evidence-bundle field families |
| Single Solve | Independent (1 submission call, CORRECT) |
| Time Trial | Independent (regular rehearsal CORRECT in 2 calls; ranked attempt CORRECT in 1 call, 3.73 s local, 50 s window) |
| Mastery | Independent (regular rehearsal CORRECT in 1 call; ranked attempt CORRECT in 1 call, 7.23 s local, 415 s window) |

## Summary

The Single Solve asked for one value: the SHA-256 fingerprint of a PEM certificate inside a 709-byte ZIP. The Time Trial and Mastery turned out to be cumulative recap bundles that also re-asked the previous day's new field (GPT partition start LBAs), and our first Time Trial attempt failed because the toolkit had no disk reader. After adding one, every mode went CORRECT.

## The challenge

The prompt, shared by all three modes, was:

> Certificates appear in binaries, packages, mail, captures, and disk images. Fingerprint each one, collapse duplicates, and identify the trust claim unsupported by evidence.

That reads as if the challenge involves dedup counts and an "unsupported claim" judgement, but none of it appeared in the questions we were asked.

- **Single Solve:** 1 question, `x509.sha256_fingerprint` for `certificate-001.pem` (exact 64-hex SHA-256, case-insensitive), delivered as `metadata-day-29-evidence.zip`.
- **Time Trial:** 34 questions on one `evidence.zip`. These are the whole accumulated family list: the generic note/roster questions (filename, size, MD5, SHA-1/256, magic prefix, BOM), then one question per earlier day's file type (JPEG, PNG, PDF, ZIP, PE, .NET, MSI, LNK, OOXML, email, audio, video, ELF, Mach-O, APK, JAR, SQLite, EVTX, registry, prefetch, pcap), then `d28-disk-001-...partition-start-lba-sequence`, the Day 29 X.509 fingerprint, and a final synthesis (`metadata-day-29-synthesis-sha256-order`).
- **Mastery:** 13 sets of those same 34 questions = 442 answers, delivered as a ZIP of per-set `evidence-NN.zip` bundles.

## Solution

1. **Identify the field.** The only Single Solve question names `certificate-001.pem` and asks for the SHA-256 fingerprint. The standard certificate fingerprint is the hash of the DER encoding, not of the PEM text. So we base64-decoded the body between the `BEGIN CERTIFICATE` / `END CERTIFICATE` lines and hashed the resulting bytes.
2. **Compute it natively.** The rubric candidate does this in a few lines and does not depend on any library reader:

   ```python
   PEM_RE = re.compile(rb"-----BEGIN ([A-Z0-9 ]+)-----(.*?)-----END \1-----", re.S)
   def pem_blocks(raw):
       return [(m.group(1).decode(), base64.b64decode(re.sub(rb"\s+", b"", m.group(2))))
               for m in PEM_RE.finditer(raw)]
   ...
   answers[qid] = hashlib.sha256(blocks[0]).hexdigest()
   ```

3. **Keep alternates, submit the primary.** The solver registered one alternate reading (SHA-256 of the PEM file bytes as stored) in case the validator wanted that. It was never needed: the DER hash went CORRECT on the first call.

Submitted answer: `b466fdbf8b8b8cb2d0b632bd24eb875576b44edb85f18f90f53dcdea1a0ec781`.

## Time Trial and Mastery

The Single Solve map covered only the Day 29 field. The bundles needed everything:

- **Older families** (Days 1-27) were already handled by the evidence-bundle toolkit (`metaread` / `question_facts`) built over the previous weeks.
- **X.509 fingerprint** reused the Single Solve logic. While solving the Single Solve, one worker also found that the toolkit's generic fallback had tied `sha1_fingerprint` with `sha256_fingerprint` for a field named "fingerprint" and put a 40-hex SHA-1 first. A digest-name guard (a field naming `sha256` skips facts naming another digest) fixed that.
- **The gap: `disk.partition_start_lba_sequence`.** This was Day 28's new field and the toolkit had no disk reader. The first regular Time Trial rehearsal resolved 33 of 34 answers and the solver exited non-zero, so nothing was submitted. About three minutes later two reworked candidates answered 34/34 by reusing the Day 28 GPT reader:
  - Read sector 1; if it begins `EFI PART`, take the entry-array LBA, entry count and entry size from the header (offsets 72-88).
  - Walk the entries in table order, skip all-zero entries, and take the first-LBA field (bytes 32-40) of each.
  - If the primary header is absent, try the backup header at the last sector, then fall back to an MBR read.
  - The answer is a compact JSON array, e.g. `[2048,...]`, in table order. A sorted-order variant was registered as an alternate when there was more than one partition.
- **Ranked runs.** With 34/34 resolved, the ranked Time Trial and the ranked Mastery each went CORRECT on the first submission (`full-prefer`, 34 and 442 answers). In the Time Trial rehearsal the base answers were accepted in one call and the fingerprint group in a second. We did not need to determine which alternates the validator would have wanted for the remaining fields.
- **Mastery sets.** All 13 sets answered 34/34 from the adapter candidate. The rubric candidate crashed on one set's ZIP (`BadZipFile` in the lane log; we did not investigate the cause) and the adapter candidate covered that set.

## Dead ends and gotchas

- **The prompt promises more than it asks.** "Collapse duplicates" and "unsupported trust claim" never turned up as fields. The worker correctly refused to guess at them.
- **Generic fallback tie.** A fuzzy field-name match on "fingerprint" put the SHA-1 first; a 40-character answer to a 64-character format should have been caught by the lint, and was.
- **A day's Single Solve can be a small slice of its TT/Mastery.** Day 29's Single Solve was one new field; the bundles re-asked Day 28's new field, which the Single Solve could never have exposed.
- **Validator coverage is thin for the disk reader.** Every graded disk image was a primary-header GPT with an ascending table. The backup-header, MBR/EBR and sorted-order paths were only tested on synthetic images.

## Takeaways

- Hash the **DER** for X.509 fingerprints (`openssl x509 -fingerprint -sha256` does the same), not the PEM text.
- When a Single Solve is tiny, assume Time Trial and Mastery re-ask the previous day's new field as well as the earlier families. Make sure last day's reader is in the shared toolkit before the next release, not written under pressure.
- After the fix, the reader and X.509 rule were promoted into the toolkit (`metaread.read_disk`, kind `disk`, plus explicit `x509.sha256_fingerprint` / `sha1_fingerprint` field rules). Running the toolkit with no wrapper reproduced the accepted answers for the Single Solve (1/1), the Time Trial (34/34) and the 13 Mastery sets (442/442), and a regression sweep of Days 28-29 compared 1,811 fields with 0 mismatches.

## Evidence

- `challenges/day29/metadata-forensics/metadata-forensics-day-29-trust-issues/single-solve/journal.md` and `accepted_plan.json`
- `.../single-solve/artifacts/cand-rubric/solve.py` and `cand-adapter/solve.py` (Single Solve candidates, reused for TT/Mastery)
- `.../time-trial/ranked-1/result.json`, `.../time-trial/regular/result.json`, `.../mastery/ranked-1/result.json`, `.../mastery/regular/result.json`
- `challenges/day29/_run/lane-metadata-forensics-day-29-trust-issues.log` and `_run/librarian/metadata-forensics-day-29-trust-issues.md`
