# Entity Relations

| | |
|---|---|
| Day / Lane | 8 / Warmups |
| Challenge ID | `warmups-day-08` |
| Technique | HTML numeric character references (`&#NN;`) decoded with a standard HTML unescaper |
| Single Solve | Independent (own `CORRECT` recorded in `submission-01.json`; no `result.json`) |
| Time Trial | Independent (backed by the ranked-1 `result.json` and `accepted_plan.json`, both `CORRECT`; the original untimed attempt is recorded only in the journal) |
| Mastery | Independent (backed by four ranked-N `result.json`/`accepted_plan.json` files, all `CORRECT`; the original attempt is recorded only in the journal) |

## Summary

A warmup whose whole blob is a run of decimal HTML character references. Decoding it with
Python's `html.unescape` gives a one-line instruction ending in the answer slug. Time Trial is the
same blob type on fresh text; Mastery is a ZIP of 38 members, each an 8-record chain in which this
day's codec is record 8.

## The challenge

Prompt: "The browser understands. Do you?" The single question (`message`) asked for the exact
answer slug that the final non-empty decoded line tells you to submit. The answer format was a
pattern: three lowercase words joined by hyphens, a hyphen, and a two-digit number, normalized with
`trim`.

- Single Solve: `blob.txt`, 620 bytes of ASCII, one long line of `&#78;&#105;&#99;&#101;...`.
- Time Trial: one blob of the same kind, one answer ID.
- Mastery: one ZIP (`warmups-day-08-mastery.zip`) of 38 members `blob-01.txt` .. `blob-38.txt`;
  answer IDs `set1.message` .. `set38.message`, with each question's `artifact` field naming the
  member it is about.

## Solution

1. Recon. The blob was a single line of `&#<decimal>;` tokens only, with no Base64 alphabet, no
   percent escapes and no labels. That, plus the prompt's wording ("the browser understands"), made
   HTML character references the obvious reading.
2. Decode from the exact downloaded bytes with the standard library:
   `html.unescape(raw.decode("utf-8"))`. The result was one line of plain text, the usual warmup
   template: `Nice work decoding this warmup! To solve, submit this exact answer to the Game
   Master: <slug>`.
3. Take the token after the last `:` of the last non-empty line. It matched the pattern from the
   question, so no other reading was needed. Single Solve answer: `jovial-rocket-marble-82`,
   submitted once under ID `message` and graded `CORRECT`.

No existing local note covered this representation at the time (the nearby entries handled labeled
Base64/ROT13/hex, binary/decimal/octal, and percent encoding), so the transform was written up as a
new technique entry afterwards.

## Time Trial and Mastery

**Time Trial.** Same codec on a fresh blob. The solve went through the shared warmup codec-chain
tool's `html-references` layer and mapped its slug onto the single answer ID; `CORRECT` on the
first submission. The first untimed start reported `deadline: null`. In the later ranked pass the
window was 20 s; that pass finished in about 1.6 s end to end and was `CORRECT` in one call.

**Mastery.** The delivery was a ZIP, not a bare blob, and the members were not bare HTML-entity text
either. The journal's later review corrected an earlier assumption: each member is an 8-record
chain, lines of the form `NN<TAB>payload`, where record N uses the N-th codec of the
cumulative table (Base64, ROT13, hex, 8-bit binary, decimal, octal, percent, and finally HTML
references on this day). In the member inspected, the records are consecutive slices of
the warmup sentence ("Nice work dec", "oding this war", "mup! To solve", ...), including the slug,
which is split across the last records (record 7 ends mid-slug). So the slug is only readable
after every record is decoded and concatenated; decoding only the HTML record gives a
truncated tail. The codec-chain tool walks the ZIP, decodes each member by record number, and the
mapper looks up `question.artifact` -> member slug -> `question.id`. All 38 slugs matched the slug
pattern and the full sentence template; the submission was `CORRECT` on attempt 1.

The ranked pass later used 55 s Mastery windows (5N + 15 for N = 8, per the playbook table); four
ranked Mastery attempts were each `CORRECT` in one call, 2 to 5 s end to end, with fresh 38-member
fixtures each time.

## Dead ends and gotchas

- The Mastery journal first described the members as single-codec HTML blobs. They are 8-record
  chains; the mapper was right only because the tool detects both shapes. Do not special-case
  "Mastery = N copies of today's codec".
- Mastery is keyed on `question.artifact`, not on ZIP order. Map members by name.
- Nothing in this challenge was rejected by the validator; the first submission in every mode was
  accepted. The one spot where a decoder choice could matter (malformed references, where lenient
  and strict unescapers differ) was not exercised: the data held only well-formed decimal
  references.

## Takeaways

- Technique name: HTML character reference (numeric entity) decoding; `html.unescape` is enough.
  Preserve line structure and read only the last non-empty line.
- Warmup answers in this lane are always the last-line slug, so the transform is the whole puzzle.
  Recognising the representation from the bytes is the work; the prompt's riddle usually names it.
- For multi-record Mastery members, decode all records and join them before applying the slug rule.
- Faster next time: drive everything from one codec-chain tool that handles ZIP members and record
  numbers, so a day's new codec is one table entry.

## Evidence

Leaf base path: `challenges/day08/warmups/warmups-day-08/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/submission-01.json`
  (graded `CORRECT` response)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`,
  `time-trial/ranked-1/{result.json,accepted_plan.json,pass.log}`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/ranked-1/` .. `ranked-4/{result.json,accepted_plan.json,pass.log}`
- Library: the HTML character references entry (`WARMUP-HTML-ENTITIES-001`) and the warmups
  lane playbook (`WARMUP-PLAYBOOK-001`, day table and record codec map).
- Progress rows: `progress.md`, rows for Day 8 `warmups-day-08` (Single Solve 2026-10-20, Time Trial
  and Mastery 2026-10-23).
