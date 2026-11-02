# Web of Lies

| | |
|---|---|
| Day / Lane | 24 / Script Analysis |
| Challenge ID | `script-analysis-day-24-jsp-webshell-source` |
| Technique | inert-descriptor decoding of a new source family (JSP), bank reuse, declared-attack-wins |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Web of Lies" is the Script Analysis lane's JSP day. The deliverable is a ZIP of synthetic specimens in
many languages, and the answers (role, behavior graph, adapter, campaign id, MITRE ATT&CK technique,
...) are stored in an inert, encoded descriptor comment inside each file. The new family added 55
JSP-related question ids on top of Day 23's bank. The one real change was that JSP closes the descriptor
comment with `--%>` instead of `-->`; once the decoder tolerated that and the JSP `attack` answers were
read from the descriptor's own field, all three modes went CORRECT, Single Solve in 3 submission calls,
Time Trial in 3 and Mastery in 1.

## The challenge

Prompt: "This JSP source is shaped like a webshell: it reads requests and stages server-side work.
Recover the order of requests and stages, then separate the inert fixture behavior from the lines that
would be real execution primitives in the wild." Nothing in the specimens is executable; the files are
fixtures whose descriptor comments carry the ground truth.

- **Single Solve:** one `evidence.zip` (about 162 members across 25 file types: sh, py, js, php, vbs,
  lua, groovy, jsp, mof, ...), 1,516 answer ids with a named schema (`host-jsp-webshell-jsp-role`,
  `polyglot-jsp-benign-utility-attack`, `resource-scenario`, ...).
- **Time Trial:** one `evidence.zip`, 1,560 ids with a **different, hash-based schema**
  (`polyglot-jsp-588efb-url-host`, `jscript-bd1689-style`, ...), 50 s ranked window.
- **Mastery:** one outer ZIP holding eight nested `evidence-NN.zip` sets, 12,480 ids (8 x 1,560, ids
  prefixed `setN.`), 260 s ranked window.

Field kinds are `enum` (role, style, scenario), `ordered_tokens` (the `>`-joined behavior graph),
`integer` (stage count) and `exact_text` (campaign id, URL, path, adapter). All values are submitted as
strings.

The JSP-specific questions were 11 `host-jsp-webshell` ids and 44 `polyglot-jsp-*` ids (benign utility,
downloader, launcher, staged chain; each with role, steps, adapter, host profile, next language, and for
some an `attack` field).

## Solution

1. **Reuse the bank.** The Day 22/23 mappers were already validator-confirmed on the earlier families.
   A cumulative inventory of Day 24 against Day 23 showed exactly 55 new ids (11 `host-jsp`, 44
   `polyglot-jsp`) and nothing removed or changed, so only the JSP path needed work.
2. **The marker terminator.** Every specimen carries a `HUNTRESS_ANALYSIS_V1:<KIND>-<payload>` marker
   (kinds `HEX`, `B64`, `HB64`, `FRAG`) inside a comment. Earlier hosts closed that comment with `-->`.
   JSP uses a scriptlet comment, so the marker line ends in `--%>`. The extractor strips the closer
   before decoding:

   ```python
   marker = re.sub(r'\s*(?:--%>|-->)\s*$', '', m.group(1).strip())
   kind, sep, payload = marker.partition('-')
   if kind == 'HEX':   raw = bytes.fromhex(payload)
   elif kind == 'B64': raw = base64.b64decode(payload, validate=True)
   elif kind == 'HB64': raw = base64.b64decode(bytes.fromhex(payload), validate=True)
   ```

   The decoded text is a v3 descriptor holding `role`, `steps`, `adapter`, `profile`, `campaign`,
   `next_language`, and sometimes `attack`. `.jsp` was routed through the same marker decoder as `.php`.
3. **Answer from the descriptor.** The `attack` prompt prints a graph-precedence rule (terminal
   `dispatch_process` means the language-specific Command and Scripting Interpreter sub-technique,
   `acquire_remote` means ingress transfer, and so on). For JSP there is no language-specific
   sub-technique, so we used the descriptor's own `attack` value where it existed (`T1059` for the
   launcher, staged chain and host webshell; `T1105` for the downloader; `none` for the benign utility)
   and fell back to the graph rule, then bare `T1059`, only where no `attack` field was present.
   `jsp` was added to the language-to-technique table as the bare parent `T1059`.
4. **Submit in groups.** 1,516 ids were solved offline in about a second. The accepted submission order
   was: a 10-id `attack` probe group (accepted), the remaining 1,505 ids (accepted), then the
   last Batch `control` id alone, which returned CORRECT. That made 3 calls. The JSP `attack` group was
   the only part with no prior validator evidence, which is why it was isolated first.

## Time Trial and Mastery

Both switched to the hash-based id schema and to `LYR`-layered descriptor payloads (a layered encoding
the Day 23 TT/Mastery mapper already understood). The solver therefore dispatches on the schema of the
start response: named ids go to the Single Solve mapper unchanged, hash ids go to the Day 23 TT/Mastery
field mapping (`campaign-token`, `url-host`, `path-leaf`, `indicator`, and a merged `style` field). Two
points mattered:

- The first Time Trial attempt had no usable candidate: the Single Solve extractor choked on the new
  layered payloads (`marker present but no HEX/B64/HB64/FRAG payload found`) and on empty fields. After
  the dispatcher was added, Time Trial went CORRECT in 3 calls (a 3-id `style` group, an 89-id `attack`
  group, then the remaining 1,468).
- Mastery needed the nested ZIPs expanded (eight sets, each 1,560 ids, `setN.` prefixes) and then went
  CORRECT in one call with all 12,480 answers, at variant 0 of every alternate group.

Variant 0 was accepted in every mode; none of the generated alternate readings (graph-derived or `T1105`
for JSP `attack`, the `for_once` Batch control) was ever needed. Fresh timed attempts on 2026-10-24 went
CORRECT in 1 call each (50 s and 260 s windows, 4.98 s and 6.77 s locally). Three later reruns of each mode
(2026-10-31 and 2026-11-01) were also CORRECT, but some were sent as an `attack`/`style` probe plus a
remainder; those were speed reruns and are not the scoring result.

## Dead ends and gotchas

- **Marker closer.** JSP's `--%>` terminator was the real new trap; a decoder that only knows `-->` would
  leave the closer attached to the payload. (The library decoder's payload alphabet already stopped before
  `--%>`; only the legacy extractor needed the explicit terminator.)
- **Id schema changes between modes.** Single Solve kept named ids while Time Trial and Mastery used hash
  ids with `LYR` payloads, so a mapper validated on Single Solve was not enough. One schema detector per
  mode, not one mapper per challenge.
- **`attack` is the unvalidated field.** The prompt prints a graph rule, but the accepted JSP values were
  the descriptor's declared ones (for example `T1059` for a host webshell, where a pure graph reading
  gives a different technique). Declared field first, graph second.
- **Single Solve round accounting.** The lane log shows 1,515 ids accepted in the first two calls and the
  last id, a Batch `control`, submitted alone; it returned CORRECT. The log does not say whether that
  id was a deliberate holdout or an alternate-reading test.
- The first Time Trial attempt failing on the new schema was a harness-side miss; see the day overview.

## Takeaways

Technique names to search for: **inert analysis-marker descriptor** decoding, **cumulative bank
inventory** (diff a new day's ids against the previous day to find what is genuinely new), and
**declared descriptor field wins**. When a new language family appears, check three things before
submitting: the comment closer, whether the id schema changed per mode, and which answer fields have no
prior validator evidence (isolate those in a probe group so a wrong guess costs one call, not the batch).
Never classify by file name or by another mode's overrides.

## Evidence

Leaf: `challenges/day24/script-analysis/script-analysis-day-24-jsp-webshell-source/`

- `single-solve/journal.md` (solver notes and the library-promotion record)
- `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` and `legacy_solve.py` (schema dispatcher and marker decoder)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`
- Reruns: `time-trial/ranked-{2,3}/result.json`, `mastery/ranked-{2,3}/result.json`
- `challenges/day24/_run/lane-script-analysis-day-24-jsp-webshell-source.log` (submission rounds)
- `challenges/day24/_run/champion-script-analysis-day-24-jsp-webshell-source.json`
- `progress.md` (grep `^| 24 .*jsp-webshell-source`)
