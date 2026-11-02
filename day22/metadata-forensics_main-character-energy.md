# Main Character Energy

| | |
|---|---|
| Day / Lane | 22 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-22-main-character-energy` |
| Technique | JAR manifest vs. class-file launcher check (`jar.main_class`); large recap bundle of earlier-day metadata fields |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 22's new field was `jar.main_class`: read a JAR and say where execution begins. The Single Solve
was exactly that one field and went CORRECT on the first submission. Time Trial and Mastery re-asked
the whole Day 1-22 metadata template (27 fields, and 6 sets of 27 = 162 fields) plus the JAR field.
Mastery looked stuck for a while, but the cause was one real extractor bug (a hard-coded file name)
and a bug in our submission bisection, not a wrong reading of any field. All three modes ended CORRECT.

## The challenge

Prompt: "Java archives contain libraries, applications, and nested dependencies whose manifests
disagree with their launchers. Determine what is runnable and where execution begins."

- **Single Solve:** one answer, `d22-jar-001-jar-main-class` ("Determine jar.main_class for
  application-001.jar"), `exact_text`, `trim`. The artifact was a ZIP (`metadata-day-22-evidence.zip`)
  with the JAR inside; the answer format allows the literal `none` when the field is absent.
- **Time Trial:** one `evidence.zip`, 27 questions. They cover the shared template (roster and
  evidence-note filename/size/hash/magic-prefix/BOM fields) and one field for each earlier day's file
  type: JPEG SOF dimensions, PNG IHDR, PDF header version, ZIP EOCD entries and central-directory names,
  PE machine/entry RVA/original filename, .NET MVID, MSI ProductCode, LNK target, OOXML external
  relationships, email Message-ID, audio codec, video dimensions, ELF `e_machine`, Mach-O CPU type set,
  APK package name, the new JAR main class, and a "synthesis" question (order five named files by
  SHA-256 digest, as a compact JSON array).
- **Mastery:** six nested `evidence-NN.zip` sets, 162 answers (`setN.<id>`), same 27 field shapes per set.

## Solution

1. **Open the nested archives.** Outer ZIP, then the JAR inside it. The solver walked the ZIP itself and
   fell back to a raw inflate when `zipfile` choked, so a slightly odd container did not stop it.
2. **Read the manifest, then check it against the launcher.** The prompt's hint that "manifests disagree
   with their launchers" suggested not trusting `Main-Class` blindly. The solver collected candidates in
   this order: manifest `Main-Class` (continuation lines joined), then any class file that has a
   `public static void main(String[])`, then `Start-Class` / `Launcher-Agent-Class`:

   ```python
   cands = []
   if mf.get("Main-Class"):
       cands.append(mf["Main-Class"])
   for name, has_main in classes:          # class_info(): this_class + public static main
       if has_main and name not in cands:
           cands.append(name)
   for k in ("Start-Class", "Launcher-Agent-Class"):
       if mf.get(k) and mf[k] not in cands:
           cands.append(mf[k])
   return {"main_class": cands[0] if cands else "none", "main_alts": cands[1:]}
   ```

3. **Answer.** For this specimen the manifest `Main-Class`, the class file's own `this_class` and its
   `public static main` all agreed: `arena.evidence.ArtifactMain` (dotted form). That was the Single
   Solve answer and it was accepted in one call. The slash form and `none` were kept only as alternates
   and were not needed. (A later review across all nine real JARs we saw found `Main-Class` always
   equalled the only class with a `main`; it never disagreed with the class files on a real specimen.)

## Time Trial and Mastery

**Time Trial.** The first regular attempt was linted out because the solver still answered only the
single JAR id. It was extended to the full template (27 ids, strings only; the MSI ProductCode and a
few hex fields carried variant groups) and went CORRECT in 1 call. Ranked Time Trial attempts later
returned CORRECT in 6 calls each, because the grouped fields (`class`, `id`, `order`, `guid`, `hex`) are
submitted group by group and the last group to be accepted ends the run.

**Mastery.** Our first regular attempt ran out of its 20 submission calls at 120 of 162 accepted. Of
the six families that stayed unaccepted (PE entry RVA, PE machine, ELF `e_machine`, .NET MVID, MSI
ProductCode, email Message-ID), none was actually wrong. Two separate problems were in play:

- **A real extractor bug.** `roster-magic-prefix-hex-16` is new to Day 22 and asks for the first 16
  bytes of `roster.csv`. The solver hard-coded `evidence-note.txt`'s prefix for every
  `*-magic-prefix-hex-16` field, so it answered with `Case FA695E7CC3F...` as hex. Resolving the file
  from the question ID (the same way the sibling filename/size/hash fields already did) gave
  `686f73742c636173650a57532d30312c` = `"host,case\nWS-01,"`, a CSV header, and it was accepted.
- **A submission-logic bug in our tooling** (see the day overview): after an unrelated composite
  failure the plan was moved to each group's alternate variant before the primary was tested alone,
  so correct values were discarded. After fixing it, the plain primary values for all 36 remaining ids
  went through in a single CORRECT call. Total for regular Mastery was 31 calls under a raised cap.

The regular Mastery then settled on 162/162. A ranked Mastery attempt (200 s window) was CORRECT in 1
call at 4.01 s. Ranked runs were bragging rights only, not the prize ranking. A later rerun was done
because the first ranked CORRECT had been credited to a solo account rather than the team; it was CORRECT
in 1 call.

## Dead ends and gotchas

- **Fields vary per set in Mastery.** The synthesis question's `Files:` list, the set manifest and even
  which file a prefix/BOM/hash question names differ between sets, so the extractor has to resolve the
  member from each question's own text and ID. Never hard-code one member. (The BOM field had the same
  problem on a smaller scale: it read only `evidence-note.txt`.)
- **Empty or `none` for an enum.** The BOM question is `true`/`false` only; both initial candidates
  emitted `none` or an empty string for the BOM field and failed lint before any submission.
- **Rubric rules held.** Zero-padded hex for PE entry RVA and machine values, lowercase unbraced .NET
  MVID, braced upper-case MSI ProductCode as stored, angle-bracketed Message-ID. The journal's early
  suspicion that these rules had broken on this specimen was wrong; it was the bisection bug above. The
  single-set journals record this correction after a later review.
- **Answer-format reminder:** every answer value was submitted as a string, including compact JSON arrays.
- Not checked: no jar in the saved fixtures had a `Main-Class` that disagreed with the launcher, so we
  could not confirm which reading the validator would prefer in that case.

## Takeaways

- For "what is runnable" JAR questions: parse `META-INF/MANIFEST.MF` (joining 72-column continuation
  lines) and cross-check by scanning every class file's constant pool for `this_class` plus a
  `public static void main([Ljava/lang/String;)V`. Do not stop at the class named by the manifest.
- In a recap/Mastery bundle, resolve each answer's source file from the question ID, not from a
  template. A hex string that decodes to readable ASCII from the wrong file is a quick tell for a
  mis-read member.
- When one batch of a large composite submission fails, test each group's primary reading on its own
  before moving to alternates; an unrelated bad field can otherwise make correct values look wrong.

## Evidence

Leaf base: `challenges/day22/metadata-forensics/metadata-forensics-day-22-main-character-energy/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-rubric/solve.py` (`jar_fields`, `class_info`, the fixed `magic-prefix-hex-16` branch)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`
- `mastery/journal.md` (post-run investigation and resolution), `mastery/regular/result.json`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `challenges/day22/_run/lane-metadata-forensics-day-22-main-character-energy.log`
- `library/metadata-forensics/` (evidence-bundle toolkit, gotchas for Days 21-23)
- `progress.md` (grep `^| 22 .*main-character-energy`)
