# Portable Deception

| | |
|---|---|
| Day / Lane | 6 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-06-portable-deception` |
| Technique | PDF header version (`%PDF-x.y`) read from raw member bytes, including nested ZIPs |
| Single Solve | Solved (journal-recorded): the journal records CORRECT on submission 1; no response or result file was saved |
| Time Trial | Independent (journal records CORRECT on submission 1; a 2026-10-26 re-run holds `ranked-1/result.json` CORRECT) |
| Mastery | Independent (journal records CORRECT on submission 1; a 2026-10-26 re-run holds `ranked-1/result.json` CORRECT) |

## Summary

The prompt promised a hunt for "renamed impostors" and "which promise is false", but every mode asked one literal
field: `pdf.header_version` of a single named PDF. The answer is the `x.y` after `%PDF-` in the member's raw bytes.
All three modes were solved on the first submission (per the journals; only Time Trial and Mastery have a preserved graded result file, from a later re-run).

## The challenge

Prompt: "A document bundle mixes real PDFs, renamed impostors, and multiple declared versions. Determine what the
files promise, what they contain, and which promise is false."

What was delivered:

- **Single Solve:** `metadata-day-06-evidence.zip` (one member, `document-001.pdf`, 1,552 bytes). One question,
  `d06-pdf-001-pdf-header-version`, format `exact_text`, normalization `trim`. No deadline.
- **Time Trial:** the same archive shape with `document-002.pdf`, one question, 50 second server-enforced window.
- **Mastery:** an outer ZIP holding two nested archives (`metadata-day-06-evidence-01.zip`,
  `-02.zip`), each with one PDF (`document-003.pdf`, `document-004.pdf`). Two questions with `set1.` / `set2.`
  prefixed IDs, 80 second window.

The answer format said to give the exact artifact-derived value, and to use the literal `none` only if the field
is absent.

## Solution

1. Checked the archive SHA-256 against the start response, then listed the members. Each ZIP held one 1,552-byte
   PDF; for Mastery the outer ZIP held two nested ZIPs, which were opened separately.
2. Confirmed the member really was a PDF and not an impostor: `file` identified a one-page PDF, the first bytes
   were `25 50 44 46 2d 31 2e 34 0a` (`%PDF-1.4\n`), `strings` showed ordinary objects and an Info dictionary, and the
   file ended with `xref`, `trailer`, `startxref`, `%%EOF`. There was no second `%PDF-` marker later in the file.
3. Read the version from the header. Readers accept the marker anywhere in the first 1024 bytes, so the extractor
   searches that bounded prefix rather than assuming offset 0:

```python
HEADER = re.compile(rb"%PDF-(\d\.\d)")

def describe(name, data):
    version = HEADER.search(data[:1024])
    print(f"{name}: pdf.header_version={version.group(1).decode() if version else 'none'}")
```

4. Submitted the bare `x.y` string (no `PDF-` prefix, no quotes).

Answers: Single Solve `1.4`; Time Trial `1.5`; Mastery set 1 `1.6`, set 2 `1.7`.
Each mode's fixture used a different version, so a value from one mode did not carry over to the next.

## Time Trial and Mastery

Scale was tiny: one field in Time Trial, two in Mastery (one per set). The only real change was packaging.
The 50 and 80 second windows were short, so the extractor was written and tested on the Single Solve bytes before
the clock started. For Mastery, the same function ran once per nested ZIP, and the parser hashed each member for
the journal. Each solve took a single submission well inside its window (the journals record solves roughly 4 s
after the Time Trial start and 12 s after the Mastery start).

Later, in a re-run of these modes (2026-10-26, rerun as a scripted pass that still graded CORRECT on one call each),
the live fixtures were larger than the originals. They carried 10 fields (Time Trial) and 20 fields (Mastery, 2
sets x 10): the cumulative evidence-bundle template, which also asks the earlier days' note-hash, JPEG and PNG
fields next to `pdf.header_version`. The PDF field was unchanged (`document-001.pdf`, answer `1.4`). Fixtures
differ between attempts of the same mode, so the solver reads the member name and answer ID from each start
response instead of hard-coding them.

## Dead ends and gotchas

- **Flavor text did not change the question.** "Renamed impostors" and "which promise is false" suggested comparing
  extension, magic bytes and declared versions. Here every PDF was genuine, the header sat at offset 0, and the
  asked field was only the header version. The deep review of the saved fixtures found no catalog `/Version`
  override and no header at a nonzero offset, so those cross-checks (catalog `/Version`, incremental updates,
  trailing bytes) never mattered, and no bytes were found that would make them matter.
- **Header version is not catalog version.** A PDF catalog may declare a later effective `/Version`; the question
  named the header (`pdf.header_version`), so the header is what was submitted. No fixture here disagreed.
- **Answer spelling.** The accepted form was `x.y` only, not `PDF-1.4` or `%PDF-1.4`.

## Takeaways

- For a "tell me what the file claims" question, answer the literal field named in the question ID. Treat the
  prompt's storytelling as a reason to cross-check, not as the question.
- Technique to search for: PDF header version, first-1024-byte `%PDF-` marker search, nested-archive member
  extraction. When the file is in an archive, read the member bytes, not the container's metadata.
- What we would do faster: nothing structural. The stdlib regex above was enough; the time went into the
  one-submission discipline and verifying hashes.

## Evidence

Leaf base: `challenges/day06/metadata-forensics/metadata-forensics-day-06-portable-deception/`

- `single-solve/journal.md`, `time-trial/journal.md`, `mastery/journal.md` (each records the start response summary,
  hashes and a `CORRECT` first submission)
- `time-trial/parse_pdf_headers.py`, `mastery/parse_pdf_headers.py` (the extractor used)
- `time-trial/ranked-1/result.json`, `mastery/ranked-1/result.json` (later scripted re-runs, `CORRECT`),
  with `accepted_plan.json` and `pass.log` beside each
- Later solver: `tools/ranked/solvers/metadata-forensics-day-06-portable-deception/solve.py`
- Local `progress.md` rows for Day 6 mark all three modes `Solved`
