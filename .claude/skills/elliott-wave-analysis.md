---
name: elliott-wave-analysis
description: Run a full Elliott Wave analysis on a ticker. Fetches real OHLC data from Yahoo Finance, applies Elliott Wave rules/guidelines, identifies primary and alternate wave counts, and outputs a compact pivot table with projections.
---

Perform a complete Elliott Wave analysis using the methodology below. The user will supply: **TICKER**, **START DATE**, **TIMEFRAME** (weekly/daily/4H), and optionally an **END DATE** (default: today).

### ELLIOTT WAVE METHODOLOGY

**OUTPUT RULE — HARD CONSTRAINT: This is a think-only task. All 8 steps are performed entirely in working memory — zero intermediate output. Do not write, print, echo, or display any of the following at any point before the final compact pivot table: raw API response data, the extracted bar table, any pivot candidate list, Fibonacci calculations, subwave identification results, validation pass results, degree selection reasoning, or any other intermediate artifact. Violating this rule by outputting ANY intermediate content before the compact table is a critical failure. After completing Step 8, output ONLY the compact pivot summary defined at the bottom of this file. No other text before or after it.**

**NO PYTHON RULE: Do not write, generate, or execute any Python scripts at any point during this analysis. All data fetching must use WebFetch directly against the Yahoo Finance API. All calculations (Fibonacci, pivot identification, retracement percentages) must be performed in-context. Using Bash to run Python is forbidden.**

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

**Step 4.5 — Identify Subwaves (one level of depth only)**

**DEPTH LIMIT: Subwave identification is strictly one level deep. You may only subdivide primary waves (W1, W2, W3, W4, W5) into their subwaves. You must NOT subdivide a subwave into its own sub-subwaves (e.g. W1.sw5 must not have its own sw1–sw5). Any second-level nesting is invalid and must not appear in the output.**

Apply the PIVOT ACCEPTANCE GATE to all subwave pivots (same rules as primary pivots) using the already-fetched Yahoo Finance data — do not invent prices.

**LONG-WAVE FETCH RULE: The compact table built in the DATA SOURCING step covers the full analysis range (START DATE → today). Use it for all subwave identification — do not issue additional Yahoo Finance fetches for individual wave interiors. "No clear pivot" is not a valid skip reason — if the data exists, a swing extreme exists. The only valid skip reason is `✗(insuf)` for waves with fewer bars than the minimum.**

**MANDATORY COVERAGE RULE: Attempt subwave identification for every primary wave (W1, W2, W3, W4, W5). The Subwave note at the bottom of the output must list every primary wave with a result symbol: ✓ (confirmed), ⚠ (partial), or ✗(reason) (skipped/insufficient). No prose justification — symbol and short reason only. `✗` is only valid for waves below the bar minimum — it is NOT valid for long waves where data exists.**

For **each motive wave** (W1, W3, W5) in the primary count that spans at least 5 bars on the selected timeframe:
- Identify the 5 internal subwave pivots (sw1 through sw5) on the same timeframe
- Label them as `W1.sw1`, `W1.sw2` … `W1.sw5` (or `W3.sw1` … `W3.sw5`, etc.)
- **SHORT-WAVE ±1 FALLBACK: If a motive wave spans fewer than 20 bars and no interior swing passes the ±2 neighboring-bar check (check 2 of the PIVOT ACCEPTANCE GATE), relax the neighboring-bar check to ±1 for that wave's subwaves only. If at least 3 of the 5 required pivots pass the ±1 gate, output those rows and mark the wave `⚠(±1 gate, N/5 subwaves)` in the Subwave confirmation line. If even the ±1 gate yields fewer than 3 pivots, mark the wave `✗(too few bars for subwave resolution)` — do NOT mark it `⚠` without outputting any rows.**
- **Apply the three absolute EW rules at the subwave level — these are not optional:**
  1. sw2 never retraces more than 100% of sw1 (sw2's extreme cannot go beyond the start of sw1, i.e. the parent wave's origin)
  2. sw3 is never the shortest among sw1, sw3, and sw5
  3. sw4 never overlaps sw1's price territory
- **If any of these three rules is violated, the PRIMARY COUNT IS INVALIDATED. The three absolute EW rules apply at every wave degree — a violation at the subwave level means the primary pivot labeling is wrong (the parent wave's start or end was mislabeled). STOP. Do not output the current primary count. Return to Step 5, select a different pivot set, and re-run subwave verification. Repeat until a primary count is found whose subwaves all satisfy the three rules.**

For **each corrective wave** (W2, W4, WA, WB, WC) in the primary count that spans at least 3 bars:
- Identify the 3 internal subwave pivots (swa, swb, swc) on the same timeframe
- Label them as `W2.swa`, `W2.swb`, `W2.swc` (etc.)
- Verify zigzag, flat, or triangle structure is internally consistent

If a wave spans fewer bars than required (motive < 5 bars, corrective < 3 bars), mark it as `✗(insuf)` in the Subwave note — do not silently omit it.

**Subwave Confidence Contribution:**
- Each motive wave whose 5 subwaves all pass the acceptance gate and satisfy EW rules: +3% to primary count confidence
- Each corrective wave whose a-b-c subwaves all pass the acceptance gate and satisfy EW rules: +2% to primary count confidence
- Any subwave EW rule violation: primary count is INVALIDATED (see rule above) — confidence adjustments do not apply; a valid count must be found instead
- If fewer than 2 waves can be subwave-confirmed (too few bars or data unavailable): note "subwave confirmation: insufficient data" — do not penalize confidence

**Alternate count subwaves:** Only if the alternate count has at least 2 historical waves spanning sufficient bars (motive ≥ 5 bars, corrective ≥ 3 bars), produce a `SUBWAVES (Alternate — confirmed waves only)` section using the same naming conventions (e.g. `WI.sw1`…`WI.sw5`, `WA.swa`…`WA.swc`). If fewer than 2 alternate waves qualify, omit the section entirely. The same one-level depth limit applies.

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

Follow the `yahoo-finance-fetch` skill (`.claude/skills/yahoo-finance-fetch.md`) for URL construction, interval mapping, BAR CAP, fetching procedure, EXTRACT-THEN-DISCARD, and error handling.

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
