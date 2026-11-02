# Return to Sender

| | |
|---|---|
| Day / Lane | 16 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-16-return-to-sender` |
| Technique | RFC 5322 `Message-ID` header read from an `.eml`, inside the usual pooled evidence bundle |
| Single Solve | Independent (our own `CORRECT` response is saved in the leaf; no `result.json`) |
| Time Trial | Independent (original untimed attempt, plus a ranked rerun with `result.json`) |
| Mastery | Independent (original untimed attempt, plus a ranked rerun with `result.json`) |

## Summary

The flavour text promises mail-threading forensics ("replies, forwards, duplicates, and
reconstructed messages ... stable message identities"), but the only new field was
`email.message_id` of one `.eml`: the verbatim `Message-ID` header, angle brackets included. The
Time Trial and Mastery were the same 21-field mixed-evidence bundle the previous days used, with the
EML added. Every mode was `CORRECT` on the first submission.

## The challenge

Prompt: "A mailbox export contains replies, forwards, duplicates, and reconstructed messages. Use
stable message identities to rebuild which mail actually belongs together." Nothing in the delivered
data asks for any threading or deduplication.

- **Single Solve:** a ZIP holding one file, `message-001.eml`. One question,
  `d16-email-001-email-message-id`, `exact_text` with `trim` normalization.
- **Time Trial:** a 13-file evidence ZIP and 21 questions.
- **Mastery:** an outer ZIP of five nested `evidence-0N.zip` bundles, 21 questions each, 105
  answers, with IDs prefixed `set1.` to `set5.`.

The 21 Time Trial questions are the cumulative template: five questions about the note, its renamed
copy and the roster (filename, size, MD5, SHA-1, SHA-256, 16-byte prefix, UTF-8 BOM), one field per
earlier day's specimen (JPEG SOF dimensions, PNG IHDR, PDF header version, ZIP EOCD count and member
names, PE machine/entry RVA/original filename, .NET MVID, MSI ProductCode, LNK target path, OOXML
external targets), the new EML Message-ID, and a synthesis question (SHA-256 ordering of four
named files).

## Solution

1. **Parse the header with the stdlib.** The Single Solve extractor is three lines: open the ZIP,
   parse `message-001.eml` with `email.message_from_bytes(..., policy=policy.compat32)`, print
   `m["Message-ID"].strip()`. `compat32` returns the raw header string, so the angle brackets are
   kept:

   ```python
   m = email.message_from_bytes(z.read("message-001.eml"), policy=policy.compat32)
   print(json.dumps({"d16-email-001-email-message-id": m["Message-ID"].strip()}))
   ```

   The value was `<artifact-assembly-line@example.test>`.
2. **Keep the brackets.** The answer format said "exact artifact-derived value only", and the
   header is the artifact's value, so the brackets stayed. It was accepted on the first call.
3. **Time Trial.** The Day 15 to 19 bundle extractor was copied with the WAV, MP4 and ELF members
   made optional, since this bundle lacks them. The mapper was adapted from the Day 19 one. It
   resolves every field from the artifact's bytes. The synthesis answer is the four filenames sorted
   by their full-file SHA-256, serialized as compact JSON.
4. **Mastery.** The mapper opens each nested `evidence-0N.zip`, runs the same extractor, and routes
   each `setN.<id>` question to that set's facts. The five note/copy/roster questions are matched by
   regex on the question ID (`(evidence-note-copy|evidence-note|roster)-(filename|size-bytes|md5|sha1|sha256)`),
   and the set number comes from the ID prefix. All 105 answers went in one call.

## Time Trial and Mastery

Scale was the only change: 21 and 105 fields, no deadline on the original untimed attempts. The
per-set variation in Mastery was limited to the note, its copy, the roster and the other files that
differ per set. The journal states that the per-set variants only change the first five
note/copy/roster questions; the later library notes add that the JPEG, PNG, PDF, nested ZIP and the
telemetry `.exe` also differed per set while the DLL, MSI, LNK, DOCX and EML were byte-identical.
Because of that, the Message-ID answer is the same value in every set. We still recomputed each
set from its own bytes.

On 2026-11-01 both modes were rerun as ranked attempts (`run_for_time: true`, a bragging-rights
sweep, not the prize scoring). That needed a per-challenge solver, which delegates to the library's
generic resolver, `question_facts.py`. It was first checked offline against the previously accepted
answers: 21 of 21 for Time Trial and 105 of 105 for Mastery. Both went `CORRECT` on call 1:
Time Trial in 3.46 s total (50 s window), Mastery in 4.9 s total (170 s window).

## Dead ends and gotchas

- **The flavour text is not the task.** Nothing asked for threading, `In-Reply-To`/`References`
  chains or duplicate detection. Only the Message-ID was asked, and we read the prompt literally
  rather than building a mailbox analyser.
- **Angle brackets are part of the answer.** `compat32` and `policy.default` both keep them. Do
  not strip `<>`.
- **A constant is not a rule.** The EML and several other members were identical across all Day
  16 to 20 sets, so their answers repeat. The solver still recomputed from each set's bytes, which
  paid off on Day 23 when every fixture field took a new value per set.
- **Do not trust the Message-ID as the only identity in a general mailbox.** The specimen here has
  a single Message-ID, but the library notes warn that multiple `Message-ID` headers are legal
  forgeries and that the `X-Huntress-Correlation` and Subject tokens are not identities. This did
  not matter for scoring.
- **Mapper hygiene.** Answer IDs come from each mode's own start response, with
  `set(map) == set(answer_ids)` asserted before any submission.

## Takeaways

Technique: **`email.message_id` extraction** with the stdlib `email` parser, bracketed form
accepted. For the bundle modes, a manifest-driven extractor that handles optional members and nested
per-set ZIPs, with question IDs mapped by regex, covers this day and its neighbours. When a prompt
describes a rich scenario but the question list is a single field, solve the field. A faster
approach would have been to call the library's generic reader (`metaread.py`) instead of copying a
per-day extractor, which is what the ranked rerun did.

## Evidence

Leaf base: `challenges/day16/metadata-forensics/metadata-forensics-day-16-return-to-sender/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-01-response.txt` (CORRECT, attempt 1 of 20)
- `time-trial/journal.md`, `time-trial/artifacts/extract_facts.py`, `time-trial/artifacts/answer_map.py`,
  `time-trial/artifacts/submission-attempt-01-response.txt` (CORRECT, attempt 1),
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/submission-attempt-01-response.txt` (CORRECT, attempt 1),
  `mastery/ranked-1/result.json`
- `tools/ranked/solvers/metadata-forensics-day-16-return-to-sender/solve.py`
- `library/metadata-forensics/eml-message-identity.md` (META-EML-MESSAGE-ID-B106),
  `library/metadata-forensics/gotchas-d11-d20.md`
- `progress.md` (grep `^| 16 .*return-to-sender`)
