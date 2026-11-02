# Package Deal

| | |
|---|---|
| Day / Lane | 21 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-21-package-deal` |
| Technique | reading an Android package name from a binary `AndroidManifest.xml` (AXML) inside nested ZIPs; recap bundle of earlier metadata fields |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt is about mirrored Android packages, rebuilds versus clones and a "borrowed product
name", but the one new field Day 21 actually asked for was `apk.package_name`: the `package`
attribute on the root `<manifest>` element of the binary manifest. Single Solve was that one field.
Time Trial and Mastery wrapped it in a 26-field recap of the metadata fields from Days 1-20 (Mastery
was five such sets, 130 answers). All three modes went CORRECT on the first graded submission.

## The challenge

Prompt: "Mobile packages have been mirrored under friendly filenames. Recover Android package
identities, distinguish rebuilds from clones, and expose the borrowed product name."

- **Single Solve:** one answer id, `d21-apk-001-apk-package-name`, `exact_text` with `trim`
  normalization ("Determine apk.package_name for mobile-001.apk"). The artifact was a ZIP
  (`metadata-day-21-evidence.zip`) that contained `mobile-001.apk`.
- **Time Trial:** 26 answers against one `evidence.zip`: file facts for `roster.csv` and
  `evidence-note.txt` (size, MD5/SHA-1/SHA-256, 16-byte magic prefix, UTF-8 BOM flag), one field per
  earlier day's format (JPEG SOF size, PNG IHDR size, PDF header version, ZIP EOCD entry count and
  central-directory names, PE machine / entry RVA / original filename, .NET MVID, MSI ProductCode,
  LNK target, OOXML external relationship targets, e-mail Message-ID, WAV codec name, MP4 video
  dimensions, ELF `e_machine`, Mach-O CPU type set), the new APK package name, and one synthesis
  question ("SHA-256 each named file, submit the filenames as a JSON array in ascending digest
  order").
- **Mastery:** the same 26-question template over five evidence ZIPs (`evidence-01.zip` ...
  `evidence-05.zip`), ids prefixed `set1.` ... `set5.`, 130 answers in one submission.

The prompt's "rebuilds versus clones" and "borrowed product name" never became questions in any mode
we saw. Only `apk.package_name` was asked for the APK.

## Solution

1. **Recon.** The Single Solve download was hash-verified against the start response. The question
   named the file inside the bundle, so the work was: open the outer ZIP, find the APK, read its
   manifest.
2. **Why a plain tool did not apply.** Inside an APK, `AndroidManifest.xml` is compiled binary XML
   (chunked format: string pool, resource-id map, start/end element chunks), not text. The solver
   included a small stdlib-only reader for it. The relevant part is that it follows the chunk
   stream and builds the root element from `START_ELEMENT` chunks (type `0x102`), decoding each
   attribute's value by its type (string pool reference, integer, boolean):

   ```python
   elif ct == 0x102 and pool:
       _l, _c, _ns, name, astart, _asz, ac = struct.unpack_from("<IIIIHHH", d, pos + 8)
       node = {"tag": pool.get(name), "attrs": {}, "children": []}
       for i in range(ac):
           ans, an, araw, _vs, _r, vt, vd = struct.unpack_from("<IIIHBBI", d, pos + 16 + astart + i * 20)
           ...
           if vt == 3:      # string
               val = pool.get(vd)
   ```

   Attribute names fall back to the resource-id map when the pool name is empty, mapping known Android attribute ids (`versionCode`, `versionName`, `minSdkVersion`,
   `targetSdkVersion`, `label`, `name`).
3. **Answer.** `package` on the root `<manifest>` gave `com.huntress.arena.p08281778`. The solver
   cross-checked this against the raw UTF-16 string pool, where it was the only `com.` string, which
   confirmed the AXML parse had picked the right attribute.
4. **Submission.** One call with the single answer returned CORRECT.

## Time Trial and Mastery

Both modes were solved by dispatching on the answer-id suffix and reusing our existing metadata
reader for every field except the APK one. The conventions that mattered, all inherited from earlier
days:

- Hex answers use padded forms (PE machine 4 digits, entry RVA 8 digits, ELF `e_machine` 4 digits).
- The MSI ProductCode is submitted as stored (with braces, upper case); the .NET MVID is bare
  lowercase GUID text.
- OOXML external relationship targets in ascending lexical order, Message-ID with its angle brackets,
  video dimensions from the `tkhd` box.
- The synthesis question's answer is the named top-level files sorted by SHA-256.

Fixtures varied between attempts, which is why the solver had to be general:

- The Time Trial's `evidence-note.txt` questions changed between fixtures: the regular fixture asked
  for `roster-sha1` and `evidence-note-copy-sha256`, while the first ranked fixture asked for
  `evidence-note-size-bytes`, `evidence-note-sha1` and `evidence-note-sha256`.
- In Mastery the per-set questions varied as well (set 1 asked for the *renamed copy's* filename,
  set 2 for the first note's filename plus the copy's size), so ids had to be read from the
  start response, not assumed.

Results, from the saved `result.json` files: regular Time Trial CORRECT in 1 call, regular Mastery
CORRECT in 1 call (130 answers), then a ranked Time Trial (50 s window, 2.93 s local) and ranked
Mastery (170 s window, 3.21 s local), each CORRECT in 1 call, all on 2026-10-21. Later ranked reruns
on 2026-10-31 and 2026-11-01 were also CORRECT but took 5 calls for the Time Trial (and, on one
rerun, the Mastery): the base answers were accepted in one call and four small answer groups (GUIDs,
`id`, `order`, `hex`) were then submitted separately, each at its first variant. We could not
determine from the saved files why those reruns split their submissions when the original ones did
not. Those ranked runs were speed attempts, not the scoring basis.

## Dead ends and gotchas

- **The Time Trial was not the Single Solve with a bigger answer list.** It was a 26-field recap
  template, so a solver written for the one `apk.*` field could not answer it (the first Time Trial
  attempt was stopped by our own pre-submission lint and nothing was submitted). That is an
  automation issue; see the day overview.
- **The validator accepted the first submission on every mode.** There was no rejected spelling for
  the APK package name, the GUID forms or the other fields on the original day, so no validator quirk
  was recorded from this challenge itself. Alternate spellings were prepared for padded hex, GUID
  forms (MSI braced vs bare), Message-ID brackets and synthesis order, but none was needed on
  Oct 21.
- **A later review flagged two unsorted outputs.** The solvers emitted ZIP member names and Mach-O
  CPU types in discovery order rather than sorted; this passed only because the Day 21 and Day 22
  fixtures did not discriminate. Verify ordering rules against a fixture where they do.

## Takeaways

- Name the technique: **binary AXML parsing**. `package` is an attribute on the root element of the
  compiled manifest, readable without any Android tooling by walking the chunk stream. Cross-check
  it against the raw string pool as the solver did.
- Do not infer the question set from the prompt's flavor text: the story mentioned rebuilds, clones
  and labels, but only the package name was asked.
- For recap-template modes, key the solver on the answer-id suffix and the per-fixture start
  response; ids and file choices change between attempts.
- A later deep review promoted the APK reader into the shared metadata reader and extended it with
  the launcher activity, `debuggable`, `shared_user_id` and the v1 signer certificate (checked
  against OpenSSL), so a future APK field can come from the library rather than a copied solver.

## Evidence

Leaf base: `challenges/day21/metadata-forensics/metadata-forensics-day-21-package-deal/`.

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (stdlib AXML reader and nested-ZIP walk),
  `single-solve/artifacts/cand-rubric/solve_src.py`
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (plus `ranked-2` to `ranked-4`,
  `ranked-vm-2`)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` (plus `ranked-2`, `ranked-3`,
  `ranked-vm-1`)
- `challenges/day21/_run/lane-metadata-forensics-day-21-package-deal.log`
- Library entries: META-BUNDLE-TOOLKIT-B101 (bundle toolkit), META-ANSWER-CONV-B109 (answer
  conventions), META-PLAYBOOK-001 (lane playbook)
- `progress.md` (grep `^| 21 .*package-deal`)
