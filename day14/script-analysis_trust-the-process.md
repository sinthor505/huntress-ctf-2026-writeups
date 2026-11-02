# Trust the Process

| | |
|---|---|
| Day / Lane | 14 / Script Analysis |
| Challenge ID | `script-analysis-day-14-msbuild-trusted-project-abuse` |
| Technique | inert-descriptor reconstruction across a script corpus; new container: MSBuild `.proj` XML |
| Single Solve | Independent (journal and saved response record CORRECT on submission 1) |
| Time Trial | Independent (journal and saved response record CORRECT on submission 1; a later ranked rerun also has a `result.json` CORRECT) |
| Mastery | Independent (journal and attempt log record CORRECT on submission 34; a later ranked rerun has a `result.json` CORRECT on call 1) |

## Summary

Day 14's script-analysis challenge is the usual 971-field "recover the descriptor behind every specimen" bundle, with
a new container type added: four MSBuild project files. Despite the title and prompt, none of the MSBuild
content is executable behavior. Each `.proj` carries the same inert descriptor the other families carry, so
the work was decoding it correctly and answering from the declared fields. Single Solve and Time Trial went
CORRECT with the full map on the first submission. Mastery took 34 submissions, almost all spent isolating a few
Batch-control fields whose accepted values contradicted the question text.

## The challenge

Prompt: "An MSBuild project looks like routine build plumbing until you follow its properties into its targets.
Recover the property and target flow, compare it with earlier execution chains, and find where a trusted
toolchain is doing untrusted work."

Delivered: an AES-encrypted `evidence.zip` (password `infected`) with 111 members (about 205 KB
unpacked) across PowerShell, JScript, Batch/CMD, VBScript, Python, Bash, Node.js and polyglot/host specimens,
plus the four `.proj` files. `start` returned 971 questions with 971 answer IDs: 574 `exact_text`, 155 `enum`,
108 `pattern`, 66 `integer`, 66 `ordered_tokens`, 2 `mitre_attack_id`. 110 members are referenced by questions. The
questions per specimen are descriptor fields: `role`, `attack`, `adapter`, `campaign`, `url` or `file-path`,
`stage-count`, `steps`, `host-profile`, `obfuscation`, `stage-profile`, `next-language`, plus per-family
extras such as Batch/VBScript `*-control`. The four `.proj` files account for 44 of the 971 fields.

## Solution

1. **Recon.** The archive was extracted offline as data; nothing was executed. Each `.proj` has the same shape: an XML
   comment `<!-- HUNTRESS_ANALYSIS_V1:<kind>-<payload> -->`, then a `<Project>` with a `PropertyGroup` holding a
   `HuntressDescriptor` property, and a single `Target Name="Analyze"` whose only task is a low-importance
   `Message` ("Inert analysis fixture"). There is no `Exec` or other task, so "trusted toolchain doing untrusted
   work" is a story told by the descriptor, not something to trace through targets.
2. **Decoding the marker.** The four projects use four different payload encodings (`B64`, `HB64` = hex-wrapped
   base64, `FRAG` = dot-separated hex fragments, `HEX`). The decoder we carried over from the previous days
   decoded 107 of 111 members and failed on exactly the four `.proj` files: in XML the marker and the
   comment terminator share one line, so the regex captured the trailing `-->` as part of the payload. The fix was
   one line:

   ```python
   marker = re.sub(r'\s*-->\s*$', '', m.group(1).strip())
   kind, sep, payload = marker.partition('-')
   ```

   After that all 111 members decoded with zero errors. As a cross-check, the `HuntressDescriptor` property
   value in each project decodes under the same declared representation to the same bytes as the comment marker.
3. **What the descriptors say.** A decoded `.proj` descriptor is a v3 pipe-delimited record, for example
   `language=msbuild_xml|role=staged_chain|attack=T1059|adapter=operation_graph|...|steps=acquire_remote,dispatch_process|stage_count=2`.
   The four projects cover four roles (`benign_utility`, `downloader`, `launcher`, `staged_chain`), each with its
   own campaign ID, URL, file path and stage profile.
4. **Mapping to answers.** The mapper reads the start response for the exact answer IDs, resolves each question to
   its named member's decoded fields, and applies the rubric where a field is not a plain lookup. For polyglot and
   host-style specimens that carry an `attack` field, the descriptor's own value is the answer rather than a
   graph-derived technique; `adapter` is likewise the descriptor value. A pre-submission check confirmed the
   candidate key set equaled the API's `answer_ids`, all values were strings and format-valid.
5. **Submission.** The full 971-field map was submitted in one call: `CORRECT`.

## Time Trial and Mastery

**Time Trial** was a fresh archive (new hash, same 971-ID schema, 50 s window). The decoder and mapper from Single
Solve were run unchanged against the new bytes and start response, and the full map went CORRECT on the first
submission inside the window. A ranked rerun on 2026-10-26 (a scripted download, solve and submit pass) took 2.2 s
end to end and was also CORRECT in one call.

**Mastery** was a bundle of five evidence sets (outer ZIP holding `evidence-01.zip` .. `evidence-05.zip`, 555
members, 4,855 answer IDs prefixed `setN.`), under a 170 s window. The first timed assignment was different:
two sets, 1,942 IDs, 80 s. Submitting that full map returned `incorrect`, and the Game Master gives no per-field
feedback on an incorrect batch, so we spent the next calls on a split-by-family strategy against fresh
retries:

- On a five-set retry, PowerShell, JScript, VBScript, Node.js, Python, Bash, polyglot/host and Batch non-control
  groups were each accepted. The 45 Batch `*-control` fields (9 per set) were what returned `incorrect`.
- Several retry windows expired mid-diagnosis, and an acceptance on one assignment does not carry to the next
  retry's different fixtures.
- On a final untimed retry, 4,810 non-control fields were accepted in one call, then the 45 controls were worked
  through in cohorts by structural type: 6 `goto_dispatch` and 3 `direct` were accepted together; `errorlevel_guard`
  (8) accepted; four literal `if_guard` reconstructions accepted; the last five (guarded accumulator
  initializers) returned CORRECT on submission 34.

## Dead ends and gotchas

- **The `.proj` marker terminator** (above) is the only new decoding trap this challenge introduced: a decoder that
  works for every other container silently fails on exactly the new one.
- **One-item `FOR /L` Batch wrappers validated as `call_subroutine`, not `for_once`.** The question text
  says a one-item `FOR /L` loop invoking a subroutine that holds the reconstruction should be `for_once`. The mapper
  followed that and was rejected on singleton probes (`set1.batch-launcher-control`, `set2.batch-stager-control`);
  `call_subroutine` was accepted for those and then for set 3 stager/impact and the remaining
  one-item cases. We kept this as a per-ID override rather than changing the general rule, because the library
  also records `for_once` being accepted for similar wrappers on another day; the answer varies by fixture.
- **Control fields decide Mastery.** Once the 4,810 non-control fields were accepted on the final retry (submission
  15), the remaining 19 submissions all went to the 45 Batch controls. Group them by structural signature (separator `goto`, `ver`/`errorlevel` guard,
  `if defined` guard, one-item `FOR`) and submit one group at a time; a rejected group says nothing about
  which member is wrong, and a one-field group is the only way to localize.
- **`attack` for Bash and Node.js staged chains was `T1105`** in every mode on this day (not the language
  sub-technique that other days accepted). The ranked solver keeps the other reading as a variant but never needed it.
- A bookkeeping gap: the shared submission ledger lost this challenge's counts; the per-leaf attempt log is the
  authoritative record (34 entries, CORRECT on the 34th). See the day overview.

## Takeaways

- Technique: **inert descriptor reconstruction with a source-first decoder.** When a new file type shows up in a
  familiar bundle, run the old decoder first and read which members it fails on; here the failures (4 of 111) named
  the new container and its quirk immediately.
- The MSBuild theme was framing. Do not trace `Target` flow looking for behavior unless the file has real tasks;
  check what the project actually executes (here: one `Message`) and use the descriptor.
- For large multi-answer maps, bisect by family and then by structural signature. Expect the long tail in
  whichever family has rubric-ambiguous fields (Batch controls), and let the first accepted value of a
  structural class generalize only within that fixture, since the answer for the same shape differed between
  assignments.

## Evidence

Leaf base: `challenges/day14/script-analysis/script-analysis-day-14-msbuild-trusted-project-abuse/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/attempt-01-response.json`
  (CORRECT, 971 fields), `single-solve/artifacts/static_decode.py`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/control-evidence.md`
- `time-trial/journal.md`, `time-trial/artifacts/attempt-01-response.json`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/submission-attempt-log.json` (34 entries),
  `mastery/retry3/artifacts/validator-corrections.json`, `mastery/retry3/artifacts/confirmed-control-answers.json`,
  `mastery/ranked-1/result.json`
- `tools/ranked/solvers/script-analysis-day-14-msbuild-trusted-project-abuse/solve.py` (ranked-pass solver)
- Library: descriptor reconstruction, inert-marker decoding, Batch control classification (script-analysis)
- `progress.md` (grep `^| 14 .*msbuild`)
