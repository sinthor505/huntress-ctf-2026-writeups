# Hatch Careful

| | |
|---|---|
| Day / Lane | 5 / Script Analysis |
| Challenge ID | `script-analysis-day-05-python-carrier-lab` |
| Technique | Static decoding of descriptor-in-code specimens; multi-answer validator bisection |
| Single Solve | Independent (graded CORRECT saved in `single-solve/probe-final-1-response.json`) |
| Time Trial | Independent (CORRECT on attempt 2; attempt 1 expired) |
| Mastery | Independent (CORRECT on attempt 1) |

## Summary

A 45-member encrypted ZIP of "specimen" scripts, each hiding a small behavior descriptor behind a
byte transform, with 357 questions about roles, MITRE ATT&CK IDs, indicators and structural labels.
The decoding was mechanical. The cost was in the validator: 14 full-map submissions came back
`incorrect` because two fields out of 357 were wrong, and only splitting the map into per-profile
chunks (which banks the correct ones) found them. Time Trial and Mastery reused the same decoder
on fresh archives.

## The challenge

The prompt: "Nine Python behaviors come wrapped in fourteen disguises: bytecode, wheels, zipapps,
frozen executables, and armored runtimes. Peel each carrier back to its intent without letting
anything hatch, and identify the package that tells a different story than its payload."

Delivered: one AES-encrypted `evidence.zip` (password `infected`) with 45 members: three
PowerShell samples (resource hijacking, collection, impact), nine each of JScript, Batch and
VBScript (the same nine behavior profiles per language: benign, stager, dropper, downloader,
launcher, chain, uploader, impact, relay), and 14 Python files plus one inert PowerShell
"handoff plan". Single Solve had 357 answers: 15 PowerShell, 72 each for JScript/Batch/VBScript, and
126 Python (nine fields per carrier). Answer kinds were enums (`role`, `control`, styles), exact
text (URLs, paths, campaign IDs, wallet, mutex) and MITRE IDs. Single Solve was untimed.

## Solution

1. **Open the archive.** `pyzipper` with `infected`; members were read as text only and never
   run. (The first local copy of the ZIP had been corrupted by line-ending translation and
   failed with "Bad magic number"; re-downloading and checking the SHA-256 from the start
   response fixed that. It was not the cause of the rejections.)
2. **The carriers are labels, not containers.** The 14 Python "carrier" files (`pyc38`,
   `current-pyc`, `zipapp`, `wheel`, `pyinstaller-onefile/onedir`, `py2exe-console/service`,
   `cxfreeze`, `cxfreeze-msi`, `pyarmor-source`, `pyarmor-pyinstaller`, `embedded-runtime`,
   `source-context`) are all already-recovered `.py` text. The ZIP holds no `.pyc`, `.whl`,
   `.pyz`, `.exe` or `.msi`, so nothing needs unpacking; the carrier appears only as a string.
   The "package that tells a different story" is most plausibly the `wheel` specimen (carrier
   label `wheel_archive`, payload `acquire_remote,dispatch_memory`), but with no `.dist-info`,
   `METADATA` or `WHEEL` member there was no real package metadata to contradict it. We answered
   from the payload (`stager`, `T1105`) and the literal carrier label, and the map was accepted.
3. **Decode each descriptor.** Every specimen stores a pipe-delimited descriptor such as
   `v=2|steps=decode_configuration,return_configuration|adapter=argparse_namespace|...|carrier=raw_source|...`
   behind a transform (decimal list reversed then XOR `0x0b` for `source-context.py`; escaped hex,
   URL-safe base64, rolling or repeating XOR, modulo add/subtract elsewhere). JScript/VBScript/Python
   constants were decoded statically; Batch descriptors are joined from `--inspect` fragments and
   cross-checked against `rem HUNTRESS-CODEC/MATERIAL` comments (they matched); PowerShell samples
   carry base64 JSON.
4. **Derive answers.** `role` and `attack` come from the decoded `steps` (terminal intent: a graph
   ending `acquire_remote,dispatch_memory` is a stager, `T1105`; `locate_local,dispatch_process` is a
   launcher; JScript launcher/chain/relay took `T1059.007`, Batch `T1059.003`, VBScript `T1059.005`,
   Python `T1059.006`). `adapter`, `campaign`, URLs, paths, `syntax`, `obfuscation`, `carrier` and
   `windows_chain` are the descriptor's literal values. Structural enums (Batch expansion and
   control, VBScript control, JScript function/invocation/property style) were read from the
   specimen's own code.
5. **Submit and bisect.** See the next sections. Final fix: `batch-downloader-control` and
   `batch-chain-control` were both `call_subroutine`, not `for_once`. The submission that returned
   CORRECT was the chunked map with those two values corrected.

## Time Trial and Mastery

- **Time Trial:** same 357-field schema, fresh archive and fresh random values, and a roughly 50 s
  server-enforced window even though the call was made without asking for a timer. The first
  attempt expired while the mapper was still being built, and a submission to it came back
  `expired` (a hard result, no grading). A `retry` issued a new archive and clock; decode, map,
  key-set check and submit were scripted as one chain and finished in about 6 s, CORRECT.
  The mapper (a per-family table keyed on member name) asserts that its keys equal the live
  `answer_ids`; that caught one schema difference immediately: **`jscript-impact` has no
  `-campaign` field** (8 fields instead of 9). The PowerShell samples' field letters were matched
  by value shape (`http...` URL, `wallet_...`, `Global\...` mutex, `.ext`, `*.txt` note,
  `Mozilla...` user-agent) rather than by key letter.
- **Mastery:** 714 answers, two independent full sets, IDs prefixed `set1.` / `set2.`, window about
  80 s. The two sets arrived as two separate top-level `evidence.zip` deliveries, each with its own
  capability URL, not as one outer ZIP of nested archives; fetching only the first delivery would
  have missed set 2. The Time Trial mapper was reused unchanged per set; one 714-field submission,
  CORRECT on the first call (about 60 s from start).
- Because Time Trial and Mastery windows were so short, these were solved by script, not by hand.
  Later ranked re-runs of both modes (2026-10-26) each went CORRECT in one call, with total pass
  times of 1.5 s and 2.6 s, using a general solver that rebuilds the answer map from the delivered
  bytes with no stored values.

## Dead ends and gotchas

- **14 full-map `incorrect` verdicts.** The service returns only a batch-level `incorrect`, so two
  wrong Batch controls among 357 fields were invisible. Half-map submissions (attempts 6 and 9)
  did not help either, because they carried stale values for other suspect fields.
- **Bisection via `accepted`.** A subset made only of correct answers returns `accepted` with
  `accepted_ids` and `remaining`, banking those fields; any wrong answer in a subset gives plain
  `incorrect`. Submitting PowerShell (15), JScript (72), VBScript (72) and Python (126) as chunks all
  returned `accepted`; Batch per profile accepted everything except downloader and chain; removing
  their `*-control` fields made both accepted, which isolated the culprits.
- **`for /l ... do call :LABEL` is `call_subroutine`.** Both bad specimens reconstruct their
  descriptor with `for /l %%Q in (1,1,1) do call :HUNTRESS_RECOVER`. The prompt defines `for_once`
  as a one-item FOR loop, and that was our reading, but the validator wanted `call_subroutine`.
  Batch controls are not stable across fixtures: Single Solve took `call_subroutine` for downloader
  and chain, while the accepted Time Trial map had downloader `direct` and chain `errorlevel_guard`.
  Classify each fresh specimen structurally. (The Time Trial journal's prose table of controls does
  not fully match its accepted map; we relied on the accepted map.)
- **Literal field beats inferred value.** A JScript uploader URL guessed as `.../upload` was wrong;
  the decoded `values.url` is the answer. The four Python `file_path` values keep their forward
  slashes (`C:/ProgramData/HuntressLab/.update-....py`) and are not Windows-normalized.
- **Adapters: descriptor field, not plan text.** For `python-source-benign-adapter` the
  operation-plan tuple says `report` while the descriptor says `argparse_namespace`; for
  `batch-benign-adapter` the `--plan` output says `set` while the descriptor says `for`. The
  descriptor literal (`argparse_namespace`, `for`) was accepted. For the PyArmor+PyInstaller
  specimen the Python file's `proxy_client_handoff` was accepted over the neighbouring PowerShell
  plan's `Start-Process`. In the Time Trial fixture the same fields took other literal values
  (`call`, `cpython_loader_handoff`), again the descriptor's own.
- **ATT&CK precision.** `resource-attack` was `T1496`. We tried `T1496.001` once (attempt 12), but
  that map still held the two wrong controls, so it said nothing; the later PowerShell chunk with
  `T1496` was accepted. The collection sample took `T1555.003` per the question's
  stated precedence for browser credential-store access.
- **Expired Time Trial attempts are not graded** (hard `expired`), unlike some other
  challenges. Have the mapper built and tested before starting the clock.
- One note on our own process: a journal claim that an earlier day had accepted `set` for the
  Batch benign adapter could not be verified anywhere and should not have been relied on (see the
  day overview for process notes).

## Takeaways

- For "carrier"/"disguise" challenges, check what is actually in the archive before reaching for
  unpackers; here the disguise was a declared string and the archive held only text.
- When a multi-answer validator gives only batch-level feedback, bisect with chunks that bank
  `accepted_ids` instead of resubmitting the full map. This would have saved 14 attempts.
- Take descriptor fields literally (adapter, URL, path); take structural enums from the
  specimen's code and re-derive them on every new fixture.
- Assert that the mapper's key set equals the live `answer_ids`, and read every delivery in
  `artifacts[]` for Mastery.
- Searchable techniques: descriptor/material codec decoding, Batch control signatures, partial-credit
  bisection, Mastery multi-delivery bundles.

## Evidence

Leaf base path: `challenges/day05/script-analysis/script-analysis-day-05-python-carrier-lab/`

- `single-solve/journal.md` (decoding, audits, "SOLVED - partial-credit bisection"),
  `single-solve/probe-final-1-response.json` (graded CORRECT), `single-solve/probe-*-response.json`
  (chunk results), `single-solve/submission-response-*.json` (the 14 `incorrect` full maps).
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `accepted_map.json`,
  `submit-1.out` (expired), `submit-2.out` (CORRECT); `time-trial/ranked-1/result.json`.
- `mastery/journal.md`, `mastery/artifacts/set_answer_map.py`, `mastery_merge.py`,
  `accepted_map.json`, `submit-1.out`; `mastery/ranked-1/result.json`.
- General solver: `tools/ranked/solvers/script-analysis-day-05-python-carrier-lab/solve.py`
  (with `tools/ranked/solvers/_shared/bank45.py`).
- Progress rows: `progress.md`, lines starting `| 5 |` for this challenge.
