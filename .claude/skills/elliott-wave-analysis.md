---
name: elliott-wave-analysis
description: Run a full Elliott Wave analysis on a ticker. Fetches real OHLC data from Yahoo Finance, applies Elliott Wave rules/guidelines, identifies primary and alternate wave counts, and outputs a compact pivot table with projections.
---

Perform a complete Elliott Wave analysis using the methodology below. The user will supply: **TICKER**, **START DATE**, **TIMEFRAME** (weekly/daily/4H), and optionally an **END DATE** (default: today).

### ELLIOTT WAVE METHODOLOGY

**NO SCRIPTS RULE: Do not write or execute any scripts (Python, PowerShell, Bash, or any shell language) for analysis, data fetching, or calculations at any point. All data fetching must use the WebFetch tool directly against the Yahoo Finance API (or `run_in_terminal` in Copilot mode solely to download and write the cache file silently). All calculations (Fibonacci, pivot identification, retracement percentages) must be performed in-context using the model's own reasoning. Running scripts to read back bar ranges, echo price rows, or produce any intermediate output to the terminal is forbidden — it produces chat output and wastes tokens. The cache file is written once; read it with `read_file` if needed, not `run_in_terminal`.**

---

**Step 1 — Identify the Trend Structure**
- Determine whether price is in an impulse phase (5-wave motive) or corrective phase (3-wave or complex)
- Identify the dominant trend direction from the START DATE
- Mark the most obvious swing highs and swing lows as candidate wave pivots

**Step 1.5 — Declare and Lock Wave Degree**

Before labeling any pivots, determine the working degree using this calibration table:

| Degree        | Duration              | Typical Price Range |
|---------------|-----------------------|---------------------|
| Supercycle    | 8–40 years            | 100–1000%+          |
| Cycle         | 1–8 years             | 20–200%             |
| Primary       | 3 months–2 years      | 10–60%              |
| Intermediate  | 2 weeks–9 months      | 5–25%               |
| Minor         | 1 week–8 weeks        | 2–12%               |

**Degree Selection Algorithm (apply in order):**
1. Measure total span: START DATE → today in calendar days.
2. Compute price range: `(highest High − lowest Low) ÷ lowest Low × 100%`.
3. Select the degree whose duration AND price range both fit the table.
4. When duration and price range disagree, **prefer the price range** — it is the more reliable degree indicator.
5. When two adjacent degrees are equally plausible, build provisional pivot sets for both and count Fibonacci confluences at each. Select the degree with the higher confluence score. If still tied, prefer the higher (larger) degree.
6. **LOCK the degree.** All pivots in both the primary and alternate count must be labeled at this degree. A pivot that fits only a different degree is a mislabeling — revise the surrounding pivots. Degree changes mid-count are forbidden.

Output before Step 2: `Degree: [Name] ([N]-day span, [X]% price range)`

**Degree Proportion Rules (absolute — same force as the three EW rules):**
- Each wave in a count must span between 5% and 85% of the total sequence duration. A wave at ≤4% or ≥86% of total sequence time belongs to a different degree — revise the surrounding pivots.
- No wave may be more than 10× the duration of any sibling wave in the same count.
- The alternate count must use the same locked degree. Pivots may be re-labeled under different wave names but the degree itself cannot change.

**Step 2 — Apply Elliott Wave Rules (these are absolute and cannot be violated)**
- Wave 2 never retraces more than 100% of Wave 1
- Wave 3 is never the shortest among Waves 1, 3, and 5
- Wave 4 never overlaps Wave 1's price territory (except in diagonals)

**Step 3 — Apply Elliott Wave Guidelines (these are tendencies, not rules)**
- Wave 2 typically retraces 50%–61.8% of Wave 1
- Wave 3 is typically the longest and strongest wave, often 1.618x Wave 1
- Wave 4 typically retraces 38.2%–50% of Wave 3
- Wave 5 is often equal to Wave 1, or 0.618x Wave 3
- In corrections, Wave C is often equal to Wave A, or 1.618x Wave A
- Alternation: if Wave 2 is sharp, expect Wave 4 to be flat, and vice versa

**Step 4 — Use Fibonacci to Confirm Pivot Levels**
- Confirm each wave pivot using Fibonacci retracement and extension levels
- Prefer pivots that align with key Fibonacci levels (23.6%, 38.2%, 50%, 61.8%, 78.6%, 100%, 127.2%, 161.8%)
- Pivots that cluster near multiple Fibonacci levels are stronger candidates
- **PRIMARY WAVE FIB COMPUTATION RULE: Every Fib % in the primary and alternate wave tables MUST be computed from actual prices — never selected from a standard list by memory. The formula depends on the wave's role:**
  - **Anchor row** (W0 in motive; first wave of a corrective sequence e.g. WA): always `--`. No computation.
  - **Retracement rows** (W2, W4 in motive; corrective waves that retrace the prior leg): `|wave_end − wave_start| ÷ |prior_motive_leg_end − prior_motive_leg_start| × 100`. For W2: prior leg = W1 (W0→W1). For W4: prior leg = W3 (W2→W3).
  - **Extension rows** (W1 beyond W0 anchor, W3, W5 in motive; WB, WC in corrective): `|wave_end − wave_start| ÷ |W1_range| × 100`, where W1_range = |W1_end − W0|. For corrective extensions (WB, WC): use `|wave_end − wave_start| ÷ |WA_range| × 100`.
  - Round to one decimal place. A value that coincidentally matches a standard Fib level is valid only if derived by this formula.
  - Apply the **EXTENSION CEILING GUARD** (see below) to every extension row.

**EXTENSION CEILING GUARD (applies to every extension row at every degree — primary waves and subwaves):**
Extensions above 261.8% are uncommon; above 423.6% they are extremely rare and almost always indicate a misidentified anchor pivot. Apply this procedure to any extension row (W3, W5, WB, WC at the primary level; sw3, sw5, swc at the subwave level) whose computed value exceeds 261.8%:
1. **Re-examine the anchor.** For primary waves, verify the W1 range (W0→W1) or WA range was not a micro-swing. For subwaves, walk back through the parent wave's interior and find a more significant anchor pivot (deeper sw1 / swa).
2. **Recompute.** If the result falls at a recognized Fibonacci extension (261.8%, 361.8%, or 423.6%), it is structurally permissible — accept the pivot but mark it `⚠(ext >261.8%)` in the Subwave confirmation line and ensure the alternate count reflects a more conservative anchor interpretation. Extensions in this range are uncommon; the `⚠` signals reduced confidence, not a hard error.
3. **If still above 423.6% after step 1–2:** the anchor is misidentified and the current pivot set is invalid. For primary waves, select a different W0/W1 or WA endpoint and recount. For subwaves, mark the wave `✗(no valid anchor)` in the Subwave confirmation line and omit its rows from the output.

**Step 4.5 — Identify Subwaves (one level of depth only)**

**DEPTH LIMIT: Subwave identification is strictly one level deep. You may only subdivide primary waves (W1, W2, W3, W4, W5) into their subwaves. You must NOT subdivide a subwave into its own sub-subwaves (e.g. W1.sw5 must not have its own sw1–sw5). Any second-level nesting is invalid and must not appear in the output.**

Apply the PIVOT ACCEPTANCE GATE to all subwave pivots (same rules as primary pivots) using the already-fetched Yahoo Finance data — do not invent prices.

**LONG-WAVE FETCH RULE: The compact table built in the DATA SOURCING step covers the full analysis range (START DATE → today). Use it for all subwave identification — do not issue additional Yahoo Finance fetches for individual wave interiors. "No clear pivot" is not a valid skip reason — if the data exists, a swing extreme exists. The only valid skip reason is `✗(insuf)` for waves with fewer bars than the minimum.**

**MANDATORY COVERAGE RULE: Attempt subwave identification for every primary wave (W1, W2, W3, W4, W5). The Subwave note at the bottom of the output must list every primary wave with a result symbol: ✓ (confirmed), ⚠ (partial), or ✗(reason) (skipped/insufficient). No prose justification — symbol and short reason only. `✗` is only valid for waves below the bar minimum — it is NOT valid for long waves where data exists.**

**SUBWAVE TIMEFRAME — DEGREE-TIMEFRAME HIERARCHY: All subwave identification uses the single persistent compact table already built in the DATA SOURCING step — no additional fetches are issued. The table was fetched at the subwave chart interval for the locked degree:**
| Degree | Primary chart | Subwave chart (fetch interval) |
|---|---|---|
| Grand Supercycle | Monthly | Weekly (`1wk`) |
| Supercycle | Weekly | Daily (`1d`) |
| Cycle | Weekly | Daily (`1d`) |
| Primary | Daily | 4H (`1h`) |

The bar minimum thresholds below (motive ≥ 5 bars, corrective ≥ 3 bars) apply to the subwave chart interval reflected in the compact table.

For **each motive wave** (W1, W3, W5) in the primary count that spans at least 5 bars on the subwave chart:
- Identify the 5 internal subwave pivots (sw1 through sw5) on the subwave chart
- Label them as `W1.sw1`, `W1.sw2` … `W1.sw5` (or `W3.sw1` … `W3.sw5`, etc.)
- **5-SUBWAVE MANDATE: Every motive wave (W1, W3, W5) that spans ≥ 5 bars on the subwave chart MUST resolve to all 5 subwaves (sw1–sw5). Partial resolution (e.g. only sw1–sw3 found) is NOT acceptable — if fewer than 5 subwaves can be identified within the current primary pivot boundaries, the primary count is wrong. STOP. Return to Step 5, try a different endpoint for that parent wave (a later high or lower low), and re-run subwave verification. Only accept the primary count when all 5 subwaves of every qualifying motive wave are confirmed. The `⚠(sw4/sw5 insufficient bars — absorbed into parent)` warning is FORBIDDEN as a final result for any wave spanning ≥ 5 bars.**
- **SHORT-WAVE ±1 FALLBACK: If a motive wave spans fewer than 20 bars on the subwave chart and no interior swing passes the ±2 neighboring-bar check, relax the neighboring-bar check to ±1 for that wave's subwaves only. If at least 3 of the 5 required pivots pass the ±1 gate, output those rows and mark the wave `⚠(±1 gate, N/5 subwaves)` in the Subwave confirmation line. If even the ±1 gate yields fewer than 3 pivots after relaxation, STOP — do not accept this primary count; return to Step 5 and try a different endpoint for the parent wave. The `⚠` partial-subwave warning is only permitted as a last resort when all alternative endpoints have been exhausted.**

- **MOTIVE SUBWAVE ACCEPTANCE CHECKLIST — compute and verify every item before accepting the pivot set. Any failing item is a hard STOP; return to Step 5 and try a different endpoint for the parent wave:**
  1. **Cache boundary**: every sw1–sw5 date ≤ the cache's last bar date. Any pivot dated after the cache last bar is UNVERIFIED — reject the pivot set; re-fetch fresh data or move the parent wave terminal to within the cache window.
  2. **sw2 retrace %**: `(sw1_high − sw2_low) / (sw1_high − W_origin) × 100`. Must be ≤ 100%. If > 100%, sw2 pushed beyond the parent wave's origin — reject.
  3. **Impulse magnitudes**: compute `sw1_range = |sw1 − W_origin|`, `sw3_range = |sw3 − sw2|`, `sw5_range = |sw5 − sw4|`. sw3 must be strictly greater than BOTH sw1_range and sw5_range. If sw3 is the smallest of the three, reject.
  4. **sw4 / sw1 no-overlap**: for a bullish parent, `sw4_low > sw1_high`. For a bearish parent, `sw4_high < sw1_low`. If there is overlap, reject (unless a diagonal structure is explicitly declared).
  4b. **sw4 corrective direction**: for a bullish parent, `sw4_low < sw3_high` (sw4 must retrace below sw3's peak — a "Low" priced above the preceding "High" is not a corrective trough). For a bearish parent, `sw4_high > sw3_low`. If this fails, the sw4 pivot is misidentified — reject.
  5. **Proportion floor**: each swN duration as % of parent wave total duration must be > 4%. Compute each; if any is ≤ 4%, reject.
  6. **Proportion ceiling**: each swN duration must be < 86% of parent wave total duration. If any is ≥ 86%, reject.
  7. **Sibling ratio**: `max(sw1…sw5 durations) / min(sw1…sw5 durations)` must be < 10. If ≥ 10, reject.

- **Apply the three absolute EW rules at the subwave level — these are not optional:**
  1. sw2 never retraces more than 100% of sw1 (sw2's extreme cannot go beyond the start of sw1, i.e. the parent wave's origin)
  2. sw3 is never the shortest among sw1, sw3, and sw5
  3. sw4 never overlaps sw1's price territory
- **SUBWAVE BOUNDARY RULE: Every subwave pivot (sw1–sw5) must fall strictly within the time and price span of its parent wave. A subwave whose date or price lies beyond the parent wave's endpoint belongs to the next parent wave — remove it. If removal leaves fewer than the required pivots (5 for motive, 3 for corrective), the parent wave boundaries are wrong. STOP. Return to Step 5, re-count the parent waves with a corrected endpoint, and re-run subwave verification. Do not retain partial subwave sets or emit a `⚠` warning as a substitute for re-counting.**
- **SW3-NOT-AT-PARENT-TOP RULE: sw3 of a motive wave must NEVER be set to the same price/date as the parent wave's terminal pivot (the wave's top for a bullish wave, or bottom for a bearish wave). If the highest swing within the parent coincides with the parent terminal, that swing IS sw5 — search within the interior of the wave for an earlier local extreme to serve as sw3, with a pullback (sw4) and final thrust (sw5) completing the structure at the terminal. Setting sw3 = parent terminal and then finding no room for sw4/sw5 is INVALID — it means an interior pivot was missed. Walk the subwave chart interval bars between sw2 and the parent terminal, apply the ±2 neighboring-bar gate, and find the correct sw3 peak before the final run.**
- **If any of these three rules is violated, the PRIMARY COUNT IS INVALIDATED. The three absolute EW rules apply at every wave degree — a violation at the subwave level means the primary pivot labeling is wrong (the parent wave's start or end was mislabeled). STOP. Do not output the current primary count. Return to Step 5, select a different pivot set, and re-run subwave verification. Repeat until a primary count is found whose subwaves all satisfy the three rules.**

For **each corrective wave** (W2, W4, WA, WB, WC) in the primary count that spans at least 3 bars on the subwave chart:
- Identify the 3 internal subwave pivots (swa, swb, swc) on the subwave chart
- Label them as `W2.swa`, `W2.swb`, `W2.swc` (etc.)
- **SWA-NOT-AT-PARENT-TERMINAL RULE: swa of a corrective wave must NEVER be set to the same price/date as the parent wave's terminal pivot (the correction's end: the wave low for a bearish correction, the wave high for a bullish correction). swa is the FIRST interior leg — the initial move away from the correction's origin toward the terminal. If the deepest swing within the corrective wave coincides with the parent terminal, that swing IS swc — search within the interior of the wave for an earlier local extreme to serve as swa, with a bounce (swb) and final leg (swc) completing the structure at the terminal. Setting swa = parent terminal leaves the ABC structurally degenerate (zero-length swb and swc) and is INVALID.**
- **SUBWAVE FIB COMPUTATION RULE (applies to every subwave row in every wave, motive and corrective): Every Fib % value in the output table MUST be computed from actual prices — never selected from a standard list (38.2%, 61.8%, 100%, 161.8%, etc.) by memory or pattern-matching. The formula depends on the subwave's role:**
  - **Anchor row** (sw1 in motive; swa in corrective): always `--`. No computation.
  - **Retracement rows** (sw2, sw4 in motive; swb in corrective): `|pivot_price − prev_pivot_price| ÷ |anchor_end − anchor_start| × 100`, where the anchor is the immediately preceding leg (sw1 for sw2; sw3 for sw4; swa for swb).
  - **Extension/continuation rows** (sw3, sw5 in motive; swc in corrective): `|pivot_price − prev_pivot_price| ÷ |anchor_end − anchor_start| × 100`, where the anchor is sw1 (for sw3 and sw5 in motive) or swa (for swc in corrective). swc_start is the swb terminus.
  - After computing, round to one decimal place (e.g. `233.7%`, `45.9%`). A result that coincidentally matches a standard Fib level is valid only if it was derived by this formula. Never output a round standard value without showing it was computed. Values above 100% are normal and expected for extension legs.
  - Apply the **EXTENSION CEILING GUARD** (defined in Step 4) to every extension row.
- Identify which corrective pattern applies and verify its internal consistency rules (all are absolute — any violation is INVALID; select different swa/swb/swc pivots):
  - **Zigzag (5-3-5):** swB retraces between 38.2%–78.6% of swA; swC MUST end at or beyond swA (swc low ≤ swa low for a bearish correction, swc high ≥ swa high for a bullish correction); swC is typically equal to swA or 1.618× swA; a swC that fails to reach the swA terminus is a truncated C and is INVALID.
  - **Flat (3-3-5):** swB retraces 85%–105% of swA (swB terminus approaches or slightly exceeds the correction's origin); swC ends near or slightly beyond swA (swc low ≤ swa low ± 5% of swA move for a bearish flat); a swB that retraces less than 85% of swA is not a flat — reclassify as zigzag or triangle.
  - **Running Flat:** Same as flat except swB exceeds the correction's origin (retraces > 100% of swA) and swC fails to reach swA — this is the only valid structure where swC does not reach swA; must be explicitly labeled as running flat in the Subwave confirmation line.
  - **Expanded Flat:** swB exceeds the correction's origin (> 100% of swA); swC ends beyond swA (swc extreme exceeds swa extreme); swC is typically 1.236×–1.618× swA.
  - **Triangle (3-3-3-3-3, five sub-legs a–b–c–d–e):** Each successive leg is shorter than the preceding leg (converging); sub-legs alternate direction; the triangle must fit within converging trendlines; label sub-legs as swa/swb/swc/swd/swe; triangles are only valid for W4 or WB (never W2).
  - **If the pattern type cannot be determined** (swB retracement does not fit any category above): the corrective structure is ambiguous — use the pivot set that most closely satisfies the flat rules and note `⚠(ambiguous flat/zigzag)` in the Subwave confirmation line.

If a wave spans fewer bars than required on the subwave chart (motive < 5 bars, corrective < 3 bars), mark it as `✗(insuf)` in the Subwave note — do not silently omit it.

**Subwave Confidence Contribution:**
- Each motive wave whose 5 subwaves all pass the acceptance gate and satisfy EW rules: +3% to primary count confidence
- Each corrective wave whose a-b-c subwaves all pass the acceptance gate and satisfy EW rules: +2% to primary count confidence
- Any subwave EW rule violation: primary count is INVALIDATED (see rule above) — confidence adjustments do not apply; a valid count must be found instead
- If fewer than 2 waves can be subwave-confirmed (too few bars or data unavailable): note "subwave confirmation: insufficient data" — do not penalize confidence

**Alternate count subwaves:** Apply the same MANDATORY COVERAGE RULE as the primary count. Attempt subwave identification for every alternate wave (motive ≥ 5 bars, corrective ≥ 3 bars on the subwave chart). Produce a `SUBWAVES (Alternate — confirmed waves only)` section using the same naming conventions (e.g. `WI.sw1`…`WI.sw5`, `WA.swa`…`WA.swc`). The same one-level depth limit applies. Mark each wave ✓, ⚠, or ✗(reason) — `✗(insuf)` is only valid for waves below the bar minimum; it is NOT valid for waves where data exists. Omit the section only when every single alternate wave qualifies as `✗(insuf)` — even one qualifying wave requires the section to be emitted.

**Step 5 — Select the PRIMARY Count**
- Choose the wave count that satisfies all three Elliott Wave rules
- Among valid counts, prefer the one where the most pivots align with Fibonacci levels
- Prefer the count where Wave 3 is the longest and most impulsive
- Prefer the count that aligns with the broader higher-timeframe trend
- Assign a confidence level (%) based on how cleanly the rules and guidelines are met
- Verify the selected count satisfies all Degree Proportion Rules from Step 1.5. A count that violates any proportion rule is invalid regardless of Fibonacci confluence — select a different pivot set.

**Step 6 — Select the ALTERNATE Count**
- Identify the next most valid wave count that also satisfies all three Elliott Wave rules
- This is typically a different wave degree, a different starting point, or a different corrective structure (zigzag vs flat vs triangle)
- The alternate count should tell a materially different story about where price is headed
- Identify the specific price level or bar pattern that would cause you to abandon the primary and adopt the alternate
- Assign a confidence level (%) reflecting how probable this scenario is relative to the primary
- **ALTERNATE PIVOT ANCHOR RULE: Every historical pivot in the alternate count MUST be anchored to an actual swing high or swing low that price physically printed on the chart. Re-labeling the same candles under a different wave name is allowed; inventing new price levels that were never traded is not. If no real swing fits a required alternate pivot, the alternate count is invalid — choose a different alternate structure instead.**
- **ALTERNATE DEGREE RULE: The alternate count must use the same degree locked in Step 1.5. Interpreting the same pivots as a different wave structure is allowed; introducing pivots from a different wave degree is not. An alternate count that requires a degree change to be internally valid must be discarded — choose a different alternate interpretation at the same locked degree.**

**Step 7 — Define Key Levels**
- Invalidation level for primary: the price at which the primary count is definitively wrong
- Invalidation level for alternate: the price at which the alternate count is definitively wrong
- Price target for primary: the projected end point of the next wave based on Fibonacci extensions
- Price target for alternate: the projected end point of the next wave under the alternate scenario

**Step 8 — Project Future Wave Movement**
- **ANCHOR RULE: Before projecting, walk the historical count all the way forward to today. Identify every confirmed swing high and swing low between the last labeled pivot and today's date on the selected timeframe. Each one that fits the wave structure must be added as a historical pivot (type: hist). Do NOT leave a gap of more than one timeframe period between the last historical pivot and today.**
- **LAST-PIVOT EXTENSION CHECK: After labeling the last hist pivot, confirm price has not since traded beyond it in the same direction. If it has, the pivot is mislabeled — re-identify the true terminal extreme, relabel it, and anchor projections from there.**
- **PROXIMITY CHECK: The last historical pivot must be within 8 weekly bars (or 8 daily bars on a daily chart) of today's date. If the most recently labeled hist pivot is older than that, the count is incomplete — continue identifying pivots until the last hist pivot is within that window.**
- **SUBWAVE DETECTION IN PROJECTION TRAJECTORY: For each projected pivot, use the already-built compact table (START DATE → today) to identify real swing highs/lows that occurred between the last historical pivot and today's date. Do NOT issue additional Yahoo Finance fetches. If actual market swings exist in the compact table between the anchor point and the projected pivot date, count them and include the count in the pivot's output (e.g., "WA (3 sw)" indicating 3 subwaves were identified in the trajectory). Apply the same PIVOT ACCEPTANCE GATE rules to these trajectory subwaves. If no real swings exist, output the projected pivot without a subwave count.**
- From that final confirmed pivot (now close to today), project the most probable future path for each count
- For the PRIMARY count: project at least 2 future pivots showing the expected next wave sequence
- For the ALTERNATE count: project at least 2 future pivots showing the expected next wave sequence under that scenario
- Estimate future pivot dates based on typical wave durations observed earlier in the same wave sequence
- Future pivots must be dated beyond today's date and clearly distinguished from historical pivots
- **PRICE BOUNDS CHECK FOR PROJECTIONS: Every projected pivot must be at a price level not yet traded since the last hist pivot — projected lows below the lowest Low, projected highs above the highest High printed since then. If a projected price falls inside the already-traded range, reclassify it as hist and re-anchor projections from the new last hist pivot.**
- **COMPLETE STRUCTURE RULE: Projected sequences must be structurally complete. For A-B-C corrections, all three legs (WA, WB, WC) must be projected. For impulse sequences, project through the full next wave. Never stop mid-structure (e.g. at WB without WC) — an incomplete projection does not represent a valid Elliott Wave scenario.**
- **MINIMUM PROJECTION SPAN: The last projected pivot in each count must be dated at least 60 calendar days after today's date. If Fibonacci-based timing yields a last projected pivot sooner than 60 days from today, extend the projection sequence by adding the next wave in the structure until coverage reaches at least today + 60 days. Do NOT use a horizontal flat line as a substitute — only real projected pivots count.**

**STEP 8 HARD STOP — Do not write any output until all three of the following are confirmed true:**

1. **Data fetched through today**: The Yahoo Finance API was called with `period2` set to today's Unix timestamp. Confirm the last bar in the returned data is within the current week — if not, the fetch was truncated, retry with a corrected `period2` before proceeding.
2. **Proximity satisfied**: The last `hist` pivot date is within 8 weekly bars (56 calendar days) of today. If not — STOP. Fetch the latest data, walk forward, and add the missing pivots before continuing.
3. **No stale projections**: For every `proj` pivot, confirm its price is outside the already-traded range since the last `hist` pivot. If any `proj` price has already been traded through — STOP. That pivot must be reclassified as `hist` and projections re-anchored from the new last `hist` pivot.

These three checks are not optional. An output written before all three pass is invalid. Note: terminal pivot integrity is enforced by the LAST-PIVOT EXTENSION CHECK in the anchor rule above.

---

### PRICE ACCURACY REQUIREMENT

**DATA SOURCING — HARD STOP:** Before recording any pivot, confirm the Yahoo Finance API response is non-null and covers the full date range. All pivot prices MUST be retrieved from the API — do not use memory or training data.

Follow the `yahoo-finance-fetch` skill (`.claude/skills/yahoo-finance-fetch.md`) for URL construction, interval mapping, BAR CAP, single-fetch strategy, persistent compact table, and error handling.

**HARD STOP — PIVOT ACCEPTANCE GATE:**
Before any pivot may be recorded in the output table, it MUST pass ALL of the following checks. A pivot that fails any check is REJECTED and must be replaced with the correct real market swing. Do NOT proceed to the next step until every pivot in the current count passes all gates.

For every candidate historical pivot (primary and alternate):
1. **Swing extreme check**: The price used MUST be the actual High of the bar (for swing highs) or the actual Low of the bar (for swing lows) on the selected timeframe — never Open, Close, or any other value.
2. **Neighboring-bar check**: The pivot bar's High (for a high pivot) must be higher than the High of every bar within ±2 bars of it on the selected timeframe; or its Low (for a low pivot) must be lower than the Low of every ±2 neighboring bars. If this is not true, the bar is not a confirmed swing extreme — find the actual extreme bar.
3. **No interpolation check**: The price must not be a rounded, averaged, midpoint, or theoretically-derived value. If the real High of the bar is $579.54, record $579.54 — not $580 or $575.
4. **Trading day check**: The date must be a confirmed trading day for the asset. If not, shift to the nearest valid trading day and re-run checks 1–3 on that bar.
5. **Alternate-count real-swing check**: If the pivot belongs to the alternate count, it must correspond to a bar that also qualifies as a swing extreme under checks 1–2 independently — not merely a bar that was relabeled to fit the alternate structure at a theoretical price.
6. **Source verification check**: The price must have been retrieved from an external data source in the DATA SOURCING step above — not recalled from model memory. A price that passed checks 1–5 against a fabricated or remembered value is still INVALID. If no external retrieval was performed for this pivot, it is REJECTED.
7. **Degree coherence check**: The wave's duration is consistent with the degree locked in Step 1.5. If this wave is more than 10× longer or shorter in duration than the median of its sibling waves in the same count, it is at the wrong degree — revise the surrounding pivot set before proceeding.

If a pivot fails any gate, it is INVALID. Do not use it. Find the nearest real swing extreme that passes all 7 checks.

**VALIDATION AT TWO STAGES:**

**Stage 1 — Historical Pivot Identification:** Apply the full PIVOT ACCEPTANCE GATE (defined above) to every pivot before accepting it. A pivot that fails any check is REJECTED at this stage — do not carry it forward.

**Stage 2 — Final Cross-Check (Before Output Only):** Re-apply checks 1, 3, 6, and 7 of the PIVOT ACCEPTANCE GATE to every accepted historical pivot. For projected pivots, apply the PRICE BOUNDS CHECK FOR PROJECTIONS (defined in Step 8). Checks 2, 4, and 5 were enforced at Stage 1 and do not require re-verification.

---

### COMPACT OUTPUT FORMAT

**SPLIT-CALL MODE:** When invoked by the main prompt as "Call A" (primary-only), output only: `Degree:` line, `PRIMARY COUNT` table, `SUBWAVES (Primary)` table, `Primary invalidation:` / `Primary target:` values, and `Subwave confirmation:` line — then stop. When invoked as "Call B" (alternate-only), output only: `ALTERNATE COUNT` table, `SUBWAVES (Alternate)` table (omit if <2 qualify), and `Alternate invalidation:` / `Alternate target:` values — then stop. When invoked without a split-call designation, output the full structure below.

After completing Step 8, output exactly this structure and nothing else.

**Header format note (consumed by `pinescript-generation-rules.md`):** The first line of the primary count block — `PRIMARY COUNT (X% confidence)` — is parsed by the generation rules' legend rule. The integer `X` is extracted as the confidence percentage displayed in the script legend. Do not alter this line's format.

```
Degree: [Name]
PRIMARY COUNT (X% confidence)
| Wave | Date       | Price    | OHLC | Fib    | Type |
|------|------------|----------|------|--------|------|
| W0   | YYYY-MM-DD | $XXX.XX  | L    | --     | hist |
| W1   | YYYY-MM-DD | $XXX.XX  | H    | XX.X%  | hist |
[... one row per pivot ...]
| W6   | YYYY-MM-DD | $XXX.XX  | proj | XX.X%  | proj |

SUBWAVES (Primary — confirmed waves only)
| Wave    | Date       | Price    | OHLC | Fib    | Type |
|---------|------------|----------|------|--------|------|
| W1.sw1  | YYYY-MM-DD | $XXX.XX  | L    | --     | hist |
| W1.sw2  | YYYY-MM-DD | $XXX.XX  | H    | XX.X%  | hist |
[... confirmed subwave rows only; omit waves that could not be confirmed ...]

SUBWAVES (Alternate — confirmed waves only)
| Wave    | Date       | Price    | OHLC | Fib    | Type |
|---------|------------|----------|------|--------|------|
[... omit section entirely if fewer than 2 alternate waves qualify ...]

ALTERNATE COUNT (X% confidence)
| Wave | Date       | Price    | OHLC | Fib    | Type |
|------|------------|----------|------|--------|------|
[... one row per pivot ...]

Primary invalidation: $XXX.XX  |  Primary target: $XXX.XX (Fib XX.X%)
Alternate invalidation: $XXX.XX  |  Alternate target: $XXX.XX (Fib XX.X%)
Subwave confirmation: [REQUIRED — list every primary wave: "W1✓ W2✓ W3✓ W4✗(insuf) W5⚠ → +10% confidence". Every wave (W1–W5) must appear here with a result. Missing waves are not allowed.]
```

The OHLC column must contain:
- `H` for swing highs (bar High used as pivot price)
- `L` for swing lows (bar Low used as pivot price)
- `proj` for projected future pivots (no historical bar)

Any pivot row with a missing or incorrect OHLC value is a signal that the PIVOT ACCEPTANCE GATE was not applied correctly.

**Subwave naming conventions for the output:**
- Motive subwaves: `W1.sw1`…`W1.sw5` (substitute W3, W5 as appropriate)
- Corrective subwaves: `W2.swa`, `W2.swb`, `W2.swc` (substitute W4, WA, WB, WC as appropriate)
- **Projected pivots with trajectory subwaves:** Append the subwave count in parentheses, e.g., `WA (3 sw)`, `WB (2 sw)`, `WC (1 sw)`. The number represents confirmed real market swings detected in the trajectory between the last historical pivot and this projected pivot. If no real swings were detected in the trajectory, omit the count (show just `WA` without parentheses).
