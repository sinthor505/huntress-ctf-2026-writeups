# Hive Mind

| | |
|---|---|
| Day / Lane | 25 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-25-hive-mind` |
| Technique | Windows registry hive (`regf`) parsing: true root key name vs. decoy embedded file name |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt talked about regrouping hives from several systems and isolating a planted one, but the
only Day 25 question was `registry.hive_root_name`: the name of a hive's root key. The trap is the
`regf` base block, which carries an embedded file name that never matches the real root. The true
root is the `nk` cell that the header's root-cell offset points at. Single Solve, Time Trial and
Mastery each went CORRECT on the first submission.

## The challenge

Prompt: "Registry hives from multiple systems have been renamed and separated from host labels.
Identify each true root, regroup the systems, and isolate the planted hive."

- **Single Solve:** one evidence ZIP (`metadata-day-25-evidence.zip`) holding `hive-001.hiv`, one
  answer: `d25-registry-001-registry-hive-root-name`, `exact_text`, `trim`.
- **Time Trial:** `evidence.zip`, 30 answers (50 s ranked window).
- **Mastery:** nine evidence ZIPs, 270 answers (9 sets x 30; 295 s ranked window).

No question ever asked about grouping systems or finding a planted hive. The only hive-specific
field was the root name; the synthesis question's `Files:` list only used the hive as one of five
files to order by SHA-256.

## Solution

1. **Parse the header.** A `regf` file has a 4 KiB base block, then `hbin` pages. The u32 at
   offset 36 is the root cell's offset relative to `0x1000`.
2. **Read the root `nk`.** The cell is an `int32` size followed by `nk`. Flags are a u16 at
   `nk+2`, the name length a u16 at `nk+0x48`, and the name starts at `nk+0x4C`. Flag bit `0x20`
   means the name is stored as ASCII (latin-1), otherwise UTF-16-LE. The solver's reader:

   ```python
   root_off = struct.unpack_from("<I", d, 36)[0]
   pos = 0x1000 + root_off                    # cell: int32 size, then 'nk'
   nk = pos + 4
   flags = struct.unpack_from("<H", d, nk + 2)[0]
   nlen  = struct.unpack_from("<H", d, nk + 0x48)[0]
   raw   = d[nk + 0x4C : nk + 0x4C + nlen]
   name  = raw.decode("latin-1") if flags & 0x20 else raw.decode("utf-16-le")
   ```
3. **Ignore the decoy.** The base block also stores a 64-byte UTF-16 file name at offset 48. For
   the Single Solve hive that read `ress-artifact-assembly-line.hiv` (truncated), while the real
   root key was `HuntressArtifactAssemblyLine`. We submitted the `nk` name exactly as stored,
   case preserved, and it was accepted on call 1. The journal's fallback variants (the embedded
   name, its stem, an upper-cased form, `none`) were never needed.

The hives were tiny: all 21 seen across the five graded attempts were 8,192 bytes (base block plus
one `hbin`), root flags `0x002c`, no subkeys. So there was no key tree to walk. In all 21 the root
name differed from the embedded name, even case-insensitively.

## Time Trial and Mastery

Time Trial and Mastery added no new hive logic. They asked the same one hive field among 30 per
set: seven file-level fields (roster/evidence-note names, sizes, MD5/SHA-1/SHA-256, magic prefix,
BOM flag), one to three fields for each of the Day 4-24 format families (JPEG, PNG, PDF, ZIP, PE, .NET, MSI,
LNK, OOXML, EML, WAV, MP4, ELF, Mach-O, APK, JAR, SQLite, EVTX), the Day 25 hive field, and a
synthesis question asking for five named files in ascending SHA-256 order. Mastery repeated that
block for nine sets, with a different evidence-note and roster name per set.

What mattered was breadth: an existing field-reader library covered every field, so the solve was
one run over the bundle and one submit. The library's `registry.hive_root_name` reader had been
added during the Single Solve window (it had returned only header fields before), and all three
modes passed the root-name field on the first call: 30/30 for Time Trial, 270/270 for Mastery.
Regular (untimed) and ranked attempts both returned CORRECT on one call each; ranked reruns later
(a second and third ranked attempt on each of Time Trial and Mastery) were also CORRECT on one
call. Our own local solve time for the first ranked attempts was 3.48 s (Time Trial) and 3.72 s
(Mastery).

## Dead ends and gotchas

- **The embedded file name is a decoy.** Answering with it, its stem, an upper-cased form, or the
  ZIP member's own file name is wrong. The only trustworthy source is the `nk` the header's
  root-cell offset points at.
- **The prompt overstates the task.** "Regroup the systems" and "isolate the planted hive" never
  appeared as questions; do not build grouping logic for this challenge.
- A single-field solver written for the Single Solve failed on every Time Trial and Mastery set
  (`unsupported field` for the other 29 fields); only the general field-reader covered them. That
  was an automation gap rather than a challenge trap (see the day overview).

## Takeaways

- **Registry hive structure:** answer from the structure the header points at (`regf` root-cell
  offset at 36, then `nk`), never from a label stored in the base block. A name that sits outside the
  hive tree is trivially forgeable and is placed there as a decoy.
- Check the flag bit `0x20` before decoding a key name; ASCII and UTF-16 names both occur in the
  format. We only saw ASCII names here, so UTF-16 root names are unconfirmed.
- On a metadata day, a Single Solve often asks one new field while Time Trial and Mastery bundle
  that field with the previous days' families. Put every new reader into the general field library
  the day it appears, not into a one-off solver.

## Evidence

Leaf: `challenges/day25/metadata-forensics/metadata-forensics-day-25-hive-mind/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-rubric/solve.py` (stdlib root-`nk` reader)
- `time-trial/{regular,ranked-1,ranked-2,ranked-3}/result.json`, `time-trial/ranked-1/start_response.json`
- `mastery/{regular,ranked-1,ranked-2,ranked-3}/result.json`, `mastery/ranked-1/accepted_plan.json`

Day level, under `challenges/day25/_run/`: `librarian/metadata-forensics-day-25-hive-mind.md`,
`lane-metadata-forensics-day-25-hive-mind.log`. Library entry META-REGF-ROOT-NAME-001
(`library/metadata-forensics/registry-hive-root-name/`). `progress.md` (grep `^| 25 .*hive-mind`).
