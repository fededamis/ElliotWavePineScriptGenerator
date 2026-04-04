---
name: yahoo-finance-fetch
description: Fetch historical OHLC price data from the Yahoo Finance v8 API using WebFetch. Covers URL construction, interval mapping, Unix timestamp handling, bar cap, JSON parsing, persistent compact table, and error handling.
---

### YAHOO FINANCE FETCH PROCEDURE

**Use the WebFetch tool to call the Yahoo Finance v8 chart endpoint.**

**SINGLE-FETCH STRATEGY — perform exactly ONE fetch per analysis:**
Always fetch at the **subwave timeframe** (daily for Cycle/Primary degree, weekly for Supercycle degree). This single fetch covers both primary pivot identification and all subwave identification — no secondary fetches are needed. Derive primary-timeframe swing extremes by identifying the highest High / lowest Low within each primary-chart period (e.g. group daily bars into weekly candles in-memory to find weekly swing extremes).

**URL construction:**
- **Base URL:** `https://query1.finance.yahoo.com/v8/finance/chart/{TICKER}`
- **Interval mapping:**
  - Subwave chart for Cycle/Primary degree → `interval=1d` (daily)
  - Subwave chart for Supercycle degree → `interval=1wk` (weekly)
  - Subwave chart for 4H primary → `interval=1h`
- **Date range:** Always supply `period1` (Unix timestamp of START DATE) and `period2` (Unix timestamp of today + 1 day).
- **BAR CAP:** Up to 1200 bars are allowed for daily intervals and 600 bars for weekly intervals. If the computed window exceeds the cap, advance `period1` to fit within the cap — do NOT split into multiple fetches.
- **Example (daily ETH-USD from 2022-06-01 to today):**
  `https://query1.finance.yahoo.com/v8/finance/chart/ETH-USD?interval=1d&period1=1654041600&period2=1743811200&events=history`

**Fetching procedure:**
1. Determine Unix timestamps for `period1` (START DATE) and `period2` (today).
2. Call WebFetch **once** with the constructed URL. Parse the JSON response:
   - `chart.result[0].timestamp[]` — array of bar open timestamps (Unix seconds)
   - `chart.result[0].indicators.quote[0].high[]` — High prices aligned by index
   - `chart.result[0].indicators.quote[0].low[]` — Low prices aligned by index

   **RAW RESPONSE SUPPRESSION — HARD CONSTRAINT: After the WebFetch call returns, do NOT echo, quote, summarize, or display any part of the raw API response in the chat. The raw JSON must never appear in the assistant turn. The only permitted action immediately after the fetch is to write the OHLCV cache file (step 2a) and then continue silently.**

2a. Immediately write the OHLCV cache file to `tmp/[TICKER].ohlcv.[timeframe].[START DATE].json` containing `{"schema":1,"ticker":"...","timeframe":"...","fetched_at":"ISO8601","bars":N,"data":[...]}`. This is the ONLY output action permitted after the fetch — no chat output.

3. Compile all extracted values into a **persistent compact internal table** (date | high | low) covering every bar from `period1` to `period2`. **DO NOT output or print this table.** This table remains in working memory for the entire analysis — all primary pivot lookups and all subwave pivot lookups use it. Do not re-reference the raw JSON after this step.
4. Record the exact `high` (for swing highs) or `low` (for swing lows) at the target index — full decimal precision as returned by the API.
5. If the WebFetch call fails, returns an error, or the ticker/date is not found:
   > **HARD STOP: Cannot verify pivot price for [TICKER] on [DATE] — Yahoo Finance API returned no data. Analysis halted. Verify ticker symbol and date range, then retry.**
   Do NOT substitute a remembered, estimated, or approximate price.

**Fallback (if Yahoo Finance v8 API is unreachable):** Retry once using the v7 endpoint: replace `/v8/finance/chart/` with `/v7/finance/chart/` in the URL, keeping all other parameters identical. If the v7 retry also fails, issue the HARD STOP above.
