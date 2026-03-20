---
name: rhythm
description: Use when building any multi-container layout that needs alignment — dashboards, sidebars with content areas, multi-panel UIs, or any design where text crosses container boundaries. Applies to any framework or CSS approach.
---
# Rhythm

Rules for beautiful spacing and layout in multi-container UIs.

## Section 1 — Terminology

Four named positions define where content sits inside any container. Every visible left edge must be on one of these.

**Container** — x=0. The region boundary itself. Nothing sits here except the background surface.

**Lane 1** — The interactive surface lane. Hover fills and active selection backgrounds have their left edge here. Most of the time it's invisible — it only reveals itself on interaction.

**Lane 2** — The primary content edge. Where all visual content starts — icons, avatars, section labels. This is the line your eye scans. 99% of everything aligns here.

**Lane 3** — The derived lane. Its position is set by Lane 2 content width + gap. Align it when Lane 2 content is consistent. Accept variation when it's not — a 22px logo icon pushes its text to one position, a 14px nav icon pushes to another, and both are correct.

**Side navigation example:**
- **Container:** sidebar background
- **Lane 1:** nav-item hover/active fill left edge (invisible until interaction)
- **Lane 2:** logo icon, nav icons, section labels, user avatar
- **Lane 3:** "TokenTrack Admin", "Dashboard", "Bradley Ryan" (set by Lane 2 content + gap)

**Content area example:**
- **Container:** content panel edge
- **Lane 1 (Line 0):** `page_x` — toolbar titles, page headings, card borders
- **Lane 2 (Line 1):** `page_x + card_padding` — text inside cards, alerts, tables

## Section 2 — Spatial Harmony

The x padding and y padding on an element should often be the same value. If the left padding is 16px, the top padding is often 16px as well.

This matters most at the anchor — the element you design first, typically the sidebar logo or brand. When its x and y insets match, the vertical lane position and horizontal baseline are set by one number. Everything downstream inherits that harmony.

**Example:** A sidebar logo with `padding: 16px` places its icon on Lane 2 at x=16 and sets the first horizontal baseline at y=16 + centering offset. The content toolbar next door uses the same 16px top padding — baselines align for free.

## Section 3 — Neighboring Alignment

Once an element has good spatial harmony on its own, check its neighbors. Elements in adjacent containers that sit on the same line should align.

### Horizontal

Elements in adjacent containers that sit on the same horizontal line should share a baseline. Alignment comes for free when both sides share the same top offset, row height, and line height. When it doesn't, one side has an ancestor the other doesn't.

The baseline y-position of any element is the sum of every ancestor's top contribution:

```
baseline_y = Σ(margin_top + padding_top) for each ancestor + ((row_height - font_size) / 2) + font_ascent
```

`font_ascent` ≈ 0.8 × font_size for sans-serif fonts.

To find a mismatch, trace both paths from viewport to element:

```
Element A: viewport → node(m:?, p:?) → node(m:?, p:?) → row(h:?) → text(fs:?)
Element B: viewport → node(m:?, p:?) → node(m:?, p:?) → node(m:?, p:?) → row(h:?) → text(fs:?)
```

The extra node in one chain is where the offset diverges. Compensate by adding padding to the shorter chain, or removing the extra offset from the longer one.

**Example:** A sidebar nav has `py: 8px` (shell padding) before the header row. The content area has `margin-top: 8px` (inset). Both use a 40px row and 14px font:

```
Sidebar:  0 + 8 + ((40-14)/2) + 11.2 = 32.2px
Content:  0 + 8 + ((40-14)/2) + 11.2 = 32.2px  ✓  match
```

**Type hierarchy note:** elements on the same horizontal line should share a font size — differentiate through weight, not size. Context text (page titles, dates) uses the same font size as the brand but lighter weight. The page title tells you *where you are*, not *what to look at*.

```
Brand:    14px / 600  (anchor — bold, owns the line)
Context:  14px / 400  (quiet — same baseline, lighter weight)
```

### Vertical

Elements in adjacent containers that sit on the same vertical line should share a left edge.

Every element's x-position is the sum of every ancestor's left contribution:

```
x = Σ(margin_left + padding_left) for each ancestor from container edge to element
```

In the content area, define two constants that produce named lanes:

```
page_x       = ?   (shell inset — padding on the scroll area)
card_padding = ?   (inner padding on every surface)

Line 0 = page_x                     (toolbar titles, page headings, card borders)
Line 1 = page_x + card_padding      (text inside cards, alerts, tables)
```

Every element's `x` must equal one of these lines. If it doesn't, an ancestor has inconsistent padding.

**Example:** `page_x = 16`, `card_padding = 12`:

```
Toolbar "Usage":           scroll(pl:16) → text                → x = 16 = Line 0 ✓
Page heading:              scroll(pl:16) → text                → x = 16 = Line 0 ✓
Card label "Total tokens": scroll(pl:16) → card(pl:12) → text → x = 28 = Line 1 ✓
Table cell:                scroll(pl:16) → table(px:12) → text → x = 28 = Line 1 ✓
Alert with px-5:           scroll(pl:16) → alert(px:20) → text → x = 36 ✗ broken
```

The alert used `px-5` (20px) instead of `px-3` (12px). One inconsistent padding value creates a third line. Fix: use `card_padding` everywhere.

### Rules

1. Prefer free alignment — share top offset, row height, and line height so baselines match automatically. The trace procedure is for debugging, not designing.
2. Every visible left edge must be on a named lane. If an element sits between lanes, it's a bug.
3. Interactive surfaces (hover fills, active backgrounds) have visible left edges — they're on a lane.
4. `card_padding` is one value used everywhere — every card, alert, table header, and table cell.
5. Elements on the same horizontal line share a font size. Differentiate through weight, not size.
6. Row height is the taller of line-height or tallest child (e.g. an icon). Two rows with the same font size but different icon heights will misalign.
