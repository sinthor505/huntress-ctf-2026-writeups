# Monkey Business

| | |
|---|---|
| Day / Lane | 27 / Warmups |
| Challenge ID | `warmups-day-27` |
| Technique | Ook! (a Brainfuck dialect) decoded through a Brainfuck VM; slug is the tail of the decoded line |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The Day 27 warmup hid its message in Ook! source: a text file of `Ook.`, `Ook?` and `Ook!` tokens that
translate pair by pair into Brainfuck operations. Every real blob used only `+`, `-` and `.`, so it was
the previous day's Brainfuck "accumulator" stream written in a different spelling. Running it through a
Brainfuck VM printed one line of text, and the answer was the slug after the last colon. A teammate
scored the Single Solve seconds after unlock, so ours returned `already_solved`; Time Trial and Mastery
each went CORRECT on the first call after a fresh `retry`.

## The challenge

Prompt: "Ook. Ook? Ook! Someone seems chatty." The question was the standard warmup one: decode the
assigned blob completely and submit the slug named by the final non-empty decoded line. Format: three
lowercase words joined by hyphens, a hyphen, then a two-digit number (`pattern`, `trim`).

- Single Solve: one `blob.txt`, one answer (`message`).
- Time Trial: the same shape on a fresh blob, 30 s window when run as a ranked attempt.
- Mastery: a ZIP of 133 blobs (`blob-NN.txt`), 133 answers (`setN.message`), 150 s window when ranked.
  Each blob is 27 numbered records (`NN<TAB>payload`). The record number selects the codec, and records
  1-26 reuse the older warmup codecs. Record 27 is the new Ook layer, applied to a fresh tape.

## Solution

1. **Recognise the encoding.** The prompt names it. The Single Solve blob held 6,786 `Ook` tokens, which
   makes 3,393 operations once read in pairs.
2. **Map the tokens.** The standard Ook! table, applied two tokens at a time:

   ```python
   OOK_MAP = {b".?": ">", b"?.": "<", b"..": "+", b"!!": "-",
              b"!.": ".", b".!": ",", b"!?": "[", b"?!": "]"}
   ```

   The converter is strict. It rejects anything that is not Ook tokens and whitespace, an odd token
   count, or the non-instruction pair `Ook? Ook?`. That strictness lets the layer sit in a generic
   codec peeler without claiming blobs that belong to other codecs. It also accepts the short form
   (bare `. ? !`) in case a later variant used it.
3. **Run it.** A small Brainfuck VM with 8-bit wrapping cells and a 30,000-cell tape. The real blobs
   contained no loops or pointer moves. They were the Day 26 stream, `+` and `-` building a running
   value and `.` emitting a byte, so the VM was more general than needed.
4. **Take the slug.** The decoded text ended in a template line; the answer is the literal text after
   the last `:` on the last non-empty line. For this day's Time Trial that read
   `brave-beacon-flint-86`. The Single Solve candidate was `hearty-castle-fern-71`, but it was never
   graded for us (see below).

The solver was written on top of the existing warmup codec-chain library and registered the Ook layer at
import time instead of editing the library, then the layer was promoted into the library afterwards.

## Time Trial and Mastery

- **Time Trial** was the same blob type. The solver decoded it in about 0.35-0.45 s. It was CORRECT on
  the first call in both the regular (untimed) attempt and the ranked attempt (30 s window, 2.88 s
  total locally).
- **Mastery** needed the per-record dispatch. Records 1-26 go through the library's record table, and
  record 27 is the Ook layer. Each record starts a fresh Brainfuck machine. The alternative, carrying
  the tape across records, was kept as a fallback variant but was never needed. All 133 sets decoded with
  zero repairs, and the whole map went CORRECT in a single call, regular and ranked. The ranked runs
  took 12.9 s and 16.9 s total locally against the 150 s window. A third ranked run on the last day was
  also CORRECT.
- Re-checking the final library decoder against the accepted plans reproduced TT 1/1 and Mastery 133/133
  for both the regular and ranked attempts, with `ook` as the last codec in all 133 sets.

## Dead ends and gotchas

- **Nothing was rejected.** No answer was ever marked `incorrect` in any mode; the decode was a single
  unambiguous template line.
- **Record 27 was missing from the library's record map.** The solver supplied it at import time. This
  matters because a Mastery blob that contains a record number the map does not know decodes to garbage;
  the same trap cost Day 29 a whole Mastery submission.
- **Loops are untested.** The real blobs used only `+ - .`. Ook programs with loops, pointer moves and
  `,` input were exercised only with synthetic inputs, so the VM's handling of those is unconfirmed
  against the validator. The `,` end-of-input convention (read zero versus leave the cell unchanged) was
  prepared as an alternate answer, but no real blob used `,`.
- **The Single Solve returned `already_solved`.** That response means the team's existing attempt was
  scored, not that our answer was wrong, and `retry` returned 405 and `practice` 409 for it. Our
  automation misread this as a rejection and burned three fix rounds on it (one submission each) and an
  escalation to the stronger model; see the Day 27 overview.

## Takeaways

- **Esolang warmups.** The Days 26-27 pair were the same Brainfuck program in two spellings. When the
  prompt names an esolang, check the library codec first. A strict token filter plus a small VM covers the
  family.
- **Fresh tape per record** was the right default for numbered-record Mastery bundles, as it was for
  Day 26's record.
- **Ook! table** is the quickest check for a blob made only of `Ook.`, `Ook?` and `Ook!` tokens.
- A team-solved Single Solve does not block Time Trial or Mastery. A fresh `retry` on each issues new
  fixtures that grade independently.

## Evidence

Base: `challenges/day27/warmups/warmups-day-27/`

- `single-solve/journal.md` (library consultation, the `already_solved` diagnosis, promotion notes)
- `single-solve/start_response.json` (prompt, question, format; a team-solved status echo)
- `single-solve/artifacts/cand-rubric/solve.py` and `single-solve/artifacts/cand-rubric/out.json` (the
  Ook layer, VM and ungraded Single Solve candidate); `single-solve/artifacts/cand-adapter/solve.py` is
  the second implementation
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`, `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`,
  `mastery/ranked-3/result.json` (plus each `accepted_plan.json`)
- `challenges/day27/_run/librarian/warmups-day-27.md`, `challenges/day27/_run/lane-warmups-day-27.log`
- `library/warmups/codec-chain/codec_chain.py` (`ook` layer, `DEFAULT_RECORD_MAP[27]`)
- `progress.md` (grep `^| 27 .*warmups-day-27`)

For the Single Solve we verified only that our offline decode is self-consistent. `progress.md` records
it as Team solve (not ours); the graded solve was a teammate's and our map was never validated in that
mode. It was validated by the Time Trial and Mastery decodes, which use the same decoder.
