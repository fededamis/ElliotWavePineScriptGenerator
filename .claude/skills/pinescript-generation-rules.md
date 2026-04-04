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
- **CRITICAL: Pine Script v6 does NOT support semicolons (`;`) as statement separators. Every statement must be on its own line. Never write `a() ; b() ; c()` on one line — write each call on a separate line.**

### PROJECTED PIVOT SUBWAVE COUNTS
When the .wave file contains projected pivots with subwave counts (e.g., "WA (3 sw)"), the Pine Script generation must:
1. Parse the subwave count from the wave name — extract the number in parentheses (e.g., "3" from "WA (3 sw)")
2. Declare a parallel array `pr_swCounts` (primary) and `al_swCounts` (alternate) to store subwave counts (int, default 0)
3. In `makePivotText()` for projected pivots: if the subwave count > 0, append " (N sw)" to the wave name in line 1
4. Pass the subwave count as a parameter to `makePivotText()` so it can format the label correctly

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
  - When visible, render subwave lines as dashed, width=1, using `color.new(wave_color, swLabelTransp)`; render subwave labels at size=size.tiny with wave name only in label text and price+fib in tooltip (see visual-style: LABEL STYLE & CONTENT)
  - `swLabelTransp` is declared as `int swLabelTransp = input.int(55, "Subwave Label Transparency", minval=0, maxval=100, group="Colors")` — applies to all primary and alternate subwave line/label colors. Do NOT hardcode transparency values (e.g. 55, 70) for subwave colors; always reference this variable.
  - **Coincident pivots (e.g. W3.sw5 shares the same timestamp and price as W3): suppress the subwave label entirely — do NOT emit a `label.new()` for it. Instead, inject the subwave Fib into the primary pivot label`s `tooltip`. Detect coincidence structurally: the last entry in each `sw_ts` array always coincides with the parent primary pivot. In the subwave drawing loop: `bool is_coincident = (sli == array.size(sw_ts) - 1)` — skip `label.new()` when true. At the primary pivot `label.new()` call site, add `tooltip = sw_name + "\nFib: " + sw_fib` (use the scalar subwave variables already declared, e.g. `pr_w3sw5_fib`). This is the only fully zoom-independent solution: one label per pivot, zero overlap at any Y-axis scale.**
- A toggle (bool input) called "Show Channel" to show/hide the Elliott Wave base channel; default value is `false`
  - Only draw the channel for impulse (5-wave) structures that have W1, W2, W3 confirmed — skip silently if the count is corrective-only (WA-WB-WC) or W3 has not yet been identified
  - Respects "Show Count" mode: draw primary channel when rendering primary, alternate channel when rendering alternate (if the alternate is an impulse)

---

### CHANNEL DRAWING

The Elliott Wave channel is a two-line parallel channel that frames the impulse structure. Draw it inside the `if barstate.islast` block whenever `showChannel` is true and the count has W1, W2, W3 confirmed.

**Construction (use the individual pivot variables, not the arrays):**

```
// slope defined by the W1→W3 motive line
float ch_slope = (w3_p - w1_p) / float(w3_ts - w1_ts)

int ch_start_ts = w0_ts                   // channel begins at the wave origin
int ch_end_ts   = <last pivot ts>         // last timestamp in the count (historical or projected)

// Motive line: through W1 and W3
float mot_y_start = w1_p + ch_slope * float(ch_start_ts - w1_ts)
float mot_y_end   = w1_p + ch_slope * float(ch_end_ts   - w1_ts)

// Base line: parallel, anchored at W2
float base_y_start = w2_p + ch_slope * float(ch_start_ts - w2_ts)
float base_y_end   = w2_p + ch_slope * float(ch_end_ts   - w2_ts)
```

Replace `w0_p`, `w1_ts`, etc. with the actual variable names for the count being drawn (e.g. `pr_w0_ts`, `pr_w1_p` for primary; `al_w0_ts`, `al_w1_p` for alternate).

For `<last pivot ts>`: use the last timestamp variable declared in the count's data section (e.g. `pr_wc_ts` or `pr_w5_ts` — whichever is the final pivot in that count). Do NOT use `array.size()` or array lookups here; reference the scalar variable directly.

**Line style:**
- `style=STYLE_DASHED`, `width=1`
- Color: apply 50% transparency at draw time — `color.new(PR_CHANNEL_COL, 50)` for primary, `color.new(AL_CHANNEL_COL, 50)` for alternate. **Never** embed transparency in the `input.color()` default (e.g. via `color.rgb(r,g,b,transp)`) — Pine Script v6 does not reliably support transparent defaults there, causing the color to resolve to `na` and the lines to be invisible.
- `xloc=xloc.bar_time` on all `line.new()` calls

**Add both channel lines to `allLines`** so they are cleaned up on each bar.

**Do NOT draw the channel when:**
- `showChannel` is false
- The count does not have W1, W2, W3 variables (corrective-only structure)
- `drawPrimary` / `drawAlternate` is false for that count's rendering branch
