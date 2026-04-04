# Elliott Wave PineScript Generator — Improvement Ideas

## 1. Analysis Depth & Wave Theory

**1.1. Multi-Degree Wave Labeling**
Add a "Wave Degree" input to the prompt (Grand Supercycle, Supercycle, Cycle, Primary, Intermediate, Minor). The generated script would display degree labels using standard EW notation (I/II/III vs 1/2/3 vs (i)/(ii)/(iii)).

**1.2. Diagonal Triangle Detection**
Explicitly detect leading/ending diagonals (overlapping waves in wedge formations). These follow different rules (W4 *can* overlap W1) and need a different visual treatment — converging trendlines instead of standard wave lines.

**1.3. Corrective Pattern Labels**
Beyond A-B-C, explicitly classify the correction type in the label: Zigzag, Flat, Expanded Flat, Running Flat, Triangle (contracting/expanding), Double/Triple Three. Each gets its own visual style.

**1.4. Wave Degree Nesting Panel**
A toggle in the Pine Script that switches between showing just primary waves, primary + subwaves, or subwaves only — instead of just a single "Show Subwaves" toggle.

---

## 2. New Pine Script Visual Features

**2.1. Channel Drawing** ✅
Auto-draw the Elliott Wave channel: a parallel channel connecting W1–W3 endpoints, with W2 and W4 inside it. This is a core EW tool currently missing from the output.

**2.2. RSI Divergence Overlay**
Add an optional panel (or inline label) showing whether RSI confirms each motive wave (momentum divergence at W5 is a classic termination signal).

**2.3. Volume Bars per Wave**
Annotate average volume for each wave segment directly in the label — Wave 3 should have the highest volume; a wave with lower volume than expected flags as weaker.

**2.4. Fibonacci Grid**
Add a toggleable Fibonacci retracement grid drawn from W0 to the current wave's high/low, showing all key levels (23.6%, 38.2%, 50%, 61.8%, 78.6%) as horizontal lines — similar to TradingView's built-in Fib tool but auto-anchored to the wave count.

**2.5. Price Alert Levels Export**
Append a comment block at the bottom of the Pine Script listing all key prices (invalidation levels, targets, next projected pivots) formatted as `// ALERT: $XXX.XX — Primary W5 target`. Users can copy these into TradingView alerts.

---

## 3. Prompt & Workflow Improvements

**3.1. Multi-Ticker Batch Mode**
Ask the user upfront if they want to analyze one ticker or a list. If a list, loop through each ticker, generate one `.pine` file per ticker, and output a summary table of all counts and targets at the end.

**3.2. Timeframe Override Input**
Currently timeframe is auto-selected (daily vs weekly). Add an explicit optional input: "Force timeframe? (auto / daily / weekly / 4H)". Advanced users often want to run the same ticker at multiple timeframes.

**3.3. Recount Specific Wave**
Add a command like "recount from W3" — instead of redoing the entire analysis, re-anchor the count from a specific confirmed pivot the user trusts, and only re-derive from there forward. Saves time and API calls.

**3.4. Cache Versioning**
The `.wave` file currently has no version header. Add a schema version line and a `generated_at` timestamp. This prevents stale caches from silently producing wrong scripts when the methodology is updated.

**3.5. Confidence Delta Reporting**
When loading from cache and regenerating a script, compare the cached confidence % to what the current market data would imply (fetch latest bars, check if any invalidation levels have been breached) and flag if the count may be stale.

---

## 4. Script Inputs & Interactivity

**4.1. Custom Color Scheme Input** ✅
Let users override the aqua/fuchsia/orange palette via color picker inputs in the script. Useful for people with color blindness or dark/light theme preferences.

**4.2. Wave Duration Statistics Table**
A small on-chart table showing the bar duration of each confirmed wave (e.g. "W1: 12 bars, W2: 5 bars, W3: 34 bars..."). Helps users gauge whether projected durations are proportionate.

**4.3. Projected Path Cone**
Instead of a single projected line per wave, draw a cone (upper/lower bound) based on the Fibonacci extension range. For example, W5 projected between 1.0x and 1.618x of W1 — drawn as two dotted lines with fill between them.

**4.4. "What Would Invalidate" Info Box**
A persistent on-chart box (toggleable) summarizing: current active count, next expected move direction, exact invalidation price, and what price action would confirm the count. This is the most actionable info for a trader.

---

## 5. Output & Delivery

**5.1. Markdown Report File**
After writing the `.pine` file, also write a `[TICKER] [START DATE].md` analysis report with: wave count table, key levels, confidence rationale, and a plain-English summary of the expected path. Useful for sharing or review without opening TradingView.

**5.2. JSON Export**
Write a `[TICKER] [START DATE].json` file with the pivot table in structured format — useful for programmatic use, backtesting pipelines, or feeding into other tools.

**5.3. Changelog on Recount**
When the user triggers a recount ("redo"), diff the new wave count against the cached one and append a `// CHANGES:` block to the `.pine` file noting which pivots shifted and by how much.

---

## Priority Recommendation

Ranked by impact:

1. **Wave channel drawing** — fundamental EW tool, visually immediately useful
2. **Corrective pattern classification** — makes the alternate count much more actionable
3. **Projected path cone** — single dotted line feels overconfident; a range is more honest
4. **Markdown report output** — easy to implement, very useful for review/sharing
5. **Multi-ticker batch mode** — high leverage for users who scan many instruments

---

## 6. Token Budget / Context Management

The current pipeline hits the token budget limit because each subagent dumps its full output back into the main conversation context. By the time the validation subagent returns the corrected Pine Script (~500 lines), the context is full and `create_file` never executes.

**6.1. Subagents write their own output files** ✅
The highest-impact fix. Each subagent should call `create_file` or `replace_string_in_file` directly and only return a short status string to the main agent (e.g. `"Wrote 490 lines to ETHUSD 2022-06-01.pine — OK"`). The main context never needs to hold the full generated code.

**6.2. Collapse subagent calls**
Currently 4 subagents run sequentially (Call A, Call B, Pine Script gen, Validation). Reduce to 2:
- **Combined EW analysis subagent**: runs both primary and alternate counts, writes the `.wave` file itself, returns only the pivot summary table.
- **Combined gen+validation subagent**: generates and validates the Pine Script in one pass, writes the `.pine` file itself, returns only a line count and any warnings.

**6.3. Pass wave data via file reference, not in-prompt** ✅
The generation subagent currently receives the full wave pivot table injected into its prompt. Instead, instruct it to read the `.wave` cache file from disk. This removes one large block from the prompt payload.

**6.4. Validation subagent reads from disk** ✅
The validation subagent currently receives the full Pine Script as prompt text. Instead, give it the file path and have it read the file, apply fixes in-place using `replace_string_in_file`, and return only a diff summary (which rules triggered and what was fixed).

**6.5. Draft file pattern**
Write a `.pine.draft` file after generation, before validation. The validation subagent reads the draft, writes the final `.pine`. If the session times out between the two steps, the draft survives and the next session can skip straight to validation without regenerating.
