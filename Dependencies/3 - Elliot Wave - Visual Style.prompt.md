### COLOR SCHEME

#### PRIMARY COUNT (solid lines, high-contrast colors)
- Motive/impulse upward wave segments (Waves 1, 3, 5): color.aqua (#00FFFF)
- Motive/impulse downward wave segments (Waves 1, 3, 5 in a downtrend): color.fuchsia (#FF00FF)
- Corrective wave segments (Waves 2, 4, A, B, C and all corrective structures): color.orange (#FF8000)
- Historical lines: line.style_solid, width=2
- Projected future lines: line.style_dotted, width=2, same aqua/fuchsia/orange as wave type
- Invalidation level line and label: color.red
- Target level line and label: color.lime

#### ALTERNATE COUNT (dashed lines, muted/desaturated palette to visually recede behind primary)
- Motive/impulse upward wave segments: #5599AA (desaturated steel blue — clearly distinct from primary aqua)
- Motive/impulse downward wave segments: #AA55AA (muted violet — clearly distinct from primary fuchsia)
- Corrective wave segments: #AA7733 (muted amber — clearly distinct from primary orange)
- Historical lines: line.style_dashed, width=2
- Projected future lines: line.style_dotted, width=2, same muted palette as wave type
- Invalidation level line and label: #CC4444 (darker red, distinct from primary red)
- Target level line and label: #448844 (darker green, distinct from primary lime)

#### LABEL BACKGROUND DISTINCTION
- Primary count pivot labels: full-opacity background using the wave's primary color
- Alternate count pivot labels: semi-transparent background (color.new(wave_color, 50)) so they visually recede
- Alternate count label text prefix: prepend "(A)" to the wave identifier on Line 1 (e.g. "(A) WB2", "(A) W3") so the count is identifiable even without color context

### LABEL STYLE & CONTENT
- **For pivot labels at specific points: use size=size.small to ensure labels are clearly readable**
- **For other labels (invalidation, target, legend): use size=size.small as well for consistency**
- Every pivot label must display the following information on separate lines within the label text:
  - Line 1: Wave identifier — primary count uses bare label (e.g. "W1", "WA"); alternate count prepends "(A)" (e.g. "(A) W1", "(A) WA")
  - Line 2: Price value formatted with str.tostring(price, "#.##")
  - Line 3: Fibonacci level (e.g. "Fib:61.8%") — use abbreviated format to save space
- Primary count pivot labels: use full-opacity label background color matching the wave color
- Alternate count pivot labels: use color.new(wave_color, 50) for a semi-transparent background so they visually recede behind primary labels
- Invalidation level labels must display:
  - Single line: "Primary Inval" or "Alt Inval" (abbreviated for space)
- Target level labels must display:
  - Single line: "Primary Target" or "Alt Target" (abbreviated for space)
- **All pivot label.new() calls must set textalign to text.align_center to center-align text at the pivot point**
- **All pivot label.new() calls must set style to label.style_label_down (for highs) or label.style_label_up (for lows) so the label floats above/below the pin without obscuring price action**
- **For labels at level lines (invalidation, targets): use style=label.style_label_down or label.style_label_up depending on chart position**