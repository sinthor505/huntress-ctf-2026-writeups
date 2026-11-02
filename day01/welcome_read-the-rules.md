# Read the Rules

| | |
|---|---|
| Day / Lane | 1 / Welcome |
| Challenge ID | `read-the-rules` |
| Technique | Hidden HTML comment in a client-rendered page (render, then search the DOM) |
| Single Solve | Solved (journal-recorded): our own submission read CORRECT on attempt 2 per the journal; no `result.json` was preserved in the leaf |
| Time Trial | Not offered (Welcome challenges are Single Solve only) |
| Mastery | Not offered |

## Summary

The prompt asks you to read the event's public rules page and find a flag hidden in its HTML. The page is a Next.js shell that renders its content in the browser, so `curl` shows nothing useful. Rendering it in headless Chromium and searching the DOM turned up a footer HTML comment holding `flag{...}`. The bare 32-hex value was rejected and the full comment text was accepted.

## The challenge

The prompt says to read the rules at the event's public rules URL and that "the flag is hidden in the page itself, invisible on screen, but present in the page HTML". It asks you to submit "the exact flag value you find". The Game Master delivered no artifact and no live target. The start response has no questions, one answer ID (`flag`) and no deadline. This is an orientation challenge rather than a solver puzzle.

## Solution

1. Fetched the rules URL with `curl`. The response was a Next.js shell: no rules text and no flag. The body is client-rendered, so a static fetch cannot see it.
2. Rendered the page in headless Chromium (Playwright) and dumped the post-render DOM. `page.content()` keeps HTML comments, which is where the flag was.
3. Searched the rendered DOM and found the flag in a footer HTML comment. The visible rules text says "There is no flag to be found on this page". That line is a joke, and the comment is the intended hidden flag.
4. Submitted the comment text exactly as found. Answer: `flag{bf61aced6e7f9335385a70f33b20d188}`. This was `CORRECT`.

## Dead ends and gotchas

- The rules page states the flag format as an MD5 (`[0-9a-f]{32}`). The first submission was the bare 32-hex value from inside the braces, and it came back `incorrect`. The full `flag{...}` text, wrapper included, was accepted on the second call. When a page states a format that conflicts with the text you found, submit the found text verbatim first.
- Static fetching proves nothing here. The page is a JS shell, so "nothing in the HTML" only meant nothing before rendering.

## Takeaways

- For a "hidden in the page" challenge on a JS-framework site, render the page in a headless browser and search the resulting DOM for comments, `display:none` nodes, hidden inputs, meta tags and `data-*` attributes.
- Submit the exact string you found before trying a normalized form. This cost one extra call here.

## Evidence

Leaf base path: `challenges/day01/welcome/read-the-rules/single-solve/`

- `journal.md`: recon, rendering approach and the two submissions.
- `start_response.json`: prompt, `answer_ids: ["flag"]`, no artifacts or deadline.
- `artifacts/answers-bare-hex-incorrect.json` and `artifacts/answers-wrapped.json`: the two submitted payloads.
- Provenance: `progress.md` records the mode as Solved on 2026-10-23, with Game Master CORRECT on attempt 2. The leaf holds no `result.json`, `accepted_plan.json` or `manual_solve.json`. The graded CORRECT rests on the journal and the `progress.md` row only. The rendered DOM was not saved locally.
