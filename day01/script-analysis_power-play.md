# Power Play

| | |
|---|---|
| Day / Lane | 1 / Script Analysis |
| Challenge ID | `script-analysis-day-01-native-signals` |
| Technique | Static reading of inert PowerShell samples: scenario classification, ATT&CK mapping, literal indicator extraction |
| Single Solve | Solved, graded result not preserved (local mark only) |
| Time Trial | Independent (ranked attempt, `CORRECT` on the first submission) |
| Mastery | Independent (ranked attempt, `CORRECT` on the first submission) |

## Summary

Three PowerShell specimens (resource hijacking, browser-credential collection, ransomware-style
impact) are never executed; the task is to classify each, give its primary ATT&CK technique and copy
out the indicators embedded in plain sight. Every question has a fixed answer shape per sample
type, so the Time Trial and Mastery were scripted and went `CORRECT` in one submission each. This
is the thinnest page in the set: the leaf has no journal and the Single Solve artifact is no longer
readable, so the details below come from the start responses and the accepted answer maps.

## The challenge

Prompt: "Three PowerShell specimens are on the table, and none of them will ever run. ... Classify
each script's intent, map its primary ATT&CK technique, and pull out the exact indicators sitting
in plain sight."

An `evidence.zip` holds `resource-sample.ps1`, `collection-sample.ps1` and `impact-sample.ps1`.
Each sample gets five questions, 15 answers in total:

| Sample | Questions |
|---|---|
| resource | scenario (enum), ATT&CK ID, URL, wallet, mutex |
| collection | scenario (enum), ATT&CK ID, URL, first filename indicator, user agent |
| impact | scenario (enum), ATT&CK ID, extension, recovery-note filename, URL |

The scenario enum is `resource_hijacking` / `synthetic_collection` / `synthetic_impact`, one per
sample. The collection ATT&CK question is worded to steer the choice: treat "access to the browser
credential store as the central behavior rather than ZIP staging or network transfer", and note
that the broader Data from Local System technique is also accepted. Free-text fields are
`exact_text` with trim normalization and no quotes.

## Solution

The Single Solve record is thin (see "Evidence"), so this section describes the reasoning that the
accepted answers imply.

1. Open the three scripts as text. They are inert and the indicators are literal strings, so no
   deobfuscation or execution is needed.
2. Classify by behavior. The scenario for each sample is fixed by its type: resource sample =
   `resource_hijacking` (miner-style, with a wallet and a mutex), collection sample =
   `synthetic_collection` (browser credential store theft), impact sample = `synthetic_impact`
   (file encryption with a recovery note).
3. Map the primary technique from the central behavior, not the surrounding plumbing. The accepted
   Time Trial and Mastery answers were `T1496` (Resource Hijacking), `T1555.003` (Credentials from
   Web Browsers) and `T1486` (Data Encrypted for Impact).
4. Copy the indicators exactly. In the accepted maps they have these shapes:
   - resource: an `https://<host>.example.invalid/<hex>.dat` URL, a `wallet_<32 hex>` value and a
     mutex of the form `Global\HuntressLab_<hex>`;
   - collection: an `.example.invalid` URL, a file named `update-<host id>.dat` and a user agent
     `Mozilla/5.0 HuntressLab/<n.n>`;
   - impact: an extension `.locked-<5 hex>`, a note `RECOVERY-<10 hex>.txt` and a URL.
5. Submit all answers as JSON strings in one call, using the answer IDs from the live start
   response.

The Single Solve flag/answers are not preserved in the leaf, so none are quoted here.

## Time Trial and Mastery

- **Time Trial:** same three samples, 15 answers, a 50 s server-enforced window. This was a
  legacy-route day, so a plain `retry` was itself the ranked attempt, and the solve was scripted
  (issue, download, extract, submit). The recorded pass took 1.9 s end to end (solve 0.03 s) and
  was `CORRECT` on call 1.
- **Mastery:** two deliveries, each its own `evidence.zip`, giving 30 answers (IDs prefixed
  `set1.` and `set2.`), under an 80 s window. The URL, wallet, mutex, file name, extension and
  note values differ per set, while the scenario and ATT&CK answers were identical to the Time
  Trial's (`T1496`, `T1555.003`, `T1486`). Pass time was 2.6 s and `CORRECT` on call 1.

Fixtures differ per attempt (the Time Trial and Mastery set 1 indicators are different values),
so the extractor reads the indicators from the downloaded bytes rather than reusing any
earlier answer.

## Dead ends and gotchas

- The local Single Solve `evidence.zip` is unreadable (a 2,470-byte file; the library notes it as
  unreadable and the cause was not recorded; other early downloads were corrupted by line-ending
  translation, but that is not confirmed here). It could not be re-analysed afterwards.
- Answers must be JSON strings and must use the exact IDs of the live start response. Mastery IDs
  carry the `setN.` prefix.
- The collection ATT&CK question explicitly accepts a broader technique, but the accepted
  Time Trial and Mastery answers used the specific sub-technique `T1555.003`.
- Time Trial and Mastery windows of 50-80 s leave no room for manual reading, which is why the
  extraction was scripted.

## Takeaways

- With a fixed record family (resource / collection / impact), the answer to every scenario and
  ATT&CK field is determined by sample type; only the indicators need extracting from the bytes.
  The same PowerShell samples reappear in the Day 2-4 banks, so a reader written for Day 1 carries
  forward (Day 2 *Host With the Most* extends the same bank).
- Save the artifact and verify its SHA-256 on download; this challenge's Single Solve write-up
  would have been far better with the original bytes and a journal.
- Name to search for: *static PowerShell indicator extraction*, *MITRE ATT&CK mapping by central
  behavior*.

## Evidence

Leaf base path: `challenges/day01/script-analysis/script-analysis-day-01-native-signals/`

- `single-solve/evidence.zip` - unreadable local copy; no journal, `start_response.json` or result
  file. `progress.md` row 1 records it as solved on 2026-10-18, basis "Player-directed local
  mark", so no graded result survives.
- `time-trial/ranked-1/{start_response.json, accepted_plan.json, result.json, pass.log}` -
  `CORRECT`, 1 call, 15 answers.
- `mastery/ranked-1/{start_response.json, accepted_plan.json, result.json, pass.log}` -
  `CORRECT`, 1 call, 30 answers, 2 deliveries.
- Ranked passes were run on 2026-10-26; `challenges/submission-ledger.json` shows one call each.
