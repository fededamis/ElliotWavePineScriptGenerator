---
name: yahoo-finance-fetch
description: Fetch historical OHLC price data from the Yahoo Finance v8 API using WebFetch. Covers URL construction, interval mapping, Unix timestamp handling, bar cap, JSON parsing, EXTRACT-THEN-DISCARD, and error handling.
---

### YAHOO FINANCE FETCH PROCEDURE

**Use the WebFetch tool to call the Yahoo Finance v8 chart endpoint.**

**URL construction:**
- **Base URL:** `https://query1.finance.yahoo.com/v8/finance/chart/{TICKER}`
- **Interval mapping:**
  - Weekly chart → `interval=1wk`
  - Daily chart → `interval=1d`
  - 4H chart → `interval=1h` (use with explicit `period1`/`period2`)
- **Date range:** Always supply `period1` (Unix timestamp of start date) and `period2` (Unix timestamp of end date + 1 day) to cover the full analysis window.
- **BAR CAP:** If the computed window exceeds 300 bars on the selected timeframe (≈300 weeks or ≈300 trading days), advance `period1` so that exactly the most recent 300 bars are fetched. Older bars are outside scope.
- **Example (weekly SPY from 2020-01-01 to 2024-12-31):**
  `https://query1.finance.yahoo.com/v8/finance/chart/SPY?interval=1wk&period1=1577836800&period2=1735689600&events=history`

**Fetching procedure:**
1. Determine Unix timestamps for `period1` (start of analysis range) and `period2` (today or end of analysis range).
2. Call WebFetch with the constructed URL. Parse the JSON response:
   - `chart.result[0].timestamp[]` — array of bar open timestamps (Unix seconds)
   - `chart.result[0].indicators.quote[0].high[]` — High prices aligned by index
   - `chart.result[0].indicators.quote[0].low[]` — Low prices aligned by index
3. Locate the array index whose timestamp corresponds to the target bar's trading day.
4. Record the exact `high` (for swing highs) or `low` (for swing lows) at that index — full decimal precision as returned by the API.
   **EXTRACT-THEN-DISCARD:** Immediately after step 4, compile all extracted values into a compact internal table (date | high | low) covering every bar. **DO NOT output or print this table — it is internal working memory only.** After building it, discard the full JSON — do not re-reference `timestamp[]`, `high[]`, or `low[]` in any subsequent step. All pivot lookups use only the extracted compact table.
5. If the WebFetch call fails, returns an error, or the ticker/date is not found:
   > **HARD STOP: Cannot verify pivot price for [TICKER] on [DATE] — Yahoo Finance API returned no data. Analysis halted. Verify ticker symbol and date range, then retry.**
   Do NOT substitute a remembered, estimated, or approximate price.

**Fallback (if Yahoo Finance API is unreachable):** Use WebFetch to retrieve historical OHLC data from `https://finance.yahoo.com/quote/{TICKER}/history/` or WebSearch for `{TICKER} OHLC {DATE} site:finance.yahoo.com`. If both fail, issue the HARD STOP above.
