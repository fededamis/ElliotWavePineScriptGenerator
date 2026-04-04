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

After the command completes, immediately run a **second single `run_in_terminal` command** that:
1. Reads the raw cache file
2. Detects swing highs and swing lows in-process (a bar is a swing high if its High > the High of both neighboring bars; a swing low if its Low < the Low of both neighboring bars — use a ±2 bar window)
3. Converts timestamps to ISO dates
4. Writes the result to `tmp/[TICKER].swings.[TIMEFRAME].[START_DATE].json` as `{"schema":1,"ticker":"...","timeframe":"...","swH":[{"date":"YYYY-MM-DD","high":X,"bar":N},...], "swL":[{"date":"YYYY-MM-DD","low":X,"bar":N},...]}` 

Use this PowerShell template (substitute values):
```powershell
$c = Get-Content "tmp/{TICKER}.ohlcv.{TIMEFRAME}.{START_DATE}.json" -Raw | ConvertFrom-Json
$d = $c.data; $n = $d.Count
$swH = @(); $swL = @()
for ($i = 2; $i -lt $n-2; $i++) {
  if ($d[$i].high -gt $d[$i-1].high -and $d[$i].high -gt $d[$i-2].high -and $d[$i].high -gt $d[$i+1].high -and $d[$i].high -gt $d[$i+2].high) { $swH += @{date=$d[$i].date;high=$d[$i].high;bar=$i} }
  if ($d[$i].low -lt $d[$i-1].low -and $d[$i].low -lt $d[$i-2].low -and $d[$i].low -lt $d[$i+1].low -and $d[$i].low -lt $d[$i+2].low) { $swL += @{date=$d[$i].date;low=$d[$i].low;bar=$i} }
}
@{schema=1;ticker="{TICKER}";timeframe="{TIMEFRAME}";swH=$swH;swL=$swL} | ConvertTo-Json -Depth 4 | Set-Content "tmp/{TICKER}.swings.{TIMEFRAME}.{START_DATE}.json"
Write-Host "swH=$($swH.Count) swL=$($swL.Count)"
```

Then read `tmp/[TICKER].swings.[TIMEFRAME].[START_DATE].json` with `read_file` to load all swing pivots into working memory. **Do not read the full bar cache file at all** — use only the swings file for all pivot identification and analysis. This reduces tool calls to 3 total (fetch → swing-extract → read swings) regardless of dataset size.

If `Invoke-WebRequest` fails (HTTP error or network issue), fall back to the `fetch_webpage` method below and prepend a note: `⚠ Terminal fetch failed — falling back to fetch_webpage (raw response will appear in chat).`

**Subagent / Claude.ai Mode — use `fetch_webpage`:**

Call `fetch_webpage` once with the constructed URL.

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
