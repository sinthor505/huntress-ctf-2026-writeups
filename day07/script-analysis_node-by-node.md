# Node by Node

| | |
|---|---|
| Day / Lane | 7 / Script Analysis |
| Challenge ID | `script-analysis-day-07-node-operation-graphs` |
| Technique | Static descriptor recovery from inert scripts, then rubric mapping (role, ATT&CK ID, Batch/VBScript control class) |
| Single Solve | Independent (469 fields, CORRECT on the 20th submission of a second 20-submission authorization) |
| Time Trial | Independent (CORRECT on the first submission; two later ranked reruns also CORRECT) |
| Mastery | Independent (CORRECT on the first submission; one later ranked rerun also CORRECT) |

## Summary

The prompt promises "eight Node.js operation graphs", but the archive is the whole Day 6 script bank (PowerShell, JScript,
Batch, VBScript, Python, Bash) plus eight new Node.js `.mjs` files, and the 469 answer fields cover all of it. Every specimen
hides a small pipe-delimited descriptor behind a source-declared byte codec; decoding it statically gives the ordered
behavior graph, and the questions are answered from that graph. The decoder was easy. The Single Solve cost 40 submissions
because the validator gave only batch-level feedback and four or five fields were classified in a way the prompt text did not
predict. Time Trial and Mastery then went through in one call each with the consolidated library tools.

## The challenge

- Single Solve: one `evidence.zip` (66 KB, AES-encrypted, password `infected`), 61 script files: 4 PowerShell, 9 JScript,
  9 Batch (5 `.cmd`, 4 `.bat`), 9 VBScript, 14 Python, 8 Bash, 8 Node.js. 469 answer IDs, all string-typed. Formats by
  count: 272 `exact_text`, 105 `enum`, 58 `pattern`, 16 `integer`, 16 `ordered_tokens`, 2 `mitre_attack_id`.
- Per specimen the questions ask for the role (benign_utility, stager, dropper, downloader, uploader, launcher,
  persistence_descriptor, staged_chain, relay, impact), an ATT&CK technique selected by a stated precedence rule, the
  reconstructed adapter, campaign ID, URL or path, the stage count, and the ordered step list (joined with `>`). Batch and
  VBScript files also get a "control flow" question (how the decode call is wrapped) and Batch gets an "expansion" question.
- The three PowerShell files are scenario questions (resource hijacking, browser credential collection, file-encryption
  impact) with literal indicators (URL, wallet, mutex, filename, user agent).
- Time Trial: same shape, 469 fields, fresh bytes (65 KB archive). Mastery: 938 fields = two independently generated copies
  of the same 61-file archive, nested as `evidence-01.zip` / `evidence-02.zip` inside one `mastery.zip`, with answer IDs
  prefixed `set1.` / `set2.`.

## Solution

1. **Check the archive.** SHA-256 matched the start response; AES members need `pyzipper`, not stdlib `zipfile`. All 61
   members extracted and were treated as inert text; nothing was executed.
2. **Find the carrier.** Each Node file declares `const Material_<x>` (a string or an array literal) and `const Codec_<x>`
   (a label of the form `representation|transform|key`). The loader code in the file is decoration; the label is the
   authority on how the bytes were encoded:

   ```python
   mm=re.search(r'\bconst\s+Material_\w+\s*=\s*("(?:\\.|[^"\\])*"|\[[^;]*?\]);',s,re.S)
   cm=re.search(r'\bconst\s+Codec_\w+\s*=\s*"(.*?)"',s,re.S)
   rep,trans,key=codec_parts(cm.group(1))     # e.g. escaped_octets | rolling_xor | ...
   lit=ast.literal_eval(mm.group(1)) if mm.group(1).startswith('"') else json.loads(mm.group(1))
   ```
3. **Invert the transform.** The Node variants seen were Base64, hex, escaped `\xNN` octets, byte reversal, single-byte
   XOR, add-mod-256, and a rolling XOR whose running state is the previous *ciphertext* byte (decode with
   `out[i] = c[i] ^ c[i-1]`, seeded from a constant in the source). The other languages used the same idea with their own
   carriers: Base64 JSON records in PowerShell, numbered `set` fragments in Batch, `Const Material_/Codec_` in VBScript,
   `MATERIAL_*`/`CODEC_*` constants (read with `ast.literal_eval`) in Python.
4. **Read the descriptor.** Every specimen decoded to a line like
   `v=1|steps=profile_host,acquire_remote,write_file,change_mode,dispatch_process|adapter=fetch_platform_fallback|stage_count=5|campaign_id=NODE-4C260462|url=...|file_path=...|service_name=...`.
   The eight Node graphs were: benign (`decode_configuration,return_configuration`), stager (`acquire_remote,dispatch_memory`),
   dropper (`decode_embedded,write_file`), downloader (`acquire_remote,write_file`), uploader (`read_file,transmit_remote`),
   launcher (`locate_local,dispatch_process`), persistence (`build_schedule,register_descriptor`) and the five-step staged chain.
5. **Map to answers.** Role comes from the terminal shape of the step list; the ATT&CK ID follows the prompt's precedence
   (terminal dispatch gives the language sub-technique, `acquire_remote` gives T1105, `decode_embedded` T1140, upload or
   transmit T1041, transform plus recovery note T1486, benign gives `none`). Other fields are the descriptor's literals,
   the step list is joined with `>`, and `stage_count` is the descriptor's own value. Accepted Node values included
   `T1059.007` for the launcher and for the staged chain, and `T1053.003` for persistence.
6. **Serialize as strings.** Integers go in as base-10 strings. The first POST sent JSON numbers for `stage_count` and was
   refused with `invalid_answers`.

The final accepted Single Solve map is in the leaf's `candidate-answers.json`; it was produced by `answer_map.py`, which
reads the exact start response and emits one value per answer ID.

## Time Trial and Mastery

Both modes regenerated the specimens (new campaign IDs, URLs, paths and, importantly, new Batch structure), so no answer
value or per-file override from Single Solve carried over. For these we used the consolidated library decoders
(`material_codec_decode` for descriptors, `behavior_rubric` for role/ATT&CK/control classification) and a thin per-leaf
`answer_map.py` that resolves each question suffix against the specimen's own decoded field names. Both decoded all specimens
with zero errors, passed an answer-format preflight (all IDs present, enums in the accepted list, strings only) and were
CORRECT on the first submission.

Mastery needed one extra step: unpack the outer ZIP, then each `evidence-0N.zip`, strip the `setN.` prefix, and run the
single-set mapper per set. Each set's rubric values were re-derived from its own bytes.

The later ranked reruns (50 s Time Trial window, 80 s Mastery window) used the same logic as a packaged solver and were
CORRECT in one call each, about 2.2 to 2.6 s end to end including the download and submit.

## Dead ends and gotchas

- **Batch controls decide the difficulty.** All Batch `*-control` answers were first guessed as `for_once` because the
  prompt says a one-item `FOR /L ... DO CALL :LABEL` wrapper must be classified `for_once`. The validator disagreed:
  - `batch-benign-control`: `goto_dispatch`, `for_once` and `direct` were rejected; `if_guard` was accepted. The `if defined`
    accumulator initialization was the operative guard; a `GOTO` in the host envelope was noise.
  - `batch-uploader-control`: the source was exactly the one-item `for /l ... do call :HUNTRESS_RECOVER` wrapper. `for_once`
    was rejected in isolation, `call_subroutine` was accepted. So the prompt rule and the validator contradict each other
    here, and the validator wins.
  - Time Trial's same-named `outbound-transfer.cmd` had a different structure (an explicit `goto`/label pair around a
    four-item `for`) and was `goto_dispatch`. File names do not carry structure across fixtures.
- **Persistence sub-technique.** The rubric says "scheduled-task persistence" without an ID. The descriptor's adapter was
  `systemd_descriptor` (suggesting T1053.005) but its `schedule_path` was under `/etc/cron.d/`. Submitted alone,
  `T1053.003` (cron) was accepted. The concrete path beat the adapter name; the same rule was used on the Time Trial fixture.
- **Batch expansion.** `batch-benign-expansion` was `for_indirect` (a `FOR` variable builds the `fragmentN` name), not the
  `call`-doubling reading; isolated submission confirmed it.
- **Batch-level `incorrect` is unreadable.** Submitting all 174 unconfirmed fields twice, with one field changed in between,
  told us nothing. Submissions by format, then by language or specimen, then single fields found the culprits. The whole
  `exact_text` group (272 fields) was accepted on the first try; the cost was almost entirely in enums and patterns.
- **VBScript control.** The wrapper to classify is the block enclosing the decode call, not later `For...Next` loops that
  merely convert bytes to text.
- **Language sub-technique for chains.** Python chains ending in `dispatch_process` (the py2exe service and pyarmor
  specimens) took `T1059.006` even though their graphs contained `acquire_remote` earlier; Node.js and Bash chains likewise took the
  language sub-technique here (`T1059.007`, `T1059.004`). This did not hold on every later day; see the day overview and
  the library's per-day staged-chain table.
- A serialization slip (numbers instead of strings) and a duplicated 174-field submission each cost a counted attempt; those
  are our own mistakes, collected in the day overview.

## Takeaways

- **Descriptor reconstruction** (decode the declared `Material_`/`Codec_` pair, ignore the loader prose) is the right first
  move for this whole family of challenges; it was fully reusable (now `material_codec_decode`).
- **Take the validator's verdict over the prompt's rule** when an isolated, single-field submission contradicts it, and write
  the discrepancy down. Batch controls and persistence sub-techniques both behaved that way.
- **Classify structure per fixture, never per file name or from another mode.** Time Trial's answers differed from Single
  Solve's for the same file name.
- **Bisect big maps early** (by format, then by specimen, then singletons) instead of resubmitting the whole map. See the
  multi-answer diagnostic approach used throughout this event.
- Run an answer-format preflight (strings only, enums in the accepted list) before the first POST.

## Evidence

Leaf base path: `challenges/day07/script-analysis/script-analysis-day-07-node-operation-graphs/`

- `single-solve/journal.md` (candidate map, attempt tables, final audit, authorized continuation)
- `single-solve/artifacts/static_decode.py`, `answer_map.py`, `candidate-answers.json`, `decoded-descriptors.json`,
  `audit-unresolved.md`
- `single-solve/artifacts/authorized-extra-20-20261020/` (the 20 continuation requests/responses; `response-attempt-20.json`
  is the CORRECT)
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/` and `ranked-2/` (`result.json` CORRECT)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json` (CORRECT)
- Repo-level: `progress.md` rows for Day 7; per-challenge solver `tools/ranked/solvers/script-analysis-day-07-node-operation-graphs/solve.py`;
  library entries on descriptor reconstruction, behavior rubric, Batch control classification and the Day 5-9 gotchas.
