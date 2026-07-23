# CLAUDE.md

## What this is

Single-file interactive retelling of Iliad Book One (Samuel Butler's translation, rewritten in Paul Graham's essay style). Everything lives in `book_1.html`. `book_1.md` is the Butler source text — read-only reference, don't edit.

## Hard constraints

- **Single file.** All CSS in `<style>`, all vanilla JS in one `<script>` at end of `<body>`. No frameworks, no build step, no localStorage, no external requests.
- **Palette is fixed.** CSS vars in `:root`: `--bg`, `--surface`, `--text`, `--muted`, `--faint`, `--clay` (accent). Serif (`--serif`) for prose and names, sans (`--sans`) for UI chrome. Don't change them.
- **Character spans are static HTML.** Essay names are wrapped by hand in `<span class="char" data-char="ID" tabindex="0" role="button">`. Never wrap names with a runtime regex — it would catch the SVG, glossary, and decoder table. Alias: "Ulysses" maps to `odysseus`.

## Architecture

- `CHARACTERS` JS object: 16 entries keyed by id. `hasNode: false` for nestor, calchas, menelaus, hephaestus (no SVG node). Glossary bios and `CHARACTERS` bios must stay word-identical.
- SVG map (`#relmap`): each character box is `<g id="node-ID" class="node" data-node="ID">`. Every edge path AND its label carry `data-chars="id1,id2"`. Highlight logic walks `[data-chars]` — a new edge only needs that attribute to work.
- Highlight = `dimmed` class on svg (everything to opacity .15) + `lit` class on selected node, touching edges, and neighbor nodes + `sel` on the selected node (2.5px `--clay` outline). Chip `#map-chip` shows current selection; Esc / svg background / chip button clear it.
- Quizzes: static `.quiz` blocks between essay paragraphs, `data-answer` on the block, `data-opt` on each button. JS reveals `.quiz-why` and marks right/wrong on first click; no score kept, no storage. Themes target the human condition (status, truth-to-power, incentives vs virtue, honor culture, mortality).
- Layout: ≥920px (kept low on purpose — browser zoom shrinks the CSS-pixel viewport, and a higher breakpoint dropped the sidebar at ~150% zoom), `.page` becomes two-column grid — essay (`.main-col`) left, sticky sidebar (`.side-col > .side-inner`) right holding map + glossary (map forced on top via `order: -1`). Clicking a name on wide screens scrolls the sidebar to top, not the page; narrow screens keep stacked layout + `scrollIntoView`. Grid must NOT use `align-items: start` — sidebar column has to stretch to full row height or the sticky inner has no travel room. Selection also fills `#char-detail` (a `.person` card above the SVG) with the character's name/role/bio; cleared with the highlight.
- Tooltip: one reusable `#char-tooltip`, position: fixed, shown after 150ms hover or instantly on focus, clamped to viewport, flips below when no room above. `aria-describedby` set while visible. Touch: first tap shows, second tap activates.
- `prefers-reduced-motion`: instant scroll, no opacity transitions.

## Checking work

No tests. Open `book_1.html` in a browser; check console is clean, keyboard-only flow works (Tab to span, Enter to highlight, Esc to clear), tooltip behaves at 380px width.
