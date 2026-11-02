# Eighty-Five Alive

| | |
|---|---|
| Day / Lane | 23 / Warmups |
| Challenge ID | `warmups-day-23` |
| Technique | single-layer Adobe ASCII85 decode; Mastery as 113 independent 23-record chains |
| Single Solve | Journal-recorded |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The Day 23 warmup hides a one-line instruction ("submit this exact answer: `<slug>`") under a single
layer of ASCII85. The prompt riddles the codec: "The alphabet got bigger. Adobe might recognize the
neighbors" means a bigger alphabet than Base64 (Base85) in Adobe's variant (ASCII85, with `<~ ~>`
delimiters). The Single Solve and Time Trial were one decode each. Mastery turned that into 113
independent chains of 23 records, where the new 23rd codec was also ASCII85. All three modes went
CORRECT on the first submission.

## The challenge

Prompt: "The alphabet got bigger. Adobe might recognize the neighbors." One question per blob: decode
it completely and submit "the exact answer slug" from the final non-empty decoded line. The answer
format is a `pattern`: three lowercase words joined by hyphens, then a hyphen and a two-digit number.

Delivered artifacts:

- **Single Solve:** one 143-byte `blob.txt`, no deadline.
- **Time Trial:** one 141-byte `blob.txt` (different bytes and slug from Single Solve). The regular
  start returned no deadline; ranked starts later used a 30 s window.
- **Mastery:** one ZIP (69,815 bytes) of `blob-01.txt` ... `blob-113.txt`, 113 answer ids
  (`set1.message` ... `set113.message`), each pointing at its own member via the question's `artifact`
  field. The regular start had no deadline; the ranked reruns had a 130 s window.

## Solution

1. **Recon.** The blob begins `<~` and ends `~>`, the Adobe ASCII85 delimiters, which matches the
   riddle. We checked the artifact SHA-256 against the start response (it matched) before decoding.
2. **Decode.** The library's warmup codec-chain tool peeled one layer, `ascii85`, and produced the
   standard warmup sentence "Nice work decoding this warmup! To solve, submit this exact answer to the
   Game Master: crisp-starfish-quartz-70". The slug is the last non-empty line, and it matched the
   answer pattern. The decoder is essentially:

   ```python
   def d_ascii85(b: bytes) -> bytes:
       t = squeeze(b)
       if t.startswith("<~"):
           t = t[2:]
       if t.endswith("~>"):
           t = t[:-2]
       return base64.a85decode(t)
   ```

   Python's `a85decode` does not accept the `<~ ~>` wrapper unless told to, so the delimiters are
   stripped by hand first.
3. **Map and submit.** A tiny offline mapper (`answer_map.py`) asserted the hash, asserted exactly one
   decoded blob, and keyed the slug by the `answer_id` from the live start response (`message`). One
   submission, CORRECT.

## Time Trial and Mastery

**Time Trial.** Same codec on fresh bytes (slug `fancy-button-marble-84` on the regular attempt). The
mapper was reused unchanged, and the codec is fixed per challenge, not per variant. The regular attempt
went CORRECT on call 1. Two ranked reruns (a speed sweep, not part of the prize scoring) also went
CORRECT on call 1: 3.65 s and 1.49 s total inside a 30 s window.

**Mastery.** Each of the 113 members is its own `NN<TAB>payload` chain of 23 records (record `N` in
Day `N`'s chain, so Day 23 has 23 records, repeated across 113 sets). Record 23 was new to the decoder
table, which only went up to 22 then. The tool's repair path found it anyway: with exactly one unknown
record and a full-template check (the entire "Nice work decoding..." sentence, not just a crib), it
identified record 23 as `ascii85` for all 113 members, and every slug matched the answer pattern. Since
the same decode path had already been confirmed by Single Solve and Time Trial, we submitted all 113
answers in one call: CORRECT. Two ranked reruns reused the solver unchanged: 6.12 s and 1.73 s in a
130 s window, both CORRECT on call 1.

## Dead ends and gotchas

- There were no rejected submissions in any mode.
- The Mastery blobs are chains, not 113 single layers. A solver that hard-codes one layer, or a record
  count, would fail. The member names also zero-pad to two digits and then three digits from set 100,
  so a plain lexical sort places `blob-100` before `blob-11`. Map by each question's `artifact` field,
  not by sort order.
- A new record number missing from a decoder table is not necessarily a blocker: removing one unknown
  record and scoring the full template can recover it. We recorded `23: ascii85` in the library table
  afterwards.
- Single Solve: when the pipeline re-started this challenge later, the Game Master already read the
  team's CORRECT, so no new Single Solve was possible. Our own solve is the journal-recorded
  first-attempt CORRECT; the leaf has no saved graded `result.json` for it.

## Takeaways

Technique: **codec-chain peeling** with a strict decoder list, and for Mastery a chain-per-member
decode with automatic repair of the one unknown record. The prompt riddle names the codec every day
("bigger alphabet", "Adobe"), so read it first. What we would do faster: add the new record's codec to
the decoder's record table before the Mastery start, so the whole bundle goes through one table-driven
path.

## Evidence

Leaf base: `challenges/day23/warmups/warmups-day-23/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/decoded.json`, `single-solve/team_solved.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/ranked-1/result.json`,
  `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/candidate_answers.json`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `challenges/day23/_run/lane-warmups-day-23.log` (the later re-start read the team's Single Solve
  CORRECT; the regular Time Trial and Mastery were already CORRECT)
- `library/warmups/codec-chain/codec_chain.py`, `library/warmups/lane-evolution-and-playbook.md`
- `progress.md` (grep `^| 23 .*warmups-day-23`)
