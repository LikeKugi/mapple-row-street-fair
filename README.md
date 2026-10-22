# Maple Row Street Fair — Event Flyer Page

A one-page event flyer built for the freeCodeCamp **Build an Event Flyer Page** CSS lab.
The page is a fictional one-day block party, designed as a small-press poster: newsprint
paper, a pasted photo, painted display type, and a bunting ribbon across the top edge.

No frameworks, no build step — one HTML file and one stylesheet.

```
index.html   markup + Google Fonts links
styles.css   the entire design system
```

---

## 1. The issue

The lab ships as a **specification-only** challenge: a list of user stories, and a set of
automated assertions that read the computed styles of `body`, `hr`, and `section`. Nothing
is scaffolded, so the whole job is to satisfy the layout contract **and** produce something
that actually looks like a flyer.

The 17 assertions can be grouped into three problems:

| Group | What it forces |
| --- | --- |
| **Semantics** (1–9, 15) | A `header` containing `img` then `h1` **in that order**, exactly one `h1`, a `main` with ≥ 2 `section`s, each with a non-empty `h2`, and at least one `hr`. |
| **The sheet** (10–14) | `body` gets `padding: 50px 0`, `margin: 0 auto`, a `vw` width, and `min-height: calc(100vh - 100px)`. |
| **Proportional rhythm** (16–17) | Every `hr` and `section` width must be a **percentage** of its parent. |

Two of those constraints quietly conflict with the obvious solution, and both are where the
design work actually happens:

1. **`margin: 0 auto` on `body` only centers if `body` is narrower than the viewport** — so
   the "flyer" has to be a sheet of paper floating on a background. That is what turned the
   page into a poster: a dark wall behind it, a printed sheet in front.
2. **`hr` and `section` widths must be percentages, while `body` uses `vw`** — mixing the two
   units means nothing lines up unless inner elements use percentages of the *sheet*, not the
   *screen*. The layout therefore shares one 88% width across header, dateline, lede, sections
   and footer — the lede nudged with a 6% left margin so it hangs under the title — with the
   `hr` inset further to 72% so the dot rule reads as a divider rather than a full-width bar.

The real difficulty was not passing the tests; it was making a page whose box model is dictated
by tests still feel art-directed rather than assembled.

---

## 2. Design direction

**Concept:** a letterpress poster pinned to a dark wall. Paper texture, hairline rules, one
loud accent, and type that does the shouting so the layout doesn't have to.

| Token | Value | Role |
| --- | --- | --- |
| `--paper` | `#f7edda` | newsprint sheet |
| `--paper-shade` | `#efe1c8` | image placeholder / pressed paper |
| `--ink` | `#16241e` | pine-black text and 2px rules |
| `--ink-soft` | `#4c554b` | body copy |
| `--tomato` | `#d3502f` | primary accent, drop shadows, dots |
| `--gold` / `--teal` / `--blue` | `#e3a72f` / `#2f6b5c` / `#3d6b8c` | bunting ribbon |

**Type stack**, three voices doing three jobs:

- **Bricolage Grotesque** — the `h1` at weight 800 with tight `-0.045em` tracking at `0.84`
  line-height so the title reads as a painted sign; the same family carries body copy at 400.
- **Instrument Serif italic** — the second half of the title, all `h2`s, the lede, and the
  footer call to action. This is the "voice" of the flyer.
- **DM Mono 400/500** — dateline, feature numbering (`01 Eat`), times, and footer meta. It
  gives every factual detail a stamped, ticket-like feel.

**Layout:** `body` is a flex column with `min-height: calc(100vh - 100px)`, so the footer's
`margin: auto auto 0` pushes it to the bottom on short viewports without breaking the min-height
contract. The header is a 2-column grid (`minmax(0, 1fr) minmax(0, 0.6fr)`) with the photo
placed in column 2 via explicit `grid-column` / `grid-row`, which keeps the `img` first in DOM
order while letting it sit to the right of the title.

---

## 3. Requirement → implementation

Every assertion, mapped to the exact rule that satisfies it.

| # | Requirement | Implementation |
| --- | --- | --- |
| 1 | `header` in `body` | `index.html:19` → `<header class="flyer-header">` |
| 2 | `img` is first child of `header` | `index.html:20` → `.flyer-photo` |
| 3 | `h1` is second child of `header` | `index.html:22` |
| 4 | Only one `h1` | Sections use `h2`; nothing else is a heading |
| 5 | `h1` has the event title | `<span class="title-line">Maple Row</span> <em>Street Fair</em>` |
| 6 | `main` in `body` | `index.html:25` |
| 7 | ≥ 2 `section`s in `main` | 3 — `.feature` for Eat / Listen / Make |
| 8 | Each `section` has an `h2` | "Eat your way down the block", "Two stages, no headliner", "Makers, menders & oddities" |
| 9 | `h2`s not empty | All three have text content |
| 10 | `body` padding `50px` top/bottom | `padding: 50px 0` |
| 11 | `body` margin `0` top/bottom | `margin: 0 auto` |
| 12 | `body` margin `auto` left/right | `margin: 0 auto` |
| 13 | `body` width in `vw` | `width: 92vw` (+ `max-width: 1000px` guard) |
| 14 | `min-height: calc(100vh - 100px)` | `min-height: calc(100vh - 100px)` |
| 15 | ≥ 1 `hr` | 2 — after the lede and after the last section |
| 16 | `hr` width in `%` | `hr { width: 72% }` |
| 17 | `section` width in `%` | `section { width: 88% }` |

The box math that satisfies 10–14 in one declaration block:

```css
body {
  width: 92vw;                        /* 13 — fluid, viewport-relative sheet */
  min-height: calc(100vh - 100px);    /* 14 — 100vh minus 50px top + 50px bottom */
  margin: 0 auto;                     /* 11 + 12 — flush top/bottom, centered sideways */
  padding: 50px 0;                    /* 10 — the 50px the calc subtracts */
}
```

`100px` is not a magic number here: it is exactly the vertical padding, which is why the sheet
fills the viewport edge-to-edge with no scrollbar while the footer still pins to the bottom.

---

## 4. Details worth pointing at

**Bunting ribbon.** A 4-stop `repeating-linear-gradient` on `body::before`, pinned to the top
edge with `inset: 0 0 auto` and an 8px height:

```css
background-image: repeating-linear-gradient(90deg,
    var(--tomato) 0 25%, var(--gold) 25% 50%,
    var(--teal) 50% 75%, var(--blue) 75% 100%);
background-size: 144px 100%;
```

**Dotted `hr`.** No border — a radial-gradient dot row layered over a soft horizontal rule,
so the divider is 12px tall but only 1px of it is a line. Both are `background-image` layers on
a single `hr`, keeping the percentage width (test 16) intact:

```css
hr {
  width: 72%;
  height: 12px;
  border: 0;
  background-image:
    radial-gradient(circle, var(--tomato) 2.3px, transparent 2.5px),
    linear-gradient(90deg, transparent, var(--rule) 16%, var(--rule) 84%, transparent);
  background-size: 14px 12px, 100% 1px;
}
```

**"One day only" stamp.** Generated content on `.flyer-header::after` — a dashed circle rotated
`-9deg` and pressed over the photo corner. Purely decorative, so it lives in CSS rather than the
DOM, and never reaches a screen reader.

**Photo as a taped print.** `transform: rotate(1.6deg)` plus a hard `12px 12px 0 var(--tomato)`
offset shadow; on header hover the rotation resolves to `0deg` and the print lifts. This is the
one interaction in the page, and it costs two properties.

**Paper grain.** Inline `feTurbulence` SVG in a `data:` URI at `opacity: 0.045`, applied to both
`body` and `html`. It adds no request and no file, and makes the flat cream background read as
stock.

**Responsive collapse.** A single `760px` breakpoint flattens both grids to one column and
reorders the header with `grid-row` so the photo stacks above the title while remaining first
in the DOM — semantics and tests stay valid at every width.

**Motion.** Staggered `rise` animations (0.05s → 0.85s) and a `photo-in` keyframe, all disabled
inside `@media (prefers-reduced-motion: reduce)`. `:focus-visible` outlines on the footer link
use `outline-offset: 3px` so they clear the underline.

---

## 5. How it was built — AI-driven development

This page was produced in an agentic loop with the lab's `AGENTS.md` as the contract. The
important part is that **the spec was treated as executable input, not as prose to skim**.

**1. Spec ingestion.** The user stories and the 17 assertions were read into a checklist
first. The two structural traps (percentage nesting vs. `vw`, and the "centering that only works
if the element is narrower than the viewport") were identified *before* any markup was written,
because both constrain the visual concept rather than following from it.

**2. Design proposal.** A direction was chosen up front — letterpress poster on a dark wall —
with a fixed token set (palette, three typefaces, one gutter). Committing to tokens early is
what keeps a generated page from drifting into a pile of one-off values.

**3. Implementation.** Markup was written to satisfy the semantic order literally (`img` then
`h1` inside `header`, one `h1`, `main` → three `section`s → one `h2` each), then the CSS was
built outward from the box model: `body` box first, shared 88% gutter second, components third,
grain and motion last.

**4. Self-verification.** Rather than assuming the browser got it right, the contract was
re-checked mechanically — the header's child tags were parsed in order, and counts for
`h1`, `main`, `section`, `h2`, and `hr` were asserted against the spec:

```
header child tags in order: img -> h1
main count: 1
h1 count: 1
sections: 3
h2s: 3
hrs: 2
```

**5. Review pass.** Final read of the diff against the user stories, plus a check that nothing
added for style broke a tested selector — e.g. `hr` is styled with `background-image` instead of
`border` specifically so its percentage width stays the only thing setting its box.

**What the agent did and did not decide.** Implementation, token selection, and the verification
harness were agent-driven. The acceptance criterion stayed the lab's — the tests are the source
of truth, and every visual decision had to remain compatible with them. Auto-generated work is
easy to spot when it satisfies assertions in isolation; the goal here was a page where the
constraints are invisible.

---

## 6. Running it

There is no build step and no dependency to install (`package.json` is a placeholder — its
`test` script is the npm default and the real suite runs in the freeCodeCamp lab runner).

Open the file directly:

```bash
open index.html          # macOS
```

Or serve it, which is closer to how the lab loads assets:

```bash
python3 -m http.server 8000
# → http://localhost:8000
```

**To validate against the lab:** load the project in the freeCodeCamp *Build an Event Flyer Page*
lab editor and run the test panel. All 17 assertions above are satisfied by the current markup
and computed styles.

Requires a network connection for the Google Fonts and the freeCodeCamp CDN image; both degrade
to system fallbacks (the font stacks end in `sans-serif` / `serif` / `monospace`).

---

## 7. Notes and limitations

- Content is fictional; `hello@maplerowfair.org` is an illustrative `mailto:` and goes nowhere.
- Dates are hardcoded ("Sat · Oct 24") rather than generated, which matches a printed flyer but
  will go stale — acceptable for a design lab.
- The photo uses `aspect-ratio: 286 / 176` to reserve space and avoid layout shift, so a
  differently-sized replacement image will crop rather than resize the header.
- Single page, no JS, no bundler. If this grew beyond one flyer, the obvious next step is
  extracting the token block into a shared `theme.css`.
- Verification here is static (structure + declared CSS values). The canonical pass/fail signal
  is the lab's own test runner, which cannot be executed from this repository.

