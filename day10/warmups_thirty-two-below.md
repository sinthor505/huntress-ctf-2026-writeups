# Thirty-Two Below

| | |
|---|---|
| Day / Lane | 10 / Warmups |
| Challenge ID | `warmups-day-10` |
| Technique | Base32 decoding; numbered codec chain (Mastery) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (own CORRECT in `ranked-1/result.json`; the original attempt is journal-only) |
| Mastery | Independent (own CORRECT in `ranked-1` to `ranked-4/result.json`; the original attempt is journal-only) |

## Summary

A warmup whose only job is to recognize the day's new encoding: padded Base32, hinted by the title
("Thirty-Two Below") and the prompt ("Same neighborhood. Smaller alphabet."). The decoded text is a
fixed sentence ending in the answer slug. Single Solve and Time Trial were one decode each; Mastery
was 48 small bundles that chain the codecs of Days 1-10.

## The challenge

- Prompt: "Same neighborhood. Smaller alphabet."
- One question per blob, answer ID `message` (Single Solve, Time Trial) or `setN.message` (Mastery):
  "Decode the assigned blob completely. What exact answer slug does its final non-empty decoded line
  instruct you to submit?"
- Answer format: a pattern, three lowercase words joined by hyphens plus a hyphen and a two-digit
  number (for example `adjective-noun-noun-NN`); submit the slug only; normalization `trim`.
- Single Solve: `blob.txt`, 184 bytes of ASCII with no line terminator, untimed. Time Trial: one
  blob of the same kind with a 20 s window. Mastery: one ZIP of 48 members (`blob-01.txt` to
  `blob-48.txt`), 48 questions, a 65 s window.

## Solution

1. Looked at the bytes. The Single Solve blob used only `A-Z`, `2-7` and trailing `=` padding, which is
   the Base32 alphabet. The riddle in the title (32 symbols) agreed with the byte evidence.
2. Checked the library for a Base32 entry. The existing text-codec notes covered Base64, ROT13 and hex,
   and the other warmup entries (numeric, JSON string, percent, HTML entity) did not fit these bytes.
3. Decoded the whole file with the standard library. The 184 characters decoded to 112 UTF-8 bytes and
   one line:

   ```python
   import base64
   print(base64.b32decode(open("blob.txt","rb").read(), casefold=False).decode("utf-8"))
   # Nice work decoding this warmup! To solve, submit this exact answer to the Game Master: groovy-starfish-willow-93
   ```

4. The slug after the final colon matched the declared pattern. Submitted `groovy-starfish-willow-93`
   under `message`; the Game Master returned CORRECT on the first submission (1 of 10 authorized).

## Time Trial and Mastery

**Time Trial.** Same codec on fresh text. The window was about 20 s, so the solve was one scripted
fetch, decode, submit pass using the library's `codec_chain.py` tool (strict `base32` layer). The slug was
checked against the full warmup sentence as well as the slug pattern, and the first submission was
CORRECT. A later ranked rerun (`ranked-1`) was also CORRECT in one submission call, 1.56 s end to end.

**Mastery.** The artifact was a ZIP of 48 `blob-NN.txt` members, mapped to questions by each question's
`artifact` field. The solve mapped member names to question IDs, not file order. The journal first
described these members as single Base32 blobs, then corrected itself in a later review: each member is
a 10-line chain of `NN<TAB>payload` records, where record N uses the codec of Day N (1 base64, 2 rot13,
3 hex, 4 binary, 5 decimal, 6 octal, 7 percent, 8 HTML references, 9 JSON string, 10 base32). The
tool detected both shapes, and `codec_chain.py` reported 10 records for all 48 members. All 48 slugs
matched the sentence template with no repairs, and the one batched submission of 48 answers was CORRECT.

Because the first attempt was journal-only, the ranked sweep reran Mastery. Four reruns (`ranked-1` to
`ranked-4`) each returned CORRECT in one submission call of 48 answers. Each rerun carried its own attempt ID.

Window arithmetic from the library notes: Mastery had `5N-2` sets of `N` records (48 x 10 here) and a
`5N+15` s clock, giving 65 s, which matches the `deadline` minus `started_at` in the start response.
The window was short enough that the solves were scripted fetch, decode, submit passes.

## Dead ends and gotchas

- None on the validator side: no rejected submissions in any mode.
- The Single Solve journal records that no existing library entry covered Base32 itself, so the
  recognition was manual. The Mastery trap was a mislabel in our own notes (single blobs versus chains),
  which did not change the solver.
- `challenges/day10/submission-attempts.md` shows 0 submissions for the Time Trial and Mastery rows. That
  table was not updated after those solves; the journals and `ranked-N/result.json` files record the
  submissions.

## Takeaways

- Title wordplay in this lane (here "32") names the new codec; verify it against the character set
  (`A-Z2-7=`, length a multiple of 8) before decoding.
- Warmup answers are always the slug on the last non-empty line, so check the whole sentence template to
  confirm a decode is right, not just the slug shape.
- For Mastery, treat the bundle as a numbered codec chain, where record N is Day N's codec, and map members
  to questions by name. A single reusable chain-decoder (`codec_chain.py`) handled this day and the others.

## Evidence

Base path: `challenges/day10/warmups/warmups-day-10/`

- `single-solve/journal.md`, `single-solve/start_response.json` (no result file saved; CORRECT recorded in the journal)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/ranked-1/result.json`, `time-trial/ranked-1/pass.log`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/ranked-1/` to `mastery/ranked-4/` (`result.json`, `accepted_plan.json`, `pass.log`)
- Mapper: `time-trial/artifacts/answer_map.py`, `mastery/artifacts/answer_map.py`; general solver `tools/ranked/solvers/warmups/solve.py`
- Library: codec-chain tool and the warmups lane playbook (day table, delivery shapes)
- `progress.md` rows for Day 10 `warmups-day-10` (all three modes marked Solved)
