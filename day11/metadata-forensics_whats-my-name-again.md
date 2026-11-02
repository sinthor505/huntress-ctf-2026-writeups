# What's My Name Again?

| | |
|---|---|
| Day / Lane | 11 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-11-whats-my-name-again` |
| Technique | PE `RT_VERSION` resource parsing (`OriginalFilename` in `StringFileInfo`) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (ranked attempt `result.json`; the original attempt is journal-recorded) |
| Mastery | Independent (ranked attempt `result.json`; the original attempt is journal-recorded) |

## Summary

Each delivery was a Windows executable with a randomized name, and the question asked for the
`OriginalFilename` string in its version resource. The value sits in the PE's `RT_VERSION` resource
as UTF-16LE text. Single Solve was done by hand with `strings`, `objdump` and `xxd`. After that a
resource-tree walker read the field directly, which was fast enough for the 50 s and 80 s Time Trial
and Mastery windows. Every mode went CORRECT on its first submission.

## The challenge

The prompt: "Filenames have been randomized, but Windows version resources remember what the binaries
called themselves. Recover the original identities and expose the impersonator." The prompt reads as
if it wants a comparison across several binaries. The actual question was narrower and the same in
every mode: `Determine pe.version_original_filename for telemetry-agent-NNN.exe.` Each question had
one answer, `answer_format: exact_text` with `trim` normalization. The answer ID keeps a `d09-pe-NNN-`
prefix even though this is Day 11 (the prefix comes from the challenge generator).

| Mode | Delivery | Answer fields | Window |
|---|---|---|---|
| Single Solve | ZIP with one PE32 (`telemetry-agent-001.exe`, 1.26 MB) | 1 | none |
| Time Trial | ZIP with one PE32+ (`telemetry-agent-005.exe`) | 1 | ~50 s |
| Mastery | outer ZIP of two inner evidence ZIPs, one PE each (`-006`, `-007`) | 2 (`set1.` / `set2.` prefixed) | ~80 s |

## Solution

1. **Verify the bytes.** The Single Solve archive's SHA-256 matched the start response. It held one
   member, and `file` called it a PE32 console executable for Intel 80386 with seven sections.
2. **Confirm there is a resource section.** `objdump -x` showed a populated Resource Directory
   (`.rsrc`) and an empty CLR header, so the sample is native, not .NET. Version info would be in
   `RT_VERSION` if anywhere.
3. **Find the field.** A UTF-16LE string search located the version-resource structure:

   ```text
   strings -el -t x telemetry-agent-001.exe
   132e5e VS_VERSION_INFO
   132eba StringFileInfo
   132ede 040904b0
   132f62 OriginalFilename
   132f84 pe-base-386.exe
   132faa ProductName
   ```

   `040904b0` is the language/codepage table key (US English, Unicode). `xxd` at `0x132f60` showed
   the key, its terminator, and then the value immediately after.
4. **Submit** the value verbatim: `pe-base-386.exe`, under `d09-pe-001-pe-version-original-filename`.

The executable was never run. A `strings` hit alone only proves the text exists somewhere in the
file. It does not prove it is a `VS_VERSION_INFO` key/value pair. Here the surrounding
`VS_VERSION_INFO` / `StringFileInfo` / codepage markers and the position inside the `.rsrc` region
supplied that confirmation. The library entry we wrote afterwards notes the same limit.

## Time Trial and Mastery

The question family did not change. The scale and the clock did.

- **Parser instead of `strings`.** The manual workflow does not fit inside 50 s. We had since added an
  RT_VERSION walker (in the metadata library) that reads every `StringFileInfo` table, and the Time
  Trial was a single call: extract the member, print `OriginalFilename`, submit. (The journal
  records a stdlib walker in the library's `metaread.py` that matched `pefile` on three PE files; the
  Time Trial itself used the library's PE header-facts tool, which has its own RT_VERSION walker.)
- **Architecture varies per specimen.** The specimen names track the architecture label, and each
  mode drew a different one: `pe-base-386.exe` (Single Solve), `pe-base-amd64.exe` (Time Trial,
  PE32+), `pe-base-arm64.exe` and `pe-base-386.exe` (the two Mastery sets).
- **Mastery nesting.** The Mastery download was an outer ZIP containing `metadata-day-11-evidence-01.zip`
  and `-02.zip`, one executable in each. We checked both members for the `MZ` magic before parsing, then
  submitted `set1.` and `set2.` fields in one call.
- **Attempts.** The original Time Trial and Mastery attempts were CORRECT on attempt 1 (journal-recorded).
  A later scripted ranked pass for each, on legacy routes where a plain retry is the ranked attempt,
  also went CORRECT on one call: 2.12 s total locally for Time Trial and 2.67 s for Mastery. Two further
  Time Trial retries returned `practice_only` with `unique_attempts_remaining: 0`, so that pool was
  spent. The scripted solver is the same engine as Day 10's, since the field is identical.

## Dead ends and gotchas

- **The prompt promises an "impersonator".** No question asked about one. Each field was a single
  `OriginalFilename`, and nothing in any mode required cross-comparing binaries, so there was nothing
  to expose.
- **Answers do not carry across modes.** The filename string depends on the specimen's architecture,
  so a value from one mode is not a safe guess for another.
- **Multiple string tables.** A PE can carry several language/codepage tables. All the specimens here
  used `040904b0`. If one had several, the question gives no way to pick between them, so we would
  have had to check the question's scope and could not have resolved it from the prompt alone.
- **Ranked-board attribution (outside the challenge).** A later check found the ranked Mastery result
  attributed to a solo account rather than the team. A fresh ranked retry to fix it returned 409
  `rotation_exhausted`, so it cannot be redone.

## Takeaways

The technique is **PE version-resource extraction**. Check that `.rsrc` exists, then walk
`RT_VERSION` -> `VS_VERSION_INFO` -> `StringFileInfo` -> language/codepage table and read the
requested key. Use `strings -el` as a lead and confirm it against the resource structure. For a
timed run, use a parser (`pefile`, or a small resource walker) rather than a manual search. In the
Mastery shape, resolve members from each set's own archive and check magic bytes rather than
trusting file names, because the names were randomized by design.

## Evidence

Leaf base: `challenges/day11/metadata-forensics/metadata-forensics-day-11-whats-my-name-again/`

- `single-solve/journal.md` (recon, hex evidence, CORRECT on attempt 1; no `result.json` or start response was saved for this mode)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/ranked-1/result.json` (`ranked-2/`, `ranked-3/` are `practice_only`)
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/ranked-1/result.json`
- Solver for the ranked passes: `tools/ranked/solvers/metadata-forensics-day-11-whats-my-name-again/solve.py`
- Library entry META-PE-VERSION-INFO-001 (`library/metadata-forensics/pe-version-info.md`), and the RT_VERSION reader in `library/metadata-forensics/evidence-bundle-toolkit/metaread.py`
- `progress.md` (grep `^| 11 .*whats-my-name-again`)
