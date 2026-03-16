---
name: pinescript-generation-rules
description: Pine Script v6 generation constraints, display input rules, and coding standards for Elliott Wave scripts. Apply every rule in this file when generating Pine Script code.
---

### GENERATION CONSTRAINTS
While writing the Pine Script, enforce the following rules in real-time as the code is written --
do not generate code that violates any of these, so they require no fix cycle during validation:

- Never use style constants (line.style_*, label.style_*, size.*, xloc.*, yloc.*, extend.*, text.align_*)
  as type keywords in variable declarations -- declare such variables as type `string`
- Explicitly declare a type for every variable (int, float, string, color, bool, line, label)
- **CRITICAL: All user-defined functions (any block ending with `=>`) MUST be declared at the top level of the script — before any `if`, `for`, or `while` block, including `if barstate.islast`. Pine Script v6 does not allow function definitions inside any conditional or loop block; doing so causes a compile-time "Syntax error at input" on the parameter type keyword. Define all helper functions immediately after the `var` declarations and before the first `if barstate.islast` block.**
- Place all line.new() and label.new() calls inside barstate.islast only
- Set max_lines_count and max_labels_count in indicator() high enough for all drawn objects
- **Use xloc.bar_time for all line.new() calls to preserve scale/zoom stability**
- **For label.new() calls: use xloc.bar_time with timestamp-anchored coordinates; timestamps from past dates may fail silently if not aligned to exact bar boundaries, so ensure all label x-coordinates use valid bar open times or consolidate labels into fixed on-chart info boxes with xloc.bar_index**
- Convert all hardcoded dates with timestamp(year, month, day) integer arguments -- never string literals
- **CRITICAL: `timestamp()` already returns milliseconds in Pine Script v6 -- NEVER multiply its result by 1000. Doing so produces year ~54000 timestamps that push all drawings off-screen. Correct: `int t = timestamp(2024, 1, 15)`. Forbidden: `int t = timestamp(2024, 1, 15) * 1000`**
- Use yloc.price explicitly on all analytical label.new() calls -- never yloc.abovebar or yloc.belowbar
- **For all label size, textalign, and style direction constraints — see `pinescript-visual-style.md` (authoritative source)**
- **Label Y-offset arrays: declare a parallel float array `pr_yoffs` (primary) and `al_yoffs` (alternate) alongside the pivot price arrays. Each entry defaults to 0.0. For any two pivots whose prices are within ~5% of the price range apart AND whose timestamps are within 8 weekly bars (or 40 daily bars) of each other, set non-zero offsets to separate their labels: push the high pivot's label up (positive offset) and the low pivot's label down (negative offset) by approximately 3% of the total price range. Apply the offset as `y = price + yoff` in every label.new() call. The offset does NOT move the wave line endpoint -- only the label box.**
- Use line.style_dashed or line.style_dotted for all horizontal lines (y1 == y2)
- Guard every array access with array.size(arr) > 0 before access
- Use while array.size(arr) > 0 / line.delete(array.pop(arr)) pattern for all delete-and-redraw loops
- Use time (not timenow) as the "today" anchor timestamp
- Keep inline comments minimal: one short comment per logical section only -- no explanatory prose, block headers, or section dividers
- **Use str.tostring(value, "#.##") for float formatting, not str.format() which doesn't support standard format specifiers in Pine v6**

---

### DISPLAY INPUTS
- A dropdown input called "Show Count" with three options: "Primary Only", "Both", "Alternate Only" -- default selected value is "Primary Only"
  - "Primary Only": renders only the primary count, solid lines, full opacity
  - "Both": renders both counts simultaneously, alternate as dashed/transparent, primary as solid
  - "Alternate Only": renders only the alternate count, displayed as solid lines at full opacity (not dashed/transparent) so it is easy to read in isolation
- A toggle (bool input) called "Show Invalidation Levels" to show/hide the invalidation lines and labels
- A toggle (bool input) called "Show Targets" to show/hide the target lines and labels
- A toggle (bool input) called "Show Legend" to show/hide the legend
  - Each count row in the legend label must include its % confidence value (e.g. "Primary Count (65%)" and "Alternate Count (30%)") -- read these values from the `.wave` file's `PRIMARY COUNT (X% confidence)` and `ALTERNATE COUNT (X% confidence)` headers and embed them as literals in the label text
- A dropdown input called "Legend Position" with four options: "Top Right", "Bottom Right", "Top Left", "Bottom Left" -- default selected value is "Bottom Right"
  - This controls where the legend cluster is placed on the chart:
    - "Top Right": anchor to the right of today, near the highest pivot price
    - "Bottom Right": anchor to the right of today, near the lowest pivot price
    - "Top Left": anchor before the first pivot timestamp, near the highest pivot price
    - "Bottom Left": anchor before the first pivot timestamp, near the lowest pivot price
  - Implement as: compute `int leg_x` and `float leg_base` from the selected option using if/else, then use those for all legend label x/y coordinates
  - `leg_x` for Right options = `today + 86400000 * 7`; for Left options = first pivot timestamp minus `86400000 * 30`
  - `leg_base` for Top options = highest pivot price in the script; for Bottom options = lowest pivot price in the script
  - Stack legend rows by subtracting `step` increments from `leg_base` (same step formula as before)
- A toggle (bool input) called "Show Projections" to show/hide the projected future wave lines
  - The transition point between historical and projected is the most recent confirmed pivot, anchored to today's date and price
  - All projected pivot dates MUST be strictly after today's date — no projected pivot may fall on or before today
  - Projected sequences must be structurally complete: for A-B-C corrections project all three legs (WA, WB, WC); for impulse sequences project through the full next wave. Never terminate a projection mid-structure (e.g. stopping at WB without WC)
  - The last projected pivot must be dated at least 60 calendar days after today. If the final projected pivot falls within 60 days, extend the projection sequence by adding the next wave in the structure
  - Do NOT add horizontal flat extension lines as a substitute for a missing projected pivot — flat lines do not represent any Elliott Wave structure
- A toggle (bool input) called "Show Labels" to show/hide all pivot labels (does not affect invalidation, target, or legend labels)
- A toggle (bool input) called "Show Subwaves" to show/hide the subwave lines and labels (primary count only); default value is `true`
  - When hidden, suppress all subwave lines, subwave pivot labels, and the subwave legend row
  - When visible, render subwave lines as dashed, width=1, at 55% transparency relative to the primary count color; render subwave labels at size=size.tiny
