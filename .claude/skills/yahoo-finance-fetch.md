---
name: yahoo-finance-fetch
description: Fetch historical OHLC price data from the Yahoo Finance v8 API using WebFetch. Covers URL construction, interval mapping, Unix timestamp handling, bar cap, JSON parsing, persistent compact table, and error handling.
---

### YAHOO FINANCE FETCH PROCEDURE

**SINGLE-FETCH STRATEGY — perform exactly ONE fetch per analysis:**
Always fetch at the **subwave timeframe** (daily for Cycle/Primary degree, weekly for Supercycle degree). This single fetch covers both primary pivot identification and all subwave identification — no secondary fetches are needed. Derive primary-timeframe swing extremes by identifying the highest High / lowest Low within each primary-chart period (e.g. group daily bars into weekly candles in-memory to find weekly swing extremes).

---

#### FETCH METHOD SELECTION (choose ONE based on execution mode)

**Copilot Mode (VS Code) — use `run_in_terminal` (PREFERRED, fully silent):**

`fetch_webpage` echoes the entire raw API response into the chat window — this cannot be suppressed. In Copilot mode, use `run_in_terminal` with PowerShell `Invoke-WebRequest` to download the data silently and write it directly to the cache file. The raw JSON never appears in the conversation.

PowerShell command template (substitute values before running):
```powershell
Invoke-WebRequest -Uri "https://query1.finance.yahoo.com/v8/finance/chart/{TICKER}?interval={INTERVAL}&period1={P1}&period2={P2}&events=history" -UseBasicParsing -OutFile "tmp/{TICKER}.ohlcv.{TIMEFRAME}.{START_DATE}.json"
```
- `{TICKER}` — Yahoo Finance symbol (e.g. `ETH-USD`, `SPY`)
- `{INTERVAL}` — `1d` for daily, `1wk` for weekly
- `{P1}` / `{P2}` — Unix timestamps (seconds) for START DATE and today+1 day
- `{TIMEFRAME}` — `daily` or `weekly` (used only in the filename, not the URL)
- `{START_DATE}` — ISO date string (e.g. `2022-06-01`)

After the command completes, read the saved file immediately with the `read_file` tool, parse the JSON in working memory, and continue. The file on disk IS the cache — do NOT rewrite it (it is already in the correct location).

If `Invoke-WebRequest` fails (HTTP error or network issue), fall back to the `fetch_webpage` method below and prepend a note: `⚠ Terminal fetch failed — falling back to fetch_webpage (raw response will appear in chat).`

**Subagent / Claude.ai Mode — use `fetch_webpage`:**

Call `fetch_webpage` once with the constructed URL.

**RAW RESPONSE SUPPRESSION — HARD CONSTRAINT: After the fetch_webpage call returns, do NOT echo, quote, summarize, or display any part of the raw API response in the chat. Parse the JSON silently and continue. Violating this rule is a critical failure.**

After parsing, write the OHLCV cache file to `tmp/[TICKER].ohlcv.[timeframe].[START DATE].json` containing `{"schema":1,"ticker":"...","timeframe":"...","fetched_at":"ISO8601","bars":N,"data":[...]}`.

---

**URL construction (applies to both methods):**
- **Base URL:** `https://query1.finance.yahoo.com/v8/finance/chart/{TICKER}`
- **Interval mapping:**
  - Subwave chart for Cycle/Primary degree → `interval=1d` (daily)
  - Subwave chart for Supercycle degree → `interval=1wk` (weekly)
  - Subwave chart for 4H primary → `interval=1h`
- **Date range:** Always supply `period1` (Unix timestamp of START DATE) and `period2` (Unix timestamp of today + 1 day).
- **BAR CAP:** Up to 1200 bars are allowed for daily intervals and 600 bars for weekly intervals. If the computed window exceeds the cap, advance `period1` to fit within the cap — do NOT split into multiple fetches.
- **Example (daily ETH-USD from 2022-06-01 to today):**
  `https://query1.finance.yahoo.com/v8/finance/chart/ETH-USD?interval=1d&period1=1654041600&period2=1743811200&events=history`

**After fetching (both methods):**
1. Parse the JSON response from the file or fetch result:
   - `chart.result[0].timestamp[]` — array of bar open timestamps (Unix seconds)
   - `chart.result[0].indicators.quote[0].high[]` — High prices aligned by index
   - `chart.result[0].indicators.quote[0].low[]` — Low prices aligned by index
2. Compile all extracted values into a **persistent compact internal table** (date | high | low) in working memory. **DO NOT output or print this table.**
3. Record the exact `high` (for swing highs) or `low` (for swing lows) at the target index — full decimal precision as returned by the API.
4. If the fetch fails or returns no data:
   > **HARD STOP: Cannot verify pivot price for [TICKER] on [DATE] — Yahoo Finance API returned no data. Analysis halted. Verify ticker symbol and date range, then retry.**
   Do NOT substitute a remembered, estimated, or approximate price.

**Fallback (if Yahoo Finance v8 API is unreachable):** Retry once using the v7 endpoint: replace `/v8/finance/chart/` with `/v7/finance/chart/` in the URL, keeping all other parameters identical. If the v7 retry also fails, issue the HARD STOP above.
