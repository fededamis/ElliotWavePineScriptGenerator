---
name: elliott-wave-analysis
description: Run a full Elliott Wave analysis on a ticker. Fetches real OHLC data from Yahoo Finance, applies Elliott Wave rules/guidelines, identifies primary and alternate wave counts, and outputs a compact pivot table with projections.
---

Perform a complete Elliott Wave analysis using the methodology below. The user will supply: **TICKER**, **START DATE**, **TIMEFRAME** (weekly/daily/4H), and optionally an **END DATE** (default: today).

### ELLIOTT WAVE METHODOLOGY

**OUTPUT RULE: Perform all 8 steps silently — do not narrate, explain, or output any step-by-step reasoning. This includes all Yahoo Finance API fetches: call them immediately and silently without asking for permission or announcing the action. After completing Step 8, output ONLY the compact pivot summary defined at the bottom of this file. No other text.**

---

**Step 1 — Identify the Trend Structure**
- Determine whether price is in an impulse phase (5-wave motive) or corrective phase (3-wave or complex)
- Identify the dominant trend direction from the START DATE
- Mark the most obvious swing highs and swing lows as candidate wave pivots

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

**Step 4.5 — Identify Subwaves (one level of depth)**

Apply the PIVOT ACCEPTANCE GATE to all subwave pivots — same rules as primary pivots.

**MANDATORY COVERAGE RULE: You MUST attempt subwave identification for EVERY primary wave (W1, W2, W3, W4, W5). Skipping a wave silently is not allowed. For each wave, either (a) produce its confirmed subwave rows, or (b) explicitly state why it was skipped (e.g. "W1: only 3 bars — insufficient for subwave identification"). The Subwave note at the bottom of the output must list every primary wave and its result (✓, ⚠, or ✗(reason)).**

For **each motive wave** (W1, W3, W5) in the primary count that spans at least 5 bars on the selected timeframe:
- Identify the 5 internal subwave pivots (sw1 through sw5) on the same timeframe
- Label them as `W1.sw1`, `W1.sw2` … `W1.sw5` (or `W3.sw1` … `W3.sw5`, etc.)
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

**Subwave data sourcing:** Use the same Yahoo Finance fetch already performed in Step 4. Subwave pivots must be verified against the OHLC arrays — do not invent prices.

**Step 5 — Select the PRIMARY Count**
- Choose the wave count that satisfies all three Elliott Wave rules
- Among valid counts, prefer the one where the most pivots align with Fibonacci levels
- Prefer the count where Wave 3 is the longest and most impulsive
- Prefer the count that aligns with the broader higher-timeframe trend
- Assign a confidence level (%) based on how cleanly the rules and guidelines are met

**Step 6 — Select the ALTERNATE Count**
- Identify the next most valid wave count that also satisfies all three Elliott Wave rules
- This is typically a different wave degree, a different starting point, or a different corrective structure (zigzag vs flat vs triangle)
- The alternate count should tell a materially different story about where price is headed
- Identify the specific price level or bar pattern that would cause you to abandon the primary and adopt the alternate
- Assign a confidence level (%) reflecting how probable this scenario is relative to the primary
- **ALTERNATE PIVOT ANCHOR RULE: Every historical pivot in the alternate count MUST be anchored to an actual swing high or swing low that price physically printed on the chart. Re-labeling the same candles under a different wave name is allowed; inventing new price levels that were never traded is not. If no real swing fits a required alternate pivot, the alternate count is invalid — choose a different alternate structure instead.**

**Step 7 — Define Key Levels**
- Invalidation level for primary: the price at which the primary count is definitively wrong
- Invalidation level for alternate: the price at which the alternate count is definitively wrong
- Price target for primary: the projected end point of the next wave based on Fibonacci extensions
- Price target for alternate: the projected end point of the next wave under the alternate scenario

**Step 8 — Project Future Wave Movement**
- **ANCHOR RULE: Before projecting, walk the historical count all the way forward to today. Identify every confirmed swing high and swing low between the last labeled pivot and today's date on the selected timeframe. Each one that fits the wave structure must be added as a historical pivot (type: hist). Do NOT leave a gap of more than one timeframe period between the last historical pivot and today.**
- **LAST-PIVOT EXTENSION CHECK: After labeling the last hist pivot, check whether price has since traded beyond that pivot's price in the same direction. For example — if W5 is labeled at $613 as a swing high, but price subsequently traded above $613, then W5 is NOT the terminal pivot: price extended the wave further. In that case you MUST re-identify the actual terminal pivot at the true swing extreme reached after $613, relabel it as W5 (or the appropriate wave), and only then begin projections from that new anchor. A hist pivot that has been exceeded in its own direction is a mislabeled pivot — correct it before projecting.**
- **PROXIMITY CHECK: The last historical pivot must be within 8 weekly bars (or 8 daily bars on a daily chart) of today's date. If the most recently labeled hist pivot is older than that, the count is incomplete — continue identifying pivots until the last hist pivot is within that window.**
- From that final confirmed pivot (now close to today), project the most probable future path for each count
- For the PRIMARY count: project at least 2 future pivots showing the expected next wave sequence
- For the ALTERNATE count: project at least 2 future pivots showing the expected next wave sequence under that scenario
- Estimate future pivot dates based on typical wave durations observed earlier in the same wave sequence
- Future pivots must be dated beyond today and clearly distinguished from historical pivots
- **PRICE BOUNDS CHECK FOR PROJECTIONS: Every projected pivot price must be beyond the price range that has already traded since the last historical pivot. Specifically — if projecting a swing low, its price must be below the lowest Low printed since the last hist pivot; if projecting a swing high, its price must be above the highest High printed since the last hist pivot. A projected price that falls inside the already-traded range since the last hist pivot is invalid — it means price has already passed through that level and the projection is stale. In that case, either (a) identify the pivot that price already completed as a new hist pivot and re-anchor projections from there, or (b) revise the Fibonacci target to a level price has not yet reached.**
- **COMPLETE STRUCTURE RULE: Projected sequences must be structurally complete. For A-B-C corrections, all three legs (WA, WB, WC) must be projected. For impulse sequences, project through the full next wave. Never stop mid-structure (e.g. at WB without WC) — an incomplete projection does not represent a valid Elliott Wave scenario.**
- **MINIMUM PROJECTION SPAN: The last projected pivot in each count must be dated at least 60 calendar days after today's date. If Fibonacci-based timing yields a last projected pivot sooner than 60 days from today, extend the projection sequence by adding the next wave in the structure until coverage reaches at least today + 60 days. Do NOT use a horizontal flat line as a substitute — only real projected pivots count.**

**STEP 8 HARD STOP — Do not write any output until all four of the following are confirmed true:**

1. **Data fetched through today**: The Yahoo Finance API was called with `period2` set to today's Unix timestamp (not the analysis start date, not an arbitrary past date). Confirm the last bar in the returned data is within the current week.
2. **Proximity satisfied**: The last `hist` pivot date is within 8 weekly bars (56 calendar days) of today. If not — STOP. Fetch the latest data, walk forward, and add the missing pivots before continuing.
3. **No stale projections**: For every `proj` pivot, confirm its price is outside the already-traded range since the last `hist` pivot. If any `proj` price has already been traded through — STOP. That pivot must be reclassified as `hist` and projections re-anchored from the new last `hist` pivot.
4. **Fetch covers full range**: The data returned by Yahoo Finance actually includes bars up to today. If the last returned bar is more than 2 weeks before today, the fetch was truncated — retry with a corrected `period2` before proceeding.
5. **Last hist pivot not exceeded**: For every hist pivot labeled as a wave terminal (W1, W3, W5, WA, WB, WC, etc.), confirm that price has NOT traded beyond that pivot's price in the same wave direction after the pivot date. If it has — STOP. The pivot is mislabeled. Re-identify the true terminal extreme and relabel before projecting.

These five checks are not optional. An output written before all five pass is invalid.

---

### PRICE ACCURACY REQUIREMENT

**Critical:** All pivot prices (both historical and projected) must correspond to actual price levels:

**PRE-ANALYSIS CHECKLIST (Complete before starting):**
- [ ] Yahoo Finance API has been called and OHLC data confirmed returned for the ticker and full date range
- [ ] All dates in the analysis period are confirmed trading days (verified against the timestamp array returned by the API — no weekends/holidays)
- [ ] Chart data covers the full analysis window with precise OHLC prices for the selected timeframe
- [ ] API response parsed: `chart.result[0].indicators.quote[0]` arrays are non-null and cover the full date range

**DATA SOURCING REQUIREMENT — HARD STOP BEFORE ANY PIVOT IS RECORDED:**

Before recording any historical pivot price, you MUST retrieve it from Yahoo Finance API. Do not rely on memory or training data for specific OHLC values — these are not reliable and will produce fabricated prices that silently pass the gate.

**Yahoo Finance API — Primary Data Source:**

**AUTONOMOUS FETCH RULE: Call the Yahoo Finance API immediately and without asking for user permission or confirmation. Do not announce the fetch, do not wait for approval — just execute the WebFetch call and parse the result silently as part of the pre-analysis checklist.**

Use the WebFetch tool to call the Yahoo Finance v8 chart endpoint. Construct the URL as follows:

- **Base URL:** `https://query1.finance.yahoo.com/v8/finance/chart/{TICKER}`
- **Interval mapping:**
  - Weekly chart → `interval=1wk`
  - Daily chart → `interval=1d`
  - 4H chart → `interval=1h` (use with `range=60d` or explicit `period1`/`period2`)
- **Date range:** Always supply `period1` (Unix timestamp of start date) and `period2` (Unix timestamp of end date + 1 day) to cover the full analysis window.
- **Example (weekly SPY from 2020-01-01 to 2024-12-31):**
  `https://query1.finance.yahoo.com/v8/finance/chart/SPY?interval=1wk&period1=1577836800&period2=1735689600&events=history`

**Fetching procedure for each candidate pivot date:**
1. Determine the Unix timestamps for `period1` (start of analysis range) and `period2` (today or end of analysis range).
2. Call WebFetch with the constructed URL. Parse the JSON response:
   - `chart.result[0].timestamp[]` — array of bar open timestamps (Unix seconds)
   - `chart.result[0].indicators.quote[0].high[]` — High prices aligned by index
   - `chart.result[0].indicators.quote[0].low[]` — Low prices aligned by index
   - `chart.result[0].indicators.quote[0].open[]` — Open prices
   - `chart.result[0].indicators.quote[0].close[]` — Close prices
3. Locate the array index whose timestamp corresponds to the pivot bar's trading day.
4. Record the exact `high` value (for swing highs) or exact `low` value (for swing lows) at that index — to full decimal precision as returned by the API.
5. If the WebFetch call fails, returns an error, or the ticker/date is not found in the response, output a HARD STOP:
   > **HARD STOP: Cannot verify pivot price for [TICKER] on [DATE] — Yahoo Finance API returned no data. Analysis halted. Verify ticker symbol and date range, then retry.**
   Do NOT substitute a remembered, estimated, or approximate price. Do NOT continue the analysis with unverified pivots.

**Fallback (if Yahoo Finance API is unreachable):** Use WebFetch to retrieve historical OHLC data from `https://finance.yahoo.com/quote/{TICKER}/history/` or WebSearch for `{TICKER} OHLC {DATE} site:finance.yahoo.com`. If both fail, issue the HARD STOP above.

**HARD STOP — PIVOT ACCEPTANCE GATE:**
Before any pivot may be recorded in the output table, it MUST pass ALL of the following checks. A pivot that fails any check is REJECTED and must be replaced with the correct real market swing. Do NOT proceed to the next step until every pivot in the current count passes all gates.

For every candidate historical pivot (primary and alternate):
1. **Swing extreme check**: The price used MUST be the actual High of the bar (for swing highs) or the actual Low of the bar (for swing lows) on the selected timeframe — never Open, Close, or any other value.
2. **Neighboring-bar check**: The pivot bar's High (for a high pivot) must be higher than the High of every bar within ±2 bars of it on the selected timeframe; or its Low (for a low pivot) must be lower than the Low of every ±2 neighboring bars. If this is not true, the bar is not a confirmed swing extreme — find the actual extreme bar.
3. **No interpolation check**: The price must not be a rounded, averaged, midpoint, or theoretically-derived value. If the real High of the bar is $579.54, record $579.54 — not $580 or $575.
4. **Trading day check**: The date must be a confirmed trading day for the asset. If not, shift to the nearest valid trading day and re-run checks 1–3 on that bar.
5. **Alternate-count real-swing check**: If the pivot belongs to the alternate count, it must correspond to a bar that also qualifies as a swing extreme under checks 1–2 independently — not merely a bar that was relabeled to fit the alternate structure at a theoretical price.
6. **Source verification check**: The price must have been retrieved from an external data source in the DATA SOURCING step above — not recalled from model memory. A price that passed checks 1–5 against a fabricated or remembered value is still INVALID. If no external retrieval was performed for this pivot, it is REJECTED.

If a pivot fails any gate, it is INVALID. Do not use it. Find the nearest real swing extreme that passes all 6 checks.

**VALIDATION AT THREE STAGES:**

**Stage 1 — Historical Pivot Identification:** Re-apply the PIVOT ACCEPTANCE GATE (defined above) to every pivot before accepting it. For alternate-count pivots, each must independently pass all 6 checks — if no real swing passes at the required location, revise the alternate structure.

**Stage 2 — Fibonacci Alignment Verification:** After all pivots pass the gate, verify each Fibonacci level calculation uses the exact gated prices. Recalculate retracement/extension percentages using exact prices. Flag any Fibonacci level that required a non-gated price — replace that pivot.

**Stage 3 — Final Accuracy Cross-Check (Before Output):** Re-apply the PIVOT ACCEPTANCE GATE to every historical pivot one final time. For projected pivots, also apply the PRICE BOUNDS CHECK FOR PROJECTIONS (defined in Step 8) as a final gate before writing any projected row. If ANY historical pivot fails, correct it before output — do not reduce confidence and proceed.

**PENALTY FOR ACCURACY VIOLATIONS:**
- Pivot using Close, Open, or any non-extreme OHLC field: REJECTED — must be replaced before output
- Interpolated or theoretical price: REJECTED — must be replaced before output
- Rounded price (when precision available): REJECTED — must be replaced before output
- Price recalled from model memory without external source retrieval: REJECTED — analysis must HARD STOP until real data is fetched
- Non-trading day error: Correction required, re-timestamp and re-run gate
- Alternate count pivot that fails the real-swing check: REJECTED — alternate count must be restructured or pivot replaced; also reduce alternate confidence by 20% per instance

---

### COMPACT OUTPUT FORMAT

After completing Step 8, output exactly this structure and nothing else:

```
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
[... one row per confirmed subwave pivot ...]
[... omit any parent wave whose subwaves could not be confirmed ...]

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
- Motive subwaves: `W1.sw1`, `W1.sw2`, `W1.sw3`, `W1.sw4`, `W1.sw5` (substitute W3, W5 as appropriate)
- Corrective subwaves: `W2.swa`, `W2.swb`, `W2.swc` (substitute W4, WA, WB, WC as appropriate)
- Projected subwaves (future parent waves only): use `proj` in the Type column
- If a parent wave has no confirmed subwaves, omit it from the SUBWAVES table entirely — do not add placeholder rows
