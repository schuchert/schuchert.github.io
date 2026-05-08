# Stochasticinator Widget Rewrite — Design

**Date:** 2026-05-08
**Scope:** The embedded CSS/JS widget in `_posts/2026-04-30-I Stochasticinator:SomaticOSEngine.md`
**Goal:** Replace the current inline widget with a cleaner, accessible, themeable implementation while preserving observed behavior (click → roll a primary hexagram with 5/6 probability of a secondary transformation, render four lenses each).

## Motivation

Code review of the existing widget surfaced:

- **One real bug:** secondary hexagram can equal primary (~1/64 of rolls).
- **Visible repetition:** 55 of 64 hexagrams fall through to one of only 3 stock paragraphs per lens, cycled by `i % 3`. Reads as canned content.
- **Inline-style churn:** ~20 style strings duplicated across headers, panels, and containers; no theme awareness.
- **Accessibility gaps:** result region has no `aria-live`; decorative glyph is not hidden from screen readers.
- **`innerHTML`-based rendering:** safe with today's static data but fragile if data ever becomes dynamic.

The author maintains the post under version control, can host locally, and has authorized a full rewrite.

## Non-goals

- Test harness. The Jekyll blog has none; verification is manual in a browser.
- Authoring the 55 missing four-lens entries. Placeholders ship; author will generate content separately.
- External JS or a Jekyll `_data/` file. Everything stays inline in the post.
- Changing the roll mechanics (1D64 primary, 5/6 probability of 1D64 secondary). Line-based transformation is out of scope.

## Architecture

Three contiguous blocks inside the post's `{::nomarkdown}` fence:

1. **`<style>`** — CSS custom properties scoped to `.hexagram-oracle`, with a `@media (prefers-color-scheme: dark)` override block. All visual rules reference the variables; no hardcoded colors in markup or JS.
2. **Markup** — one `<div class="hexagram-oracle">` containing a `<button class="hexagram-oracle__roll">` and a `<div class="hexagram-oracle__result" aria-live="polite">` for output.
3. **`<script>` IIFE** — three internal concerns:
   - `HEXAGRAMS` data (all 64 entries, same shape)
   - `roll()` pure function
   - DOM builders (`renderHexagram`, `renderDrift`, click wiring)

Units are small enough to reason about in isolation. `roll()` can be verified by repeated invocation; rendering can be verified by inspecting the produced DOM.

## Data model

One object keyed 1..64. Every entry has the same shape:

```js
{
  name: string,
  traditional: string,
  meatMechanic: string,
  somatic: string,
  connection: string
}
```

- **Curated (9):** hexagrams 1, 2, 3, 4, 5, 11, 12, 63, 64 — preserve existing prose verbatim.
- **Placeholder (55):**
  - `name` — from the existing `baseDictionary`.
  - `traditional` — the second element of each `baseDictionary` row (the richer phrase, not just the name).
  - `meatMechanic`, `somatic`, `connection` — each set to the string `"— Pending — lens commentary awaiting generation."`.

The render layer detects the pending sentinel and applies a `.lens--pending` modifier class (muted italic). This communicates provisional state both visually and textually. The current 3-variation cycling arrays (`mmVariations`, `somVariations`, `connVariations`) are deleted.

## Logic

```js
function roll() {
  const primary = Math.floor(Math.random() * 64) + 1;
  let secondary = null;
  if (Math.random() < 5 / 6) {
    do {
      secondary = Math.floor(Math.random() * 64) + 1;
    } while (secondary === primary);
  }
  return { primary, secondary };
}
```

- Secondary probability is `5/6` directly; no intermediate `secondaryRoll` variable.
- Collision guard ensures `secondary !== primary` when a secondary is drawn.
- Pure — no DOM access — so behavior is self-evident from the code.

## Rendering

DOM-building, not string templating. Each roll clears the result region and appends freshly-built nodes:

```
<article class="hexagram hexagram--primary">         (or --secondary)
  <h2 class="hexagram__title">Hexagram 7: The Army</h2>  (h3 if secondary)
  <div class="hexagram__symbol" aria-hidden="true">䷇</div>
  <section class="lens lens--traditional">
    <h4 class="lens__title">1. The Traditional Lens</h4>
    <p class="lens__body">…</p>
  </section>
  <section class="lens lens--meat">…</section>
  <section class="lens lens--somatic">…</section>
  <section class="lens lens--connection">…</section>
</article>
```

When a secondary rolls, a trailing `<aside class="drift">` is appended with the transformation-drift message.

- Unicode hexagram glyph computed as `String.fromCodePoint(0x4DC0 + n - 1)`.
- Glyph is decorative — the `h2`/`h3` title carries the identity, so the glyph gets `aria-hidden="true"`.
- Text content written with `.textContent` (not `innerHTML`) so data cannot inject markup.

## Styling

CSS variables live on `.hexagram-oracle`:

```css
.hexagram-oracle {
  --bg: #f5f5f5;
  --panel: #fff;
  --text: #111;
  --border: #333;
  --primary: #009900;
  --secondary: #cc6600;
  --traditional: #009900;
  --meat: #cc0000;
  --somatic: #0066cc;
  --connection: #9900cc;
  --drift-bg: #ffffcc;
  --button-text: #fff;
}

@media (prefers-color-scheme: dark) {
  .hexagram-oracle {
    --bg: #1a1a1a;
    --panel: #232323;
    --text: #e8e8e8;
    --border: #666;
    --primary: #7fd67f;
    --secondary: #ffaa55;
    --traditional: #7fd67f;
    --meat: #ff6b6b;
    --somatic: #6bb0ff;
    --connection: #c98dff;
    --drift-bg: #2d2a16;
    --button-text: #111;
  }
}
```

Rules reference only these variables. Lens panels pick their accent through per-lens classes (`.lens--traditional` → `border-left-color: var(--traditional)`, and so on). Pending lenses use `opacity` and `font-style: italic` — no separate color.

Layout matches the existing widget (vertical stack of lens panels with colored left borders) so visual identity is preserved.

## Accessibility

- `aria-live="polite"` on the result region so screen readers announce new content after each roll.
- `aria-hidden="true"` on the decorative glyph.
- Headings descend cleanly from the post's `h1`: `h2` primary hexagram, `h3` secondary hexagram, `h4` lens titles.
- The button has a clear accessible name from its visible text.
- Pending state is communicated both semantically (literal text `"— Pending —"`) and visually (italic, reduced opacity).
- Color contrast pairs chosen to meet WCAG AA in both light and dark palettes (spot-check target: 4.5:1 for body text).

## Verification (manual)

1. `bundle exec jekyll serve` from repo root; open the post.
2. Click "Engage Stochastic Roll" ~30 times. Confirm:
   - Rolls span the full 1..64 range over time.
   - Some rolls produce no secondary; most do (roughly 5:1).
   - Secondary is never equal to primary.
   - Curated hexagrams (1, 2, 3, 4, 5, 11, 12, 63, 64) show full four-lens prose.
   - Non-curated hexagrams show the traditional line plus three italicized pending lenses.
3. Toggle system appearance (macOS System Settings → Appearance). Confirm both palettes read cleanly without layout shifts.
4. With VoiceOver on, click the button and confirm the result region is announced.
5. Resize viewport to phone width; confirm panels stack cleanly.

## Files touched

- `_posts/2026-04-30-I Stochasticinator:SomaticOSEngine.md` — the `{::nomarkdown}` block is replaced end-to-end. Frontmatter and surrounding prose are untouched.

## Open questions

None. All design decisions resolved during brainstorming:

- Scope: full rewrite authorized.
- Layout: all inline in the post.
- Theming: `prefers-color-scheme` with CSS variables.
- Fallbacks: pending-marked placeholders for 55 hexagrams; author fills in later.
