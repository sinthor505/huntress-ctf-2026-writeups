# Lost in Translation

| | |
|---|---|
| Day / Lane | 8 / Script Analysis |
| Challenge ID | `script-analysis-day-08-cross-language-iocs` |
| Technique | Static reconstruction of encoded descriptors in 75 inert scripts across about ten languages, then a validator-specific rubric for role / ATT&CK / wrapper-style fields |
| Single Solve | Independent (CORRECT on request 60, all 615 fields) |
| Time Trial | Independent (untimed attempt CORRECT on request 18; later timed reruns CORRECT in one call each) |
| Mastery | Independent (untimed attempt CORRECT on request 5; later timed reruns CORRECT in one call each) |

## Summary

The challenge hands over an AES-encrypted ZIP of 75 small, inert scripts in PowerShell, JScript, Batch,
VBScript, Python, Bash, Node.js, PHP and script-host containers. Each script hides a behavior descriptor
behind its own encoding, and 615 questions ask for the decoded fields plus a few classifications. Decoding was
mechanical; the cost was the classification fields, where the validator's accepted values differed from the
literal reading of the prompt. The Single Solve took 60 submissions to learn those rules, and Time Trial and
Mastery were then solved with the same mapper in far fewer calls.

## The challenge

The prompt: "The same indicators show up in every language the campaign speaks, each with its own quoting,
escaping, and encoding accent. Normalize them to their exact form and find the one that doesn't match its
siblings." The journals never identify a single "odd one out" specimen. In practice the work was normalizing
every specimen's indicators and answering per-specimen fields.

Delivery for Single Solve: `evidence.zip` (76,687 bytes, AES-encrypted, 75 members, password `infected`) and 615
answer IDs of the form `<specimen>-<field>`. Counts by family prefix: Python about 126, Polyglot PHP 88, JScript
72, Batch 72, VBScript 72, Bash 70, Node.js 56, host containers (WSF/HTA/PHP webshell) 44, plus three small
PowerShell samples (`resource`, `collection`, `impact`; 15 fields). The field suffixes were `attack` (74), `role`
(71), `campaign` (70), `adapter` (67), `url`, `count`, `steps`, `style`, `obfuscation`, `control`, `path`,
`syntax`, `carrier` and others. Answer formats were enums, exact text, integers, ordered tokens (`a>b>c`) and
MITRE IDs.

## Solution

1. **Decrypt and read, never execute.** The ZIP was opened offline with `pyzipper`. Every specimen is a
   benign-looking script whose real content is an encoded descriptor plus an inert `--inspect`/`--plan` routine
   that would print it.
2. **Rebuild the descriptor per specimen.** Each language stores the same logical record (`steps`, `adapter`,
   `campaign`, URLs, paths, role/attack claims) in a different representation. The decoder
   (`artifacts/static_decode.py`) handled, among others:
   - JScript `packedText` in percent, colon, dash, hex and Base64 forms, and integer arrays;
   - VBScript `Const Material_*` / `Codec_*` with chunked Base64 and reversed hex;
   - Python `MATERIAL_` / `CODEC_` literals read with `ast.literal_eval`;
   - Bash fragments joined across adjacent quotes;
   - Batch descriptors rebuilt from numbered `set itemN=` fragments;
   - v3 `HUNTRESS_ANALYSIS_V1:` markers in PHP and host containers.

   Byte transforms (identity, reverse, single-byte / rolling / repeating XOR, add or subtract mod 256) apply in
   source order: a reversal inside the enclosing `recover_*` call and one inside the decoder do not commute.
   For Batch, a second route, the `rem HUNTRESS-CODEC:` / `HUNTRESS-MATERIAL:` comments, decoded to the same
   descriptor and served as a cross-check.
3. **Map fields.** Literal fields (campaign, adapter, URL, path, host id, note name) are copied straight from the
   descriptor. Structural fields come from the source: Batch expansion form and control wrapper, VBScript
   control wrapper, and JScript function / invocation / property style. `role` and `attack` come from the
   ordered `steps` graph using the prompt's rule text (for example `acquire_remote` then `write_file` is
   `downloader`; terminal `dispatch_process` after write/profile steps is `staged_chain`).
4. **Submit in families.** The API only says `incorrect` for a whole batch and lists `accepted_ids` for a batch
   that fully passes, so fields were submitted by family and then singly. 15 PowerShell fields and 54
   JScript/Batch/VBScript role+attack fields were accepted early; the rest were bisected in the rounds below.
5. **Final map.** After the corrections in the next section the full 615-field map was accepted (request 60,
   `CORRECT`; the Single Solve flag was `68b9e0e3cf98113fb0fead6fb175a2e8`).

## Time Trial and Mastery

- **Time Trial:** same 615-ID structure with a fresh archive (a different SHA-256). The Single Solve decoder and
  mapper were adapted, and a `final.py` pre-applied the confirmed overrides (host/PHP descriptor `attack`
  values, six Python launcher/relay/chain attacks as parent `T1059`, `batch-chain-attack`). Even so, 5 of 18
  requests came back `incorrect` while the rest were accepted by family; request 18 (the last 42 fields) was
  `CORRECT`. The attempt was untimed (`deadline: null`), so there was no clock pressure.
- **Mastery:** 3,075 fields, five sets of 615 with `setN.` ID prefixes, delivered as an outer bundle holding
  `evidence-01..05.zip`. The Time Trial mapper ran unchanged in each set directory. Requests 1 to 4 (one per
  set) were `accepted`; request 5 was `CORRECT`. Again untimed.
- **Later timed reruns:** a ranked sweep re-ran both modes against fresh fixtures with a consolidated solver
  (`tools/ranked/solvers/` in the workspace, built from the same rubric). Time Trial (50 s window) and
  Mastery (170 s window) were each `CORRECT` on the first call, in 1.9 to 2.6 s and 3.3 to 8.3 s total
  respectively (two Time Trial runs, three Mastery runs; the results are in the leaf's `ranked-N/`).
  The ranked fixtures were not harder, which is evidence that the rubric learned in the Single Solve
  transferred.

## Dead ends and gotchas

- **Submitting all 615 at once was a poor first move** (request 1, `incorrect`, no per-field feedback). Family
  by family, with confirmed families removed, is the only way to localize errors with this validator.
- **Batch `*-control` follows the validator, not the prompt.** The prompt says a one-item `FOR` loop that calls
  a subroutine is `for_once`. `local-dispatch.bat`, `impact-plan.cmd` and the launcher specimen were rejected
  as `for_once` and accepted as `call_subroutine` (as were the dropper, chain and relay). The outbound-transfer
  wrapper looked like `direct` (the `FOR` runs unconditionally after an `if defined ... set "cfg="` line) but
  `direct` was rejected and `if_guard` accepted. We could not determine why from the data; the accepted values
  were taken as authoritative. Fixtures regenerate, so `local-dispatch.bat` later had other shapes on other
  days: classify by structure, not by filename.
- **Descriptor `attack` beats the graph for v3 specimens.** The HTA specimen's `dispatch_language` step has no
  rule in the prompt; the graph reading `T1059.007` was rejected and the decoded descriptor's own `T1105` was
  accepted. For the WSF/PHP hosts the descriptor's attack was also not the graph answer: all three of
  `host-php-webshell-php-attack`, `host-wsf-jscript-jscript-attack` and `host-wsf-mixed-vbscript-attack` were
  accepted only as the bare parent `T1059` (the graph readings `none` for the PHP webshell, `T1140` and
  `T1059.005` for the mixed VBScript WSF were each rejected on their own; the JScript WSF's `T1105` was
  rejected only in a two-field test with the PHP `none`, so it is not isolated). The PHP polyglots needed per-specimen descriptor values (for example uploader `T1041`,
  launcher `T1059`, persistence `T1053`).
- **Python launcher/relay/chain attacks want parent `T1059`.** The language sub-technique `T1059.006` failed on
  six specimens (`current-pyc-launcher`, `embedded-runtime-relay`, `py2exe-console`, `py2exe-service`,
  `pyarmor-pyinstaller`, `zipapp-relay`). The 126-field Python batch was bisected by field type (descriptor
  fields, syntax/obfuscation/carrier, windows-chain and roles all accepted; only attacks failed) before the
  fix, which went `CORRECT`.
- **An empty request body counts.** Request 26 posted an empty file; the proxy answered `challenge_unavailable`
  and it still counted against the attempt cap. Build, validate and POST in one script (see the day overview).
- **Rejections carry no field list.** Several groups of 3 to 7 fields were `incorrect` and could only be
  resolved by single-field retries, which is where most of the 60 requests went.

## Takeaways

- Decode first, with one general tool covering every codec label, and verify it against any second encoding of
  the same data (the Batch comment route). The library now holds this as the Material/Codec decoder.
- Treat classification fields as validator-defined and test them in isolation: Batch control wrappers, v3
  `attack`, and Python chain attacks all differed from a literal reading of the prompt.
- When a bank repeats across modes, apply the confirmed override table before the first submit, and still send
  rubric-sensitive suffixes (`control`, `attack`, `role`) as separate groups. Time Trial still lost 5 requests
  by sending a pre-corrected map in coarse groups.
- Searchable names: Material/Codec descriptor decoding, behavior-graph rubric, Batch control-flow
  classification, descriptor `attack` precedence.

## Evidence

Leaf base: `challenges/day08/script-analysis/script-analysis-day-08-cross-language-iocs/`

- `single-solve/journal.md` - full 60-attempt ledger and the audit note; `single-solve/artifacts/static_decode.py`,
  `answer_map.py`, `submission-NN-{request,response}.json` (NN = 01..60), `start-response.json`
- `time-trial/journal.md` (reconstructed after the fact), `time-trial/artifacts/final.py`, `submission-01..18-*`
- `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md` (reconstructed after the fact), `mastery/artifacts/msub.py`, `m-01..05-{request,response}.json`
- `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`, `mastery/ranked-3/result.json`
- Rows in `progress.md` for Day 8 (all three modes "Solved"; Time Trial and Mastery rows were added by an
  audit from saved responses)
- Solver used for the ranked reruns: `tools/ranked/solvers/script-analysis-day-08-cross-language-iocs/solve.py`
