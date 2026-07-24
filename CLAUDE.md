# CLAUDE.md

## What this is

Interactive retellings of the Iliad (Samuel Butler's translation, rewritten in Paul Graham's essay style), one self-contained HTML page per book. `book_1.html` is the reference implementation — treat it as the template for every new chapter.

## Repo layout

- `index.html` — homepage: chapter registry for the whole work, per-chapter and overall progress read from localStorage, "Coming soon" cards for unpublished books. Deployed via GitHub Pages (push to main = deploy).
- `raw/book_N.md` — Butler source text per book. Read-only reference, gitignored, never edit or commit.
- `book_N.html` — the finished page for book N. Committed.
- New chapter workflow: user drops `raw/book_N.md`, you produce `book_N.html` by following "Adding a chapter" below.

## Hard constraints (every chapter)

- **Single file.** All CSS in `<style>`, all vanilla JS in one `<script>` at end of `<body>`. No frameworks, no build step, no external requests. localStorage is allowed for ONE thing: reading progress (see schema below); never anything else.
- **Palette is fixed.** CSS vars in `:root`: `--bg`, `--surface`, `--text`, `--muted`, `--faint`, `--clay` (accent). Serif (`--serif`) for prose and names, sans (`--sans`) for UI chrome. Don't change them — chapters must look like one publication.
- **Character spans are static HTML.** Essay names are wrapped by hand in `<span class="char" data-char="ID" tabindex="0" role="button">`. Never wrap names with a runtime regex — it would catch the SVG, glossary, and decoder table. Butler's Roman names are aliases (e.g. "Ulysses" → `odysseus`, "Jove" → `zeus`): wrap the alias text, point `data-char` at the Greek id.

## Adding a chapter (book_N.html)

1. Copy the newest existing `book_N.html` — CSS and the whole `<script>` are chapter-agnostic; only content changes.
2. Read `raw/book_N.md`. Rewrite as a PG-style essay: plain words, short sentences, talking to a smart friend; lead with the human mechanism (status, incentives, honor), not plot summary.
3. Rebuild the chapter-specific parts, keeping ids stable across books (a character keeps the same id in every chapter):
   - `CHARACTERS` object — this book's cast only; `hasNode: false` for anyone left off the map.
   - Essay `<article>` with hand-wrapped spans (aliases included).
   - Glossary `.person` cards — bios must stay word-identical to `CHARACTERS` bios.
   - SVG map — nodes for the major cast, `data-chars` on every edge path AND its label.
   - Quizzes — ~5 `.quiz` blocks, each right after the passage it tests, aimed at the human-condition point of that passage, not plot trivia.
   - Decoder table — only rows for Roman names that actually appear in this book's source.
4. Register the chapter in `index.html`'s `WORK.chapters` array (set `file` and `checkpoints` = number of quizzes), update README's page list, and add the Contents/prev/next links in `.book-nav` (new page links back, previous page links forward). Verify (below), then commit the html — never `raw/`. Pushing main deploys GitHub Pages.

## Architecture (identical in every chapter)

- `CHARACTERS` JS object keyed by id; `hasNode` gates map interactions. Glossary bios and `CHARACTERS` bios word-identical.
- SVG map (`#relmap`): each box is `<g id="node-ID" class="node" data-node="ID">`. Every edge path AND its label carry `data-chars="id1,id2"`. Highlight logic walks `[data-chars]` — a new edge only needs that attribute to work.
- Highlight = `dimmed` class on svg (everything to opacity .15) + `lit` on selected node, touching edges, neighbor nodes + `sel` on selected node (2.5px `--clay` outline). Selection also fills `#char-detail` card above the SVG and filters the glossary to the lit set. Chip `#map-chip`, Esc, or svg background clears all of it.
- Quizzes gate the essay: everything after the first unpassed quiz gets `.locked` (display:none) plus a `.continue-hint` after that quiz; wrong picks mark `.wrong-pick` and stay clickable (retry), only the correct option reveals `.quiz-why` and unlocks the next section. Static `.quiz` blocks, `data-answer` on the block, `data-opt` on buttons. Names inside quiz blocks stay unwrapped (spans can't nest in buttons).
- Progress: localStorage key `iliad:book_N` holds a JSON array of passed quiz indices; `#chapter-progress` in the header shows n-of-total and a fill bar; passed quizzes re-render answered on load. A sticky checkpoint rail (`.rail`, first grid column at ≥920px — grid is `56px / essay / sidebar`) shows one numbered dot per quiz (passed = filled clay, current = outlined, locked = faint); passed/current dots scroll to their quiz; `renderRail()` runs from `updateProgress()`. index.html reads the same keys for per-chapter and overall progress, and offers a reset. Storage failures (private mode) must fail silently.
- Quiz options must not telegraph the answer. Keep all options similar length and parallel structure ("X — clause"); the correct one must never be identifiable as the longest or most detailed, and its position (a/b/c) must vary across the page. Distractors have to be genuinely tempting: partially-true readings, lessons pattern-matched from earlier books, or options that quote the text — each wrong for a reason the `.quiz-why` explicitly names (ideally the why addresses why the best distractor fails).
- Layout: ≥920px, two-column grid — main column is the essay ONLY; the sticky sidebar holds map (on top via `order: -1`), glossary, then the decoder table at the bottom. Breakpoint is deliberately low: browser zoom shrinks the CSS-pixel viewport, and higher breakpoints collapse the sidebar at ~150% zoom. Grid must NOT use `align-items: start` — the sidebar column has to stretch or sticky has no travel room. Wide screens scroll the sidebar, not the page; narrow screens stack and use `scrollIntoView`.
- Tooltip: one reusable `#char-tooltip`, position: fixed, 150ms hover delay / instant on focus, viewport-clamped, flips below when no room above, `aria-describedby` while visible. Touch: first tap shows, second activates.
- `prefers-reduced-motion`: instant scroll, no opacity transitions.

## Checking work

No tests. Open the page in a browser; check console is clean, keyboard-only flow works (Tab to span, Enter to highlight, Esc to clear), tooltip behaves at 380px width, sidebar still follows scroll at 150% zoom, wrong quiz picks allow retry, the correct pick unlocks the next section and bumps the progress bar, and a reload restores passed quizzes from localStorage.
