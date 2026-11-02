# Actions Speak Louder

| | |
|---|---|
| Day / Lane | 21 / Script Analysis |
| Challenge ID | `script-analysis-day-21-github-actions-persistence` |
| Technique | static descriptor decoding of a new polyglot family (GitHub Actions `.yml`), reusing a validated rubric |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt talks about workflow triggers, runner trust and persistence, but every question is a
field of a machine-readable descriptor hidden inside each specimen, not a judgment about
`pull_request_target` or self-hosted runners. Day 21 added one new family to the Day 20 bank
(44 `polyglot-github_actions-*` answer IDs in four roles). The marker decoder and answer mapper
from Day 20 worked unchanged, and the declared descriptor fields were submitted as-is. All three
modes went CORRECT; Single Solve on 1 call, regular Time Trial and Mastery on 1 call each.

## The challenge

Prompt: "A GitHub Actions workflow decides who can trigger it and which runner inherits the trust.
Identify each trigger and trust boundary, follow the static handoff into the next language, and
find the job that keeps coming back."

- **Delivery:** one AES-encrypted `evidence.zip` (password `infected`, so `pyzipper`, not stdlib
  `zipfile`), SHA-256 matching the start response.
- **Single Solve:** 1,373 answer IDs. By the ID prefixes the families were: 660 `polyglot-*`
  (of which 44 were the new GitHub Actions family), 168 `bash-*`, 132 `host-*`, 126 `python-*`,
  72 each of Batch, JScript and VBScript, 56 Node.js, and 15 PowerShell sample answers
  (`resource-*`, `collection-*`, `impact-*`).
- **Time Trial:** the same 1,373-ID schema against a fresh artifact. **Mastery:** 6,865 IDs, which
  is five sets of 1,373.
- Answer kinds were mostly `enum`, `mitre_attack_id`, `exact_text`, `integer` and `ordered_tokens`
  (steps joined with `>`), all with `trim` normalization.

## Solution

1. **Recon.** The 44 new IDs follow the earlier v3 "polyglot" pattern: for each of four roles
   (`benign-utility`, `downloader`, `launcher`, `staged-chain`) there are about 11 fields: `adapter`,
   `attack`, `campaign`, `host-profile`, `next-language`, `obfuscation`, `role`, `stage-count`,
   `stage-profile`, `steps`, plus `url` or `file-path` depending on the role. The interesting one is
   `next-language`, the "static handoff into the next language" from the prompt: it names where the
   workflow's embedded payload goes next (`powershell`, `jscript`, `batch`, `vbscript` in the
   Single Solve map).
2. **Marker.** Each workflow `.yml` carries the usual
   `# HUNTRESS_ANALYSIS_V1:HEX-...` (or `HB64-...`) comment, and the same JSON is repeated inside
   a `HUNTRESS_DESCRIPTOR` environment string. The Day 20 marker decoder read it without changes,
   so the descriptor, not the workflow's YAML semantics, supplied the answers.
3. **Mapping.** The solver is the Day 20 decode and answer map (already validator-confirmed on the
   previous day) plus the library's Batch-control signature classifier. The one rule that mattered
   for the new family is that `role`, `attack` and `adapter` for `polyglot-*` and `host-*` IDs are
   taken from the descriptor's own declared value, not derived from the step graph:

   ```python
   elif suffix == 'attack' and (aid.startswith('polyglot-') or aid.startswith('host-')) \
        and f.get('attack') is not None:
       value = f['attack']
   ```

   For the GitHub Actions family that produced `none` for benign-utility, `T1105` for downloader
   and the bare parent `T1059` for launcher and staged-chain. Nothing in a graph reading would
   have predicted a `none`, or `T1059` rather than a sub-technique, which is why the declared
   field is the safe source.
4. **Submission.** The solver reached 1,373/1,373 answers, lint-clean (every enum value in its
   accepted set, every ATT&CK ID well-formed, no empty or untrimmed strings), in about a second.
   The adapter's own notes flagged two open doubts: no validator evidence yet for the GitHub Actions
   family, and the VBScript controls reusing Day 20's classifier unchanged. Both were resolved by
   the first submission: one call with all 1,373 answers returned CORRECT.

## Time Trial and Mastery

The regular (untimed) attempts and the first ranked attempts all went CORRECT on one call each:
Time Trial 6.3 s total (regular) and 5.3 s (ranked, 90 s window); Mastery 8.1 s for both (ranked
window 170 s). Mastery was the same solver run once per set (five sets) and submitted as one
6,865-answer call. The mappings were not re-derived for these modes; the Single Solve map generalized
directly.

Four later ranked Time Trial reruns and two Mastery reruns (bragging-rights only) also went CORRECT,
with fresh fixtures each time. The Time Trial reruns took 2 to 5 submission calls because the
answer was submitted per group (the `attack` fields, then the rest); in every case the first
(declared-descriptor) variant of every group was the accepted one, so none of the prepared
alternate readings (e.g. `T1105` for every GitHub Actions role, or `T1059` for every role) were
needed. Why those reruns split the submission rather than sending it whole is a harness choice, see
the day overview.

## Dead ends and gotchas

- **No rejected answers were recorded for this challenge.** The usual trap on these days is the
  `attack` field (graph reading vs. declared value) and Batch `*-control` classification; here the
  declared value won for every GitHub Actions `attack`, and the Batch controls submitted by structural
  signature (one group had a `for_once` alternate prepared) needed no alternate.
- **Prompt text is flavor.** Nothing asks for trigger events, `permissions:` scopes, or runner
  labels; looking for them in the YAML would have cost time for no answers.
- **Extractor fragility on fresh fixtures.** One ranked rerun's adapter solver failed lint on a JScript
  downloader's fields (empty values) on a fresh Time Trial fixture; the rubric solver, which decodes
  through the library, still produced all 1,373 answers and was the one submitted. The cause was
  not diagnosed in the files; treat it as a reminder to run more than one independent extractor.

## Takeaways

- For a **new polyglot family** in this lane, check the descriptor first: the answers are the
  declared fields (`role`, `attack`, `adapter`, `steps`, `next-language`, ...), and the descriptor's
  own `attack` value wins even when a graph reading would give a sub-technique or a different
  technique. The decode-and-map toolchain carried over from the previous day without edits, so
  the day's work was verification, not new reverse engineering.
- Decode all 1,373 answers offline, lint against the start response's own `answer_format` (enum
  sets, ATT&CK syntax, trim), and submit once; split into groups only after a rejection.
- Search terms: descriptor marker `HUNTRESS_ANALYSIS_V1`, "declared descriptor wins", Batch-control
  structural signature, polyglot family bank.

## Evidence

Leaf base: `challenges/day21/script-analysis/script-analysis-day-21-github-actions-persistence/`

- `single-solve/journal.md` (library consultation, doubts, round 0 CORRECT in 1 call),
  `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (the decode and mapping logic quoted above),
  `single-solve/artifacts/cand-rubric/solve.py`
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (and `ranked-2` to `ranked-4`)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` (and `ranked-2`, `ranked-3`)
- `challenges/day21/_run/lane-script-analysis-day-21-github-actions-persistence.log`
- Library entries: SCRIPT-TECHNIQUE-C105 (family playbook), SCRIPT-TECHNIQUE-C101 (marker
  decoding), SCRIPT-TECHNIQUE-004 (Batch control signatures), SCRIPT-REFERENCE-C102
  (descriptor `attack` field)
- `progress.md` (grep `^| 21 .*github-actions-persistence`)
- Note: no librarian note exists for this challenge under `challenges/day21/_run/`.
