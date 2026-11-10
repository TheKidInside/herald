# Herald — How It Works (a learning walkthrough)

Herald is a mockup, so this is short. The point of the file is the three ideas it
demonstrates, because the real build will keep all three.

## Part 0 — Run it

Open `index.html` in a browser, or visit https://thekidinside.github.io/herald/.
Navigation is by hash: `#/` is the front page, `#/people` … `#/pulpit` are sections.
Add `?frame=phone` to see the page inside a 390 px phone frame on a desktop.

## Part 1 — The big picture

### 1.1 Three primitives, seven filters
All data lives in three arrays near the top of the script: `LOOPS`, `LOGS`, `BACKLOGS`.
Every row carries a `sec` (section id) and a `tab`. A section page is nothing more than
`LOOPS.filter(l => l.sec === id && l.tab === tab)` rendered by the primitive's renderer.
`SECTIONS[].tabs` declares which primitive each tab uses, e.g. `['Mentoring','loop']`.
Adding a section is a row in `SECTIONS`; adding a tab is a pair in `tabs`. No new code.

### 1.2 Headlines are computed, not written
`headline(loop)` turns a loop's fields into a sentence: a drifting loop becomes
"*Title* has gone quiet", a loop with an arc becomes "*Title* on Wednesday: *Arc*, week 3 of 7".
The front page sorts non-`ok` loops by urgency (`sortLoops`: due today first, then due this
week, then drifting) and the first one is the lead story. Sections with nothing due appear
only in the "Nothing to report" notices column, so a quiet day prints a thin paper.
This is the Loop plan's digest rendered as a page.

### 1.3 Last touch drives everything
`last` (days since the loop was touched) produces the "34 days quiet" byline and the
Drifting pill. In the real build, writing a Log entry on a subject stamps that subject's
`last_touch`, so the check-in *is* the update. Never ask the user to maintain state separately.

## Part 2 — Techniques worth keeping

### 2.1 Section colour as a CSS variable
Each section has a colour token (`--people`, `--family`, …). A wrapper sets
`style="--sec:var(--people)"` and every child uses `var(--sec)`. One rule, seven colours,
and dark mode only has to redefine the seven tokens.

### 2.2 Phone lock
Viewport `maximum-scale=1, user-scalable=no`; `gesturestart` and two-finger `touchmove`
cancelled in `<head>`; `overflow-x: clip; overscroll-behavior: none`. Under 760 px the body
is `position: fixed` and only `.scroller` scrolls; the section nav becomes a fixed bottom bar.
This is the standing rule for every one of Kevin's web apps.

### 2.3 Theme before paint
An inline `<head>` script sets `data-theme` from `localStorage` before the stylesheet
applies, so a dark-mode user never sees a flash of cream. Tokens are defined three times:
light on `:root`, dark under `prefers-color-scheme` guarded by `:root:not([data-theme="light"])`,
and dark again under `:root[data-theme="dark"]`.

### 2.4 The phone frame
`?frame=phone` hides the page and renders an `<iframe>` of the same URL without the
parameter inside a phone-shaped box. The iframe is 390 px wide, so the responsive CSS
inside it switches to the phone layout on its own. One file previews both views.

## Part 3 — What is deliberately left out

No storage, no sync, no auth. The real build takes its data from the loop slice that
`scan/loops.py` will push (see `plans/personal-ai-assistant.md` §4) and sends answers
("preached it", "snooze") to the intent queue, never to the vault directly.
