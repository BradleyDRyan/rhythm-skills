---
name: rhythm
description: Use when building any multi-container layout that needs alignment — dashboards, sidebars with content areas, multi-panel UIs, or any design where text crosses container boundaries. Applies to any framework or CSS approach.
---
# Rhythm

Two formulaic alignment skills for any UI with multiple containers. No eyeballing — compute the position, then apply it.

## Horizontal Rhythm

When two elements sit on the same visual row but live in different containers, their baselines must align.

### The anchor

The anchor is the element you design *first*. Give it perfect spacing, then its position becomes the truth that everything else aligns to.

1. **Place the anchor** — typically the sidebar logo or brand. Give it equal insets (top padding = left padding). The left inset places it on a vertical lane. The top inset sets the baseline. Same value, two jobs.
2. **Draw the line** — the anchor's baseline becomes the horizontal line.
3. **Align everything else** — every other element on the same visual row matches the anchor by sharing three properties: **top offset**, **row height**, and **line height**. When all three match, alignment is free — no manual fixes needed.

### Procedure

**Step 1 — Identify elements that need horizontal rhythm.** Before writing any layout code, draw an imaginary horizontal line across the design. Any text, icon, or control that the line crosses and that lives in a different container than its neighbor is an alignment pair. Scan top to bottom — a layout can have multiple alignment lines, each with its own set of pairs. Every pair gets its own trace in steps 2–5.

**Step 2 — Trace each ancestor chain.** For each element, walk from the viewport down to the element. At every node, record its `margin-top` and `padding-top`:

```
Element A path:
  viewport
   └─ node_1  (margin-top: ?, padding-top: ?)
       └─ node_2  (margin-top: ?, padding-top: ?)
           └─ row_A  (height: ?, items-center)
               └─ element_A  (font-size: ?)

Element B path:
  viewport
   └─ node_1  (margin-top: ?, padding-top: ?)
       └─ node_2  (margin-top: ?, padding-top: ?)
           └─ row_B  (height: ?, items-center)
               └─ element_B  (font-size: ?)
```

**Step 3 — Compute baseline_y for each.** Sum every ancestor's top margin and padding, then add the centering offset:

```
cumulative_offset = Σ(margin_top + padding_top) for each ancestor
baseline_y = cumulative_offset + ((row_height - font_size) / 2) + font_ascent
```

`font_ascent` ≈ 0.8 × font_size for sans-serif fonts like Inter.

**Step 4 — Compare.**

```
baseline_y_A = baseline_y_B  →  aligned, no fix needed
baseline_y_A ≠ baseline_y_B  →  go to step 5
```

**Step 5 — Fix.** When both elements use the same row height and font size, the centering and ascent terms cancel. The fix reduces to:

```
fix = cumulative_offset_B - cumulative_offset_A
```

Add `padding-top: fix` to whichever element's chain has the smaller cumulative offset. Prefer adding the padding to the row container, not the element itself.

### Type hierarchy and horizontal rhythm

Font size choices directly affect the baseline equation. When two elements share a font size, the `((row_height - font_size) / 2) + font_ascent` terms cancel and alignment is free. When they don't, you have to compensate with padding — and the math gets harder.

This means **type hierarchy and horizontal rhythm are the same decision.** Elements that share a horizontal line should share a font size. Differentiate through weight, not size.

```
same top offset + row height + line height → free alignment
any one differs → baseline diverges → manual fix required
```

**The rule:** context-level text (page titles, toolbar titles, dates) matches the brand font size to preserve horizontal rhythm. It differentiates through weight, not size.

```
Brand:    14px / 600  →  baseline_y = X  (anchor — bold, owns the line)
Context:  14px / 400  →  baseline_y = X  (quiet — same line, lighter weight)
```

The page title tells you *where you are*, not *what to look at*. It should be the quietest confident text in the header — not the loudest thing on the page. The data below becomes the hero by default because the title got out of the way.

### Rules

1. Place the anchor first. Everything else follows.
2. Prefer free alignment (shared offset, row height, line height) over manual fixes. The trace procedure is a debugging tool, not the design method.
3. Use `items-center` on both rows. Never use `items-end` with manual bottom padding — it breaks when adjacent elements (like icon buttons) have fixed heights.
4. Elements on the same horizontal line should share a font size. Differentiate through weight, not size.
5. Row height is the taller of the line-height or tallest child (e.g. an icon). Two rows with the same font size but different icon heights will misalign.
6. Icons centered in a button box align automatically when the button is centered in the same row as the text.
7. When a manual fix is needed, use a token value or `var()` reference, not a magic number.

---

## Vertical Rhythm

Every visible left edge in the UI should sit on a named lane. Fewer lanes = calmer. Every element's left edge must be on a lane, and you must be able to name which lane and why.

### The equation

Every text element in the content area has a left-edge x-position:

```
x = Σ(margin_left + padding_left) for each ancestor from content edge to element
```

Two depth levels exist. Each has a formula:

```
level_0 = page_x
level_1 = page_x + card_padding
```

Two elements are vertically aligned when their `x` values are equal.

### Procedure

**Step 1 — Define the two constants.**

```
page_x       = ?   (shell inset — padding on the content scroll area)
card_padding = ?   (inner padding on every surface: cards, alerts, tables)
```

These two values produce exactly two vertical lines:

```
line_0 = page_x                      (shell level)
line_1 = page_x + card_padding       (content level)
```

Example: `page_x = 16`, `card_padding = 12` → line_0 = 16, line_1 = 28.

**Step 2 — Classify every element.** Walk the content area top to bottom. For each text element, determine its level:

- **Level 0 (shell):** toolbar titles, page headings, page descriptions, card borders. These have no surface wrapper — they sit directly inside the scroll container at `page_x`.
- **Level 1 (content):** text inside a card, alert, or table. These are nested one level deeper — `page_x` from the scroll container, then `card_padding` from the surface edge.

**Step 3 — Compute x for each element.** Trace the ancestor chain left-to-right, summing `margin_left + padding_left` at each node:

```
Element inside a card:
  scroll container (pl: page_x) → card (pl: card_padding) → text
  x = page_x + card_padding = line_1  ✓

Element without a card:
  scroll container (pl: page_x) → text
  x = page_x = line_0  ✓
```

**Step 4 — Verify.** Every element's `x` must equal either `line_0` or `line_1`. If an element produces a third value, its padding is wrong — find the ancestor with the inconsistent padding and fix it.

```
x = line_0  →  correct (shell level)
x = line_1  →  correct (content level)
x ≠ line_0 AND x ≠ line_1  →  broken — fix the padding
```

### Rules

1. Every left edge must be on a named lane. If an element sits between lanes, it's a bug. If you need another lane, name it and own it.
2. `card_padding` is one value used everywhere — every card, alert, table header, and table cell uses the same inner padding.
3. Interactive surfaces (hover fills, active backgrounds) have visible left edges — they're on a lane, not invisible implementation details.
4. The toolbar is a shell element at level 0. Do not push it to level 1.
5. Page headings and descriptions are shell elements at level 0 — no surface wrapper means no extra inset.
6. Card borders align to level 0. Card content aligns to level 1.
