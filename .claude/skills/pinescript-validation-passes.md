---
name: pinescript-validation-passes
description: Pine Script v6 integrated validation scan for Elliott Wave scripts. Run this after generating a Pine Script to check and silently fix all syntax, type safety, logic, coordinate, and array bounds issues.
---

## PINE SCRIPT V6 — INTEGRATED VALIDATION SCAN

Read the generated script once from top to bottom and check all categories below in a single pass.
For any issue found, fix it silently in the script and continue immediately.
Do not output the fix, the reason for the fix, or any before/after comparison.
Do not output anything during this scan except the progress lines specified below.

Output before starting: `⟳ [3/4] Integrated Validation Scan`

---

### A — SYNTAX & COMPILATION
- All functions called with correct argument types and correct number of arguments
- No missing or extra parentheses, brackets, or commas
- All if/for blocks properly indented and closed
- No use of deprecated v5 syntax (e.g. use `array.from()` not `array.new_*`)
- No functions defined inside if/for blocks
- No use of `var` keyword on non-persistent variables inside functions
- No void functions called in expressions
- No series values used where simple values are required (e.g. in input() calls)
- No `bar_index` or `time` referenced inside `array.from()` declarations
- No reserved Pine Script v6 keywords used as variable names
- No return type mismatches in ternary expressions
- No missing `=>` in function definitions
- **FUNCTION PARAMETER TYPES**: Pine Script v6 user-defined functions must NOT use type keywords on parameters. Type keywords (`bool`, `int`, `float`, `string`, `color`, `line`, `label`) are valid for variable declarations but are **invalid** in function signatures:
    isProj(t) => t > today          ✅ correct Pine v6 function syntax
    bool isProj(int t) => t > today  ❌ FORBIDDEN — `bool` return type prefix and `int` parameter type are not valid Pine v6 function syntax
  - Scan every function definition (`name(...) =>`) and remove any type keywords from parameter names and any return type prefix before the function name
  - The return type is inferred automatically by Pine v6 — never declare it explicitly
- No undeclared variables or functions referenced before definition
- **DECLARATION ORDER**: All variables and functions must be declared BEFORE they are used:
    bool enableDragging = input.bool(true)  // ✅ declared first
    label.new(..., interactive=enableDragging)  // ✅ used after declaration
    var bool x = undeclaredVar  ❌ FORBIDDEN — undeclaredVar not yet declared
  - input() calls must come before any variable uses them
  - All identifiers referenced in code must have explicit declarations in scope
  - Check that all function calls reference declared functions, not undefined ones
- **NO POSTFIX IF STATEMENTS**: No statements ending with `if condition` (e.g., `array.push(arr, label.new(...)) if showLabels` is FORBIDDEN). All conditional logic must use proper nested `if` blocks:
  ```
  if condition
      array.push(arr, label.new(...))
  ```
  NOT:
  ```
  array.push(arr, label.new(...)) if condition  ❌ INVALID in Pine v6
  ```

### B — TYPE SAFETY
- Every variable has an explicit type declaration (int, float, string, color, bool, line, label)
- No implicit type coercion between incompatible types
- Function return types are consistent across all branches
- No style constant (line.style_*, label.style_*, size.*, xloc.*, yloc.*, extend.*, text.align_*)
  used as a type keyword in any variable declaration — these are constants, not types
- Style constants stored in variables must use type `string`:
    string myStyle = line.style_solid  ✅
    line.style myStyle = line.style_solid  ❌
- Every line.new() style= argument is one of:
    line.style_solid, line.style_dashed, line.style_dotted,
    line.style_arrow_left, line.style_arrow_right, line.style_arrow_both
- Every label.new() style= argument is one of:
    label.style_none, label.style_label_up, label.style_label_down,
    label.style_label_left, label.style_label_right, label.style_label_center,
    label.style_label_lower_left, label.style_label_lower_right,
    label.style_label_upper_left, label.style_label_upper_right,
    label.style_arrowup, label.style_arrowdown, label.style_cross,
    label.style_xcross, label.style_circle, label.style_triangleup,
    label.style_triangledown, label.style_flag, label.style_square, label.style_diamond
- Every label.new() style= argument must be a literal constant passed directly -- never a string variable:
    label.new(... style=label.style_label_left ...)   ✅
    string s = label.style_label_left; label.new(... style=s ...)  ❌ (causes silent label failure in v6)
  If conditional style is needed, use if/else with two separate label.new() calls, each passing a literal constant
- **VALID label.new() PARAMETERS ONLY**: Only use parameters that exist in Pine Script v6's label.new() function:
    ✅ VALID: xloc, x, y, text, color, textcolor, style, textalign, size, tooltip
    ❌ FORBIDDEN: interactive, draggable, onclick, editable, or any non-existent parameter
  - Common mistakes: `interactive=enableDragging` does NOT exist in Pine v6
  - Check Pine Script v6 documentation for exact parameter names and types
  - All arguments must match the official function signature
- **ARRAY TYPE MATCHING**: Lines and labels MUST be pushed to their correct array types:
    array<line> pr_lines = array.new<line>()
    array<label> pr_labels = array.new<label>()
    array.push(pr_lines, line.new(...))   ✅ line into line array
    array.push(pr_labels, label.new(...))   ✅ label into label array
    array.push(pr_lines, label.new(...))   ❌ FORBIDDEN — type mismatch
  - Check every array.push() call: verify the array variable type matches the element type
  - Lines go only into `pr_lines`, `al_lines`, `*_inval_lines`, `*_target_lines`
  - Labels go only into `pr_labels`, `al_labels`, `*_inval_labels`, `*_target_labels`, `legend_labels`

### C — LOGIC & RUNTIME
- Array sizes for times, prices, and labels are equal for both counts
- Array index access is always within bounds (no off-by-one errors)
- All pivot timestamps are valid and in chronological order
- Historical pivots end on or before today's date; every projected pivot date is strictly after today's date -- if any projected timestamp is <= today, shift it forward until it is after today
- Invalidation and target levels are on the correct side of current price
- "Alternate Only" mode renders alternate lines as solid and full opacity — not dashed or transparent
- Projected lines are dotted, distinct from solid historical (primary) and dashed historical (alternate)
- Upward motive segments use color.aqua, downward motive use color.fuchsia, corrective use color.orange
- All drawing calls (line.new, label.new) are inside barstate.islast
- max_lines_count and max_labels_count in indicator() are high enough for all drawn objects
- No division by zero or operations on na values
- xloc.bar_time used consistently wherever timestamps are passed as x coordinates
- Every pivot label.new() call is guarded by the showLabels bool input -- if showLabels is false, no pivot labels are created; if showLabels is true, all pivot labels for the active count(s) are created and visible
- Verify that at least one label.new() call for pivot labels exists in the script and is reachable when showLabels is true -- if no pivot labels are drawn at all, add them

### D — COORDINATE & SCALE STABILITY
- All line.new() and label.new() calls use xloc.bar_time — never xloc.bar_index
- All timestamp x values are in milliseconds via timestamp(year, month, day) integer arguments — `timestamp()` already returns ms, so it must NEVER be multiplied by 1000; scan every timestamp declaration and remove any `* 1000` suffix
- All label.new() calls use yloc.price — never yloc.abovebar or yloc.belowbar
- All label.new() calls set size=size.auto
- All label.new() calls set textalign=text.align_left
- No line.new() uses extend.right or extend.left unless it is an invalidation or target level
- Horizontal lines (y1 == y2) use line.style_dashed or line.style_dotted — not line.style_solid
- All line.new() y1/y2 values are fixed floats — never open, high, low, close, or other series
- No drawing object coordinates set or updated outside of the barstate.islast block
- "Today" anchor uses time (current bar open time) — never timenow
- Projected pivot timestamps use whole trading-day multiples added to last bar's time value
- All stale line/label objects are deleted via while/array.pop() before redrawing
- Add this comment directly above the barstate.islast block:
  // All drawing objects use xloc.bar_time + fixed float prices for scale/zoom stability

### E — ARRAY BOUNDS SAFETY
- Every array.get(), array.set(), array.remove(), array.pop(), array.shift(),
  array.first(), or array.last() call is preceded by array.size(arr) > 0 guard
- Every `for i = 0 to array.size(arr) - 1` loop is guarded with `if array.size(arr) > 0`
- All shared loops over parallel arrays assert equal sizes before iteration:
  // assert: array.size(times) == array.size(prices) == array.size(labels)
- Delete-and-redraw loops use the safe while pattern:
    while array.size(linesArr) > 0
        line.delete(array.pop(linesArr))
- array.size(arr) - 1 is never used as a direct index without first confirming array.size(arr) > 0
- Pivot arrays for both counts are fully initialized (even if empty) before any rendering code runs
- Add this comment directly above any array access:
  // array size checked before access

---

Output after completing: `✓ [3/4] Integrated Validation Scan — complete`
