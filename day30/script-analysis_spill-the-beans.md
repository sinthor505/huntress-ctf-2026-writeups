# Spill the Beans

| | |
|---|---|
| Day / Lane | 30 / Script Analysis |
| Challenge ID | `script-analysis-day-30-beanshell-jsp-graph` |
| Technique | descriptor-driven specimen analysis; one new language family (BeanShell) plus a JSP-hosted BeanShell page |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 30's Script Analysis challenge added one new language to the "thirty-language puzzle": BeanShell
(`.bsh`), plus a second kind of JSP host that wraps a `.bsh` specimen. No new solver code was needed.
The existing solver already read each specimen's embedded descriptor and answered from the declared
fields, and that held for BeanShell too. A teammate had already scored the Single Solve before our first
submission, so we have no graded result for it. Time Trial (7 submission calls) and Mastery (1 call) were
both graded CORRECT for us on fresh `retry` attempts.

## The challenge

Prompt: "BeanShell source sits inside an inert JSP host container, and it's the last piece of a
thirty-language puzzle. Analyze both, then rebuild the complete execution, host, and evidence graph,
and find the edge that holds everything together."

Each mode delivered a ZIP of inert specimens (never executed) and a large question list:

| Mode | Questions | Delivery |
|---|---|---|
| Single Solve | 1,802 | one `evidence.zip`, 188 members, named ids such as `resource-scenario`, `host-jsp-webshell-jsp-role` |
| Time Trial | 1,898 | one `evidence.zip`, hash-style ids such as `powershell-84522a-url` |
| Mastery | 26,572 | 14 sets of 1,898 questions, ids prefixed `setN.` |

The questions ask for role, ATT&CK technique, adapter, stage count, URLs, tokens, paths and similar
fields of each specimen, in `enum`, `mitre_attack_id` and `exact_text` formats. Most Time Trial ids
(1,443 of 1,898) are `polyglot-*`: a script in one of about 26 languages carrying an encoded descriptor
marker.

## Solution

1. **Diff the question bank against the previous day.** Against Day 29's start responses, Single Solve
   was a pure superset: 55 ids added, none removed or changed (44 `polyglot-beanshell` plus 11
   `host-jsp`, which together make one new JSP-hosted member, `host-jsp-beanshell-beanshell.bsh`). The
   Time Trial bank was Day 29's plus 65 `polyglot-beanshell` ids. So the whole day's novelty was one
   language.
2. **Find the descriptor in the new file shapes.** Each specimen carries a `HUNTRESS_ANALYSIS_V1:`
   marker holding a structured descriptor. `.bsh` files put it in a `//` line comment. The JSP-hosted
   `.bsh` is a JSP page (`<%@ page ... import="bsh.Interpreter" %>`) whose marker sits inside a Java
   string literal in a `<%! ... %>` declaration, with no `--%>` terminator. A decoder keyed on file
   extension would look for the wrong comment syntax. The library's marker decoder already handled both
   shapes without changes (all 188 Single Solve members decoded with zero errors).
3. **Answer from the declared descriptor.** The rule from the earlier new-language days (Elixir, Erlang,
   Dockerfile, Makefile) held again: `role`, `attack`, `adapter`, `steps` and the other fields are
   answered exactly as the descriptor declares them, not as a graph reading of the code would suggest.
   BeanShell has no ATT&CK sub-technique of its own, so the declared value (the bare parent `T1059`
   where the descriptor says so) is what gets submitted. The solver's `NEW_FAMILIES` tuple, which queues
   alternate readings for unfamiliar families, did not even list BeanShell.
4. **Dispatch on the id schema.** The solver (the schema-dispatching adapter from Days 24 and 29) has
   two paths, the named-id Single Solve path and the hash-id Time Trial/Mastery path, chosen from the
   start response rather than from the day.
5. **Submit by group.** All unambiguous fields went in one call (1,885 ids). Each small group whose answer
   could plausibly have an alternate reading was then probed at its first variant: the per-language
   `attack` ids for JavaScript, JScript, VBScript and AppleScript polyglots, a Batch control group, and
   three Node.js staged-chain `attack` ids.

## Time Trial and Mastery

Single Solve was scored by a teammate before our first submission, so ours returned `already_solved`
(nothing graded for us) and we stopped submitting there. The leaf does not record a retry or practice attempt on
this exact id, only that the Single Solve was team-scored, so we did not get a graded Single Solve. We then issued a
fresh `retry` for each of Time Trial and Mastery, which the Game Master grades on their own merits.

- **Time Trial** (1,898 ids): CORRECT in 7 calls. Six group probes were each `accepted` and the final
  `v3-attack-polyglot-applescript` group (4 ids) returned CORRECT. Every group was accepted at variant 0,
  so nothing needed fixing, including the three Node.js staged-chain ids (two `T1059.007` chains and one
  `T1105` stager ending in `dispatch_memory`).
- **Mastery** (14 sets x 1,898 = 26,572 ids): CORRECT in one 26,572-answer call at variant 0, with the
  solver run once per set (about a second each).

Later ranked attempts (50 s Time Trial window, 580 s Mastery window) also went CORRECT, in 7 calls and
1 call. Under the Day 30 prize rules ranked runs did not count, so the earlier untimed `retry` results
are the ones that matter. A third ranked Time Trial pass on 1 November stopped after six calls at our own
20-submission ledger cap (`cap_reached`); it affected nothing (see the day overview).

## Dead ends and gotchas

- **The Single Solve was never graded for us.** Its named-id map, including the JSP-hosted BeanShell
  member, is unvalidated. The declared-field rule was confirmed on Time Trial and Mastery, but those use
  hash ids, so the named-id `host-jsp-beanshell-beanshell.bsh` answers were never tested by the validator.
- **Node.js staged-chain `attack` follows the question-id schema, not the day.** Tabulating every
  accepted plan across days showed the Node.js `acquire_remote ... dispatch_process` chain takes
  `T1059.007` on hash-id plans (Time Trial/Mastery on Days 24-26 and 28-30; 12 of 12 rows on Day 30) and
  `T1105` on named-id plans (Days 11-22, Single Solve of 24-26, and Day 27's Time Trial/Mastery). A
  per-day rule did not fit. Hash-id Bash chains never occurred in those days, so that case is
  unobserved.
- **No format surprises.** The usual `exact_text`/`trim` and enum answers were accepted as declared.

## Takeaways

- When a day adds a language, diff the question bank against the previous day first to see exactly what
  is new (`bank_inventory`-style), then check how the descriptor marker is carried in the new file shapes
  (line comment, string literal inside a host page) before writing any decoder.
- **Declared descriptor wins** has now held for five consecutive new-language days. Diffing the
  solver's output against the accepted plans per language (`declared_vs_plan`) showed zero mismatches
  across all 26 languages: beanshell 50/50 (Time Trial) and 700/700 (Mastery), jsp 50/50 and 700/700,
  the other 24 languages 1,010/1,010 and 14,140/14,140.
- Dispatch on the start response's id schema (named vs hash-id), not on the day number.
- A Single Solve a teammate already scored cannot be validated by us; the independent Time Trial and
  Mastery results are what show the workflow works.

## Evidence

Leaf base: `challenges/day30/script-analysis/script-analysis-day-30-beanshell-jsp-graph/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-solo/solve.py` (schema-dispatching solver; `legacy_solve.py` is the named-id path)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`, `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`, `mastery/ranked-1/result.json`
- `challenges/day30/_run/lane-script-analysis-day-30-beanshell-jsp-graph.log`
- `challenges/day30/_run/librarian/script-analysis-day-30-beanshell-jsp-graph.md`
- `library/script-analysis/descriptor-schema-and-rubrics.md` (section 4.1g)
- `progress.md` (grep `^| 30 .*beanshell`)
