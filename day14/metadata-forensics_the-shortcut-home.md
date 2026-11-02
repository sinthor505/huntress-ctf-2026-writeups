# The Shortcut Home

| | |
|---|---|
| Day / Lane | 14 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-14-shortcut-home` |
| Technique | Windows Shell Link (`.lnk`) parsing: LinkInfo `LocalBasePath` cross-checked against the IDList |
| Single Solve | Independent (`submission-01.json` reads CORRECT) |
| Time Trial | Independent (`submission-01.json` CORRECT; later ranked reruns CORRECT) |
| Mastery | Independent (`submission-01.json` CORRECT; later ranked reruns CORRECT) |

## Summary

The challenge hands over a ZIP holding a Windows `.lnk` and asks for its target path. The only trap
is reading the right field: the shortcut also carries a description and a working directory, which
are not the target. We parsed the Shell Link structure, took the LinkInfo local base path, confirmed
it against the path rebuilt from the IDList, and submitted. All three modes went CORRECT on the first
submission.

## The challenge

Prompt: "A shortcut collection is a map of intent - if you read where each link points instead of
where it sits. Reconstruct the targets and identify the impossible destination." The wording
suggests a collection of links with one odd target, but every delivery asked for exactly one thing
per shortcut: `lnk.target_path_windows`, answer format `exact_text`, normalization `trim`.

- **Single Solve:** one question (`d14-lnk-001-...`), artifact `metadata-day-14-evidence.zip`
  containing one 547-byte member, `shortcut-001.lnk`. Untimed.
- **Time Trial:** same shape, `shortcut-002.lnk`, one question, server deadline 50 s after start.
- **Mastery:** an outer ZIP holding two nested archives (`metadata-day-14-evidence-01.zip`,
  `-02.zip`, about 475 bytes each), each with one shortcut (`shortcut-003.lnk`, `shortcut-004.lnk`).
  Two questions with `set1.`/`set2.` prefixes, 80 s window.

## Solution

1. **Recon.** Checked the download's SHA-256 against the start response, listed the archive (one
   member), and confirmed the member began with the Shell Link header signature. Static `strings`
   showed several Windows paths and a UTF-16 description, which were leads only: a shortcut stores a
   target in more than one place and the strings do not say which is which.
2. **Parse, do not grep.** The Shell Link layout (MS-SHLLINK) is a 76-byte header, an optional
   LinkTargetIDList, an optional LinkInfo structure, counted string data (name, relative path,
   working dir, arguments, icon) and extra-data blocks. The target path lives in two independent
   places: LinkInfo's `LocalBasePath` (a flat string) and the IDList (a shell-item chain that resolves
   drive, folders and file name). The first solve used `pylnk3` on the ZIP member bytes and read
   `link._link_info.local_base_path`:

   ```python
   with zipfile.ZipFile(archive) as zf:
       link_bytes = zf.read("shortcut-001.lnk")
   link = pylnk3.parse(io.BytesIO(link_bytes))
   print(json.dumps({"d14-lnk-001-lnk-target-path-windows":
                     link._link_info.local_base_path}))
   ```

3. **Cross-check.** The parser reported a local volume labelled `HUNTRESS` and the path
   `C:\Windows\System32\cmd.exe`. The IDList independently resolved `C:` -> `Windows` -> `System32` ->
   `cmd.exe`. Two structured representations agreeing was the confidence signal; the description and
   working directory were not used.
4. **Submit.** The answer was the path exactly as stored:
   `C:\Windows\System32\cmd.exe`. It was sent in one call and graded CORRECT.

Single Solve used `shortcut-001.lnk`; Time Trial and Mastery each used their own specimens but
the same method. Nothing was executed.

## Time Trial and Mastery

Nothing changed in kind. The Time Trial was a single fresh shortcut inside a 50 s window; the
Mastery added one level of nesting (outer ZIP -> two inner ZIPs -> one `.lnk` each) and two answers
submitted together in one call. Both shortcuts resolved to the same path as the Single Solve; LinkInfo
and IDList agreed in each. Both were CORRECT on submission 1 of the original attempts.

Later ranked reruns (scoreboard attempts, not part of the solve) used a different, larger fixture
than these first deliveries. The Time Trial carried 19 questions and the Mastery 95 (5 sets of 19),
with the Shell Link question as just one field in a general evidence bundle (the question text
became "Inspect `shortcut-001.lnk`. Determine ..." against `evidence.zip`). The target path was
`C:\Windows\System32\cmd.exe` in all five Mastery sets of the saved ranked plan. The per-challenge solver
for those runs wraps the library LNK reader and an independent LinkInfo reader as a cross-check. Both ranked
passes (two each for Time Trial and Mastery) were CORRECT in one call.

## Dead ends and gotchas

- **The prompt promises more than the questions ask.** "Identify the impossible destination"
  reads like a hunt for an anomalous target, but no question asked for one and every specimen we
  parsed pointed at the same place. We did not find, and could not determine from the files, what
  the "impossible destination" phrase referred to.
- **Description and working directory are not the target.** They are separate string-data fields
  and easy to confuse when skimming `strings` output.
- **Two target representations.** Use LinkInfo `LocalBasePath` as the answer and the IDList as a
  check. If a shortcut has no LinkInfo (`ForceNoLinkInfo`), fall back to the IDList path and then the
  environment-variable block. The specimens here all had LinkInfo with an empty common path suffix,
  so the join rule (base + `\` + suffix) was never exercised against the validator.
- **Private parser field.** `pylnk3` exposes the LinkInfo path only through `_link_info`, so the
  solve depended on a private attribute. A stdlib reader later replaced it in the library.

## Takeaways

Technique to search for: **Shell Link LinkInfo/IDList target extraction** (`.lnk`, MS-SHLLINK).
Parse the structure rather than trusting strings, take `LocalBasePath` for the target, and
confirm with the IDList. The same rule held for later days' shortcuts, including per-set different
targets on Day 23, according to the library entry. With a stdlib reader the whole job is a single
call on the archive and takes seconds; the main cost the first time was finding a parser, since no
Shell Link note existed at that point.

## Evidence

Base path: `challenges/day14/metadata-forensics/metadata-forensics-day-14-shortcut-home/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/submission-01.json`,
  `single-solve/artifacts/answer_map.py`
- `time-trial/journal.md`, `time-trial/submission-01.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/submission-01.json`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `tools/ranked/solvers/metadata-forensics-day-14-shortcut-home/solve.py`
- `library/metadata-forensics/lnk-target-path.md` (META-LNK-TARGET-PATH-B103)
- `progress.md` (grep `^| 14 .*shortcut-home`)
