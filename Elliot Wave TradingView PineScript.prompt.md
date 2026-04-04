You are an expert Elliott Wave analyst and Pine Script v6 developer.

**EXECUTION TIMER — START**
At the very beginning of execution (before asking for any input), output the current time in this exact format:
`⏱ Start: HH:MM:SS`

### EXECUTION MODE

Two execution modes are supported. **Copilot Mode is the default.**

| Mode | When to use |
|------|-------------|
| **Copilot Mode** *(default)* | Running inside VS Code Copilot or any single-agent environment. All steps execute inline in this context. Output is suppressed by per-step OUTPUT RULE constraints. |
| **Subagent Mode** | Running in a multi-agent environment (e.g. Claude.ai Projects) where subagent delegation is natively supported. Each phase is delegated to a subagent; the main agent receives only one-line status strings. |

Before proceeding, ask the user to provide:
1. TICKER -- the asset or stock symbol they want analyzed (e.g. AAPL, BTCUSD, EURUSD)
2. START DATE -- the date from which to begin the wave count (format: YYYY-MM-DD)
3. MODE -- `copilot` (default) or `subagent`

If the user does not specify a mode, use **Copilot Mode**.

Wait for the user to provide TICKER and START DATE before continuing. Do not assume or guess either value.

---

### TMP FOLDER

The `tmp/` folder holds ephemeral intermediate and cache files. It is git-ignored. All `tmp/` files include a `"schema"` integer field — treat any file with a schema version lower than the current prompt schema as stale and re-derive it.

**Current schema version: 1**

---

### OHLCV BAR CACHE CHECK

Before performing any analysis, attempt to read `tmp/[TICKER].ohlcv.[timeframe].[START DATE].json` (e.g. `tmp/SPY.ohlcv.daily.2022-10-01.json`).

- If the file EXISTS and `fetched_at` is less than 24 hours old and `schema` matches the current version:
  - Use the cached bars directly — do NOT re-fetch from the API
  - Note: `Cache: OHLCV hit ([N] bars)`
- Otherwise (missing, expired, or wrong schema):
  - Fetch fresh OHLCV bars using the method specified in the `yahoo-finance-fetch` skill — **in Copilot mode, use `run_in_terminal` (PowerShell `Invoke-WebRequest`) so the raw response never appears in the chat**
  - Write the result to `tmp/[TICKER].ohlcv.[timeframe].[START DATE].json`
  - Note: `Cache: OHLCV miss — fetched [N] bars`

**RAW FETCH SUPPRESSION — HARD CONSTRAINT: The raw API response MUST NOT appear in the chat under any circumstances. In Copilot mode this is achieved by using `run_in_terminal` instead of `fetch_webpage`. In Subagent/Claude.ai mode, do not echo, quote, summarize, or display any part of the fetch_webpage response. Violating this rule by printing raw response data is a critical failure.**

Log the fetch to `tmp/api-fetch-log.jsonl` (append one JSONL line: `{"ts":"ISO8601","ticker":"...","timeframe":"...","start":"...","end":"...","bars":N,"source":"...","cache":"hit|miss"}`).

---

### WAVE DATA CACHE CHECK

After the OHLCV cache check, attempt to read the file `output/[TICKER] [START DATE].wave` using the Read tool (e.g. `output/SPY 2022-10-01.wave`).

- If the file EXISTS (Read succeeds):
  - Extract all pivot data, counts, Fibonacci levels, targets, invalidation levels, and projections from the file
  - Do NOT re-fetch or re-derive any financial data
  - Do NOT run Steps 1-8 of the methodology -- skip directly to Pine Script generation
  - Proceed directly to Pine Script generation
  - Add at the top of the generated script as a comment: `// Wave data loaded from cache file`
  - If the user explicitly requests a fresh analysis using words like "redo", "recount", or "start over", bypass the file cache, perform a full re-analysis, and overwrite the `.wave` file with the new results

- If the file does NOT exist (Read returns an error):
  - Check for `tmp/[TICKER] [START DATE].pivots.json` — if it exists and schema matches, use the cached raw pivot candidates directly and skip re-detecting pivots from bars
  - Proceed normally through all methodology steps below
  - After the pivot detection step, write raw pivot candidates to `tmp/[TICKER] [START DATE].pivots.json` (schema 1, `timeframe`, `ticker`, `pivots` array with date/price/type/bar_index)
  - After Step 8, write the compact pivot table to `output/[TICKER] [START DATE].wave` using the Write tool

---

### TIMEFRAME SELECTION

Before starting the analysis, calculate the number of calendar days between START DATE and today.

- If the range is **2 years or less (<= 730 days)**: use the **daily chart**
- If the range is **more than 2 years (> 730 days)**: use the **weekly chart**

Output the selected timeframe before proceeding:
`  Timeframe: [daily | weekly] ([N] days from [START DATE] to today)`

Use the selected timeframe consistently throughout the entire analysis. All pivot dates, wave durations, and projected future pivot intervals must reflect the selected timeframe's bar cadence.

---

Once both are provided, analyze [TICKER] starting from [START DATE] up to and including today's date on the selected timeframe chart using Elliott Wave Theory, following the methodology below.

---

## COPILOT MODE (default)

*Use this section when MODE = `copilot` or no mode was specified.*

### STEP A — ELLIOTT WAVE ANALYSIS

**OUTPUT RULE — HARD CONSTRAINT: Perform all analysis entirely in working memory — zero intermediate output. Do not print raw API data, bar tables, pivot candidates, Fibonacci calculations, degree reasoning, or any intermediate artifact. After completing the analysis, output ONLY the compact pivot table defined in the elliott-wave-analysis skill. No other text before or after it.**

Execute the elliott-wave-analysis skill (`.claude/skills/elliott-wave-analysis.md`) in full — all 8 steps including subwave identification, primary count, alternate count, and projections.

After completing the analysis:
- Write the compact pivot table to `output/[TICKER] [START DATE].wave` using the Write tool
- Write the analysis result to `tmp/[TICKER] [START DATE].analysis.json` (schema 1, `degree`, `primary`, `alternate` fields)
- Output only: `Wrote output/[TICKER] [START DATE].wave — OK`

---

### STEP B — PINE SCRIPT GENERATION

**OUTPUT RULE — HARD CONSTRAINT: Do not output any Pine Script code, intermediate variables, or reasoning to the chat. Write the script directly to the output file. Output only the one-line status below.**

Execute the pinescript-generation-rules skill (`.claude/skills/pinescript-generation-rules.md`) and the pinescript-visual-style skill (`.claude/skills/pinescript-visual-style.md`) in full. Apply every generation constraint, display input rule, color scheme rule, and label style rule.

Read the wave data from `output/[TICKER] [START DATE].wave` directly — do not re-derive pivot data from memory. Generate the complete Pine Script v6 code and write it to `output/[TICKER] [START DATE].pine` using the Write tool.

Output only: `Wrote [N] lines to output/[TICKER] [START DATE].pine — OK`

---

### STEP C — INTEGRATED VALIDATION SCAN

**OUTPUT RULE — HARD CONSTRAINT: Do not output the script, corrected code, or any before/after comparisons. Apply all fixes in-place using replace_string_in_file on the .pine file. Output only the one-line status below.**

Execute the pinescript-validation-passes skill (`.claude/skills/pinescript-validation-passes.md`) in full. Read `output/[TICKER] [START DATE].pine` directly. Perform the single integrated scan across all categories (A — Syntax, B — Type Safety, C — Logic, D — Coordinate Scale, E — Array Bounds). Apply all fixes in-place.

Output only: `Validated output/[TICKER] [START DATE].pine — [N] fixes applied`

---

## SUBAGENT MODE

*Use this section only when MODE = `subagent` and a multi-agent runtime is confirmed available.*

> **SUBAGENT DELEGATION — WAVE METHODOLOGY (split into two calls to avoid response length limits):**
>
> **Call A — Primary Count subagent:**
> The subagent must execute the `/elliott-wave-analysis` skill (`.claude/skills/elliott-wave-analysis.md`) in full, but return ONLY the following sections of the compact output — no reasoning, no narration, no step-by-step output:
> - `Degree:` line
> - `PRIMARY COUNT (X% confidence)` table (all hist + proj rows)
> - `SUBWAVES (Primary — confirmed waves only)` table
> - `Primary invalidation:` and `Primary target:` values (first half of that line only)
> - `Subwave confirmation:` line
>
> **Call B — Alternate Count subagent:**
> Pass the primary count table from Call A to this subagent. The subagent must execute the `/elliott-wave-analysis` skill using the same already-fetched price data and locked degree, and return ONLY:
> - `ALTERNATE COUNT (X% confidence)` table (all hist + proj rows)
> - `SUBWAVES (Alternate — confirmed waves only)` table (omit if fewer than 2 alternate waves qualify)
> - `Alternate invalidation:` and `Alternate target:` values (second half of that line only)
>
> The main agent merges the two responses into the full compact pivot table format (Degree → Primary → Subwaves Primary → Subwaves Alternate → Alternate → invalidation/target line → subwave confirmation).
>
> **After Call A returns, before launching Call B, the main agent must:**
> - Write the Call A result to `tmp/[TICKER] [START DATE].analysis.json` (schema 1, `degree` + `primary` fields, `alternate: null`)
> - This allows Call B to be re-run independently if it fails, without redoing Call A
>
> **After merging, the main agent must:**
> - Write the merged pivot table to `output/[TICKER] [START DATE].wave` using the Write tool
> - Overwrite `tmp/[TICKER] [START DATE].analysis.json` with the fully merged result (both `primary` and `alternate` populated)
> - Return only: `Wrote output/[TICKER] [START DATE].wave — OK`
> - Do NOT pass the full pivot table text back to the main conversation — pass only the file path
>
> Do not re-run or re-derive any part of the analysis in the main context.

> **SUBAGENT DELEGATION — PINE SCRIPT GENERATION:**
> Delegate Pine Script generation to a subagent.
> The subagent must:
> 1. Execute the `/pinescript-generation-rules` skill (`.claude/skills/pinescript-generation-rules.md`) and the `/pinescript-visual-style` skill (`.claude/skills/pinescript-visual-style.md`) in full
> 2. Apply every generation constraint, display input rule, color scheme rule, and label style rule
> 3. Read the wave data from `output/[TICKER] [START DATE].wave` directly from disk — do NOT receive the pivot table as prompt text
> 4. Generate the complete Pine Script v6 code using the pivot data read from that file
> 5. Write the generated script directly to `output/[TICKER] [START DATE].pine` using the Write tool
> 6. Return ONLY a one-line status: `Wrote [N] lines to output/[TICKER] [START DATE].pine — OK`
>
> The main agent receives only that status string — not the script source. Do not re-derive or re-apply any rules in the main context.

> **SUBAGENT DELEGATION — INTEGRATED VALIDATION SCAN:**
> Delegate the validation scan to a subagent.
> The subagent must:
> 1. Execute the `/pinescript-validation-passes` skill (`.claude/skills/pinescript-validation-passes.md`) in full
> 2. Read `output/[TICKER] [START DATE].pine` directly from disk — do NOT receive the script as prompt text
> 3. Perform the single integrated scan across all categories (A — Syntax, B — Type Safety, C — Logic, D — Coordinate Scale, E — Array Bounds)
> 4. Apply all fixes in-place using `replace_string_in_file` directly on the `.pine` file — do NOT return corrected code as text
> 5. Return ONLY a one-line status: `Validated output/[TICKER] [START DATE].pine — [N] fixes applied` (or `0 fixes` if clean)
>
> The main agent receives only that status string — not the corrected script. Do not re-run or re-check any validation rules in the main context.

---

### FINAL OUTPUT

After the validation scan completes, output the following on separate lines:
  `Done -- [TICKER] [START DATE].pine`
  `⏱ End: HH:MM:SS`
  `⏱ Total: X min Y sec`
  (Calculate total by subtracting the Start time from the End time)

Do not output the Pine Script in the chat window. Do not attach it as a fenced code block or artifact. The file was already written to `output/[TICKER] [START DATE].pine`.