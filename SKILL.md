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

1. **Place the anchor** — typically the sidebar logo or brand. Give it equal insets (top padding = left padding). Get it in a beautiful spot.
2. **Draw the line** — the anchor's baseline becomes the horizontal line.
3. **Align everything else** — every other element on the same visual row adjusts to match the anchor's baseline.

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

### Rules

1. Always run the procedure. Never eyeball alignment or guess padding values.
2. Place the anchor first. Everything else follows.
3. Use `items-center` on both rows. Never use `items-end` with manual bottom padding — it breaks when adjacent elements (like icon buttons) have fixed heights.
4. Use the same row height and font size for both elements so the centering terms cancel and only the cumulative offset matters.
5. Icons centered in a button box align automatically when the button is centered in the same row as the text — no separate icon alignment needed.
6. When the fix requires added padding, use a token value or a `var()` reference, not a magic number.

---

## Vertical Rhythm

Every piece of content sits on one of two depth levels. Each level has a consistent left-edge offset from the content area's left boundary. **All text at the same level must share the same x-coordinate.**

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

1. Exactly two vertical lines. A third line means inconsistent padding somewhere.
2. `card_padding` is one value used everywhere — every card, alert, table header, and table cell uses the same inner padding.
3. The toolbar is a shell element at level 0. Do not push it to level 1.
4. Page headings and descriptions are shell elements at level 0 — no surface wrapper means no extra inset.
5. Card borders align to level 0. Card content aligns to level 1.
