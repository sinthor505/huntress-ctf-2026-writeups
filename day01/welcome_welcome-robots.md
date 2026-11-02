# Welcome Robots

| | |
|---|---|
| Day / Lane | 1 / Welcome |
| Challenge ID | `welcome-robots` |
| Technique | Read the start response (orientation challenge, no analysis) |
| Single Solve | Solved (journal-recorded): our own submission read CORRECT on the first call per the journal; no machine-readable result file was kept in the leaf |
| Time Trial | Absent (Single Solve only) |
| Mastery | Absent (Single Solve only) |

## Summary

Welcome Robots is an orientation challenge. It checks that the player's AI teammate can talk to the
Game Master, find the challenge and hand back the value the platform issues. The answer was already in
the start response, and we submitted it unchanged.

## The challenge

The prompt asks the teammate to "identify the Welcome Robots challenge and provide the answer for the
CTF platform". The accompanying guidance describes the general loop: list challenges, get your
artifact or target from the Game Master, solve it from the real bytes, submit through the Game
Master.

The start response contained no artifact, no live target and no question list. It had:

- `mode: single_solve`, `deadline: null`
- `result: "issued"`
- an `issued_flag` field holding the value
- `answer_ids: ["flag"]`, so there is one answer, a string

## Solution

1. Listed the day's challenges and started `welcome-robots`. We saved the response as
   `start_response.json`.
2. Noticed that `result` was `issued` and that the response carried `issued_flag`. This is the whole
   puzzle: the value is delivered with the start response, so no triage or extractor was needed.
3. Wrote that value under the single answer ID, `flag`, in an answers file. The submitted value was
   `91ae397590224824098f0dced0959d28`.
4. Ran the submission as a dry run first (attempt 1 of the 20 allowed), then live. The Game Master
   returned `CORRECT` on the first call.

## Dead ends and gotchas

None for this challenge. The sibling Welcome challenge, Read the Rules, is where the trap is: its flag
exists only in the rendered DOM of a client-rendered page, and it was accepted only in its
`flag{...}`-wrapped form. That has its own page.

## Takeaways

- For a challenge with no artifact and no target, read the saved start response before writing any
  tooling. Orientation challenges can hand over the answer in a field such as `issued_flag`.
- Submit the issued value exactly as given, as a JSON string under the advertised answer ID.

## Evidence

Leaf: `challenges/day01/welcome/welcome-robots/single-solve/`

- `journal.md`: recon, submission (1 call, CORRECT) and the library note.
- `start_response.json`: the `issued` start response.
- `artifacts/answers.json`: the submitted answer file.
- `progress.md` row (Day 1, `welcome-robots`): Solved, Game Master CORRECT on 2026-10-23. The same
  row says that an earlier 2026-10-18 local mark came before any accepted submission.
- Provenance note: the leaf has no `result.json`, `accepted_plan.json` or `manual_solve.json`, so the
  CORRECT is recorded in the journal and `progress.md` row only.
