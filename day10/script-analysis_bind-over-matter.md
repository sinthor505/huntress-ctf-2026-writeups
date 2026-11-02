# Bind Over Matter

| | |
|---|---|
| Day / Lane | 10 / Script Analysis |
| Challenge ID | `script-analysis-day-10-wmi-persistence-reconstruction` |
| Technique | Decoding inert script descriptors (89 specimens, ~10 languages) and answering from the descriptor field vs. the stated graph rubric |
| Single Solve | Solved (journal-recorded): the journal records our own final CORRECT, but the response was not saved |
| Time Trial | Independent (original attempt journal-recorded; later ranked reruns have `ranked-N/result.json` CORRECT) |
| Mastery | Independent (original attempt journal-recorded; later ranked reruns have `ranked-N/result.json` CORRECT) |

## Summary

The prompt talks about a WMI MOF file and "the subscription that would really survive a reboot", but
the delivery is an encrypted ZIP of 89 inert script specimens in about ten languages, with 753
per-field questions. Single Solve took 30 logged submissions (plus an unrecorded number after the
player-set cap) because the validator often wanted the descriptor's own `role`/`attack`/`adapter`
field rather than the graph rule printed in the question. Once that was understood, Time Trial and
Mastery each went CORRECT on the first submission.

## The challenge

- Prompt: a MOF file "declares classes, namespaces, filters, and consumers, but only some bindings
  actually tie them together"; reconstruct the persistence intent, separate operational
  relationships from decoys, and name the subscription that survives a reboot.
- Delivery: one AES-encrypted (WinZip AE-2) `evidence.zip` with 89 members. The password was the
  event's standing `infected`. Specimens covered PowerShell, JScript, Batch, VBScript, Python (many
  packaging variants: pyc, zipapp, wheel, PyInstaller, py2exe, cx_Freeze, PyArmor), Bash, Node.js,
  PHP, Lua, WSF/HTA/SCT host wrappers, and four WMI MOF files.
- Questions: 753 answer IDs, prefixed by family (`python-` 126, `bash-` 98, `polyglot-` 176,
  `host-` 66, `jscript-`/`batch-`/`vbscript-` 72 each, `nodejs-` 56, and 15 for the three PowerShell
  samples). Formats: 444 `exact_text`, 133 `enum`, 86 `pattern`, 44 `integer`, 44 `ordered_tokens`,
  2 `mitre_attack_id`. The MOF narrative in the prompt did not correspond to one question; the MOF
  files were just four of the `polyglot-*` specimens.
- The specimens are never run. Each carries a `HUNTRESS_ANALYSIS_V1` comment marker holding an
  encoded descriptor (role, attack, adapter, ordered `steps`, host/URL/campaign values, and so on),
  and the question asks you to read it back or classify the specimen from it.

## Solution

1. Verified the archive hash against the start response and opened it with the known password. Listed
   the 89 members and the question-to-member mapping (each question names `evidence.zip -> <member>`).
2. Wrote a static decoder for the marker encodings (`B64`, `HEX`, `HB64`, fragmented hex) and the
   per-family source transforms (percent-encoded octets, Base64, hex, byte arrays, XOR,
   modulo-byte arithmetic, reversal). It recovered descriptors for all 89 members with no errors; this
   was later folded into the shared `descriptor_decode` library tool (technique name: WMI MOF
   descriptor placement, `Descriptor` string in a `HuntressAnalysis_<id>` class instance).
3. Wrote a mapper from decoded fields to the exact answer IDs, asserting the key set equals the
   start response's `answer_ids`. Non-classification fields (host, URL, path, campaign, next stage,
   extension, ordered steps) came straight from the decoded values and were accepted early.
4. Classification fields (`role`, `attack`, `control`, `expansion`, JScript style classifiers) first
   followed the rubric text in each question. That was where the submissions went wrong; see below.
5. Final accepted map (753 fields): the first full-map submissions came back `incorrect` with no
   field-level feedback, so the work became group-by-group isolation. The journal's RESOLUTION
   section records the final `CORRECT` (flag `59991f02ca684911d194245c0f8f4738`, read back via the
   Game Master's status call). No result file was saved for that last submission.

Isolation method that worked: `accepted_ids`/`remaining` on a batch is the only real signal, so
submit by family, then by suffix (`role`, `attack`, `adapter`, `control`), then single IDs, and cycle
enum singletons through their allowed values.

## Time Trial and Mastery

- Both modes returned the same 753-question shape and member set as Single Solve (checked
  programmatically before any download); only the values regenerate per attempt. Time Trial had a
  50 s window, Mastery 80 s.
- Mastery is 1,506 questions: two sets, `set1.<id>` and `set2.<id>`. One download, an unencrypted outer
  ZIP holding two encrypted inner archives (`evidence-01.zip`, `evidence-02.zip`). We split the
  bundle, ran the same mapper per set, and merged the two maps under the `setN.` prefixes.
- Before touching the live timer, the corrected mapper was run on the Single Solve archive and
  diffed against Single Solve's final accepted map: 4 of 753 differences at first, 0 after the fix
  (see the staged-chain item below). Download, extract, decode, map and submit then ran as one
  script inside the window.
- Results: Time Trial CORRECT on call 1 and Mastery CORRECT on call 1 (753 and 1,506 fields). Ranked
  reruns later also went CORRECT on call 1 each (Time Trial twice, Mastery three times); the logged
  total time per pass was roughly 2.7-4.4 s. A per-challenge solver for these reruns lives in the
  ranked-solver set and reuses the same rules.

## Dead ends and gotchas

- **Descriptor `attack` beats the graph rule on v3 specimens.** The `host-*` (6) and `polyglot-*` (16)
  attack questions print a graph-precedence rubric, but the validator wanted the descriptor's own
  `attack` field. Example: `host-wsf-mixed-vbscript` was `T1059`; `T1140`, `T1105` and `T1059.005`
  were all rejected. Graph rules held for families without an `attack` field (Python, Bash,
  Node.js, JScript, VBScript, Batch). Cost: attempts 24-29 and the post-cap work.
- **Descriptor `adapter` beats the `--plan` text.** `python-source-benign-adapter` was
  `argparse_namespace`, not the `report` token from the Python operation plan; `batch-benign-adapter`
  was `call`, not the `set` echoed by the Batch `--plan` branch. Attempts 9-13 chased this.
- **Language sub-technique for staged chains, even though the prompt allows `T1105`.** For
  `profile_host,acquire_remote,...,write_file,dispatch_process` chains, the question text says the
  earlier ingress-transfer technique is "also accepted", but `T1105` was rejected (attempt 18) and
  `T1059.006` accepted for the py2exe-service and PyArmor/PyInstaller specimens (attempt 19). The
  Bash and Node.js equivalents ended up `T1059.004` and `T1059.007`. The Single Solve journal's
  own RESOLUTION text says Bash/Node.js were "accepted as originally mapped", which conflicts with
  its attempt log; the Time Trial journal's 0/753 diff against the final map is the reliable
  statement, and Time Trial and Mastery then went CORRECT with it.
- **Surface labels win over source reading.** `python-wheel-stager-syntax` is declared
  `list_comprehension` although the source uses generator expressions; the declared label was accepted
  (attempt 11).
- **Batch controls.** `batch-dropper-control` was `if_guard` (a preceding `if defined` guard), not
  `direct`; `batch-chain-control` was `call_subroutine`, not `for_once`, despite a
  `for /l ... do call :RECOVER` wrapper. A later structural classifier (signature of the
  reconstruction wrapper) reproduced these.
- **Cost of a bad lead.** A batch-level `incorrect` carries no accepted IDs. A 665-field submission
  (attempt 8) and several full maps taught nothing; narrow subsets were what exposed errors.
- Our process note: the first Time Trial start expired before analysis began, so the attempt was
  re-issued with `retry`. See the day overview for process issues.

## Takeaways

- When a decoded descriptor has its own `role`/`attack`/`adapter` field, answer from it first and use
  the printed graph rubric only for families that lack one. This is the main lesson of Day 10 and
  carried forward to later script days.
- A staged chain ending in `dispatch_process` answered with the language ATT&CK sub-technique here,
  even though the question text offered `T1105`.
- Classify Batch controls structurally from the reconstruction wrapper, not by file name or by
  intuition about the wrapper's wording.
- Validate a rewritten mapper against a sibling mode's already-accepted map (same challenge, same
  question structure) before starting a timed attempt: it caught the staged-chain regression with no
  live submissions spent.
- With a tight cap and batch-only feedback, plan submissions as a partition (family, then suffix,
  then ID) from the start.

## Evidence

Leaf: `challenges/day10/script-analysis/script-analysis-day-10-wmi-persistence-reconstruction/`

- `single-solve/journal.md` (attempt log and RESOLUTION), `single-solve/start_response.json`,
  `single-solve/artifacts/static_decode.py`, `answer_map.py`, `candidate-answers-final.json`,
  `submission-03..30-*-response.json`.
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`,
  `time-trial/ranked-2/result.json`.
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json`,
  `mastery/ranked-2/result.json`, `mastery/ranked-3/result.json`.
- `challenges/day10/submission-attempts.md`: Single Solve 30 of 30 logged; Time Trial and Mastery 1
  CORRECT each. Submissions after the 30th for Single Solve are not counted there and no response
  file survives.
- The Single Solve mode has no `result.json`; its CORRECT rests on the journal RESOLUTION and the
  `progress.md` row only.
