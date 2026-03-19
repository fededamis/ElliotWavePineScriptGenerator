# Prompt Optimization Report
### Target: `Elliot Wave TradingView PineScript.prompt.md`

---

## PHASE 1 — TOKEN BUDGET BREAKDOWN

| Component | Source File | Approx. Tokens | % of Total |
|-----------|-------------|---------------|------------|
| System preamble / role definition | main prompt L1 | ~15 | 0.4% |
| Execution timer | main prompt L3–5 | ~25 | 0.6% |
| Input collection (TICKER, START DATE) | main prompt L7–11 | ~50 | 1.2% |
| Cache check logic | main prompt L15–29 | ~180 | 4.4% |
| Timeframe selection | main prompt L33–43 | ~120 | 2.9% |
| Subagent delegation — Wave Analysis | main prompt L51–58 | ~130 | 3.2% |
| Output rule + Subagent delegation — Generation | main prompt L60–70 | ~140 | 3.4% |
| Subagent delegation — Validation | main prompt L74–83 | ~130 | 3.2% |
| Output/file-writing instructions | main prompt L91–102 | ~110 | 2.7% |
| **elliott-wave-analysis.md** | skill file | **~950** | **23.1%** ⚠️ HIGH |
| **pinescript-validation-passes.md** | skill file | **~550** | **13.4%** ⚠️ HIGH |
| pinescript-generation-rules.md | skill file | ~250 | 6.1% |
| pinescript-visual-style.md | skill file | ~180 | 4.4% |
| optimize-prompt.md | skill file | ~650 | 15.8% ⚠️ HIGH |
| bug-fix-protocol.md | skill file | ~50 | 1.2% |
| **TOTAL** | | **~3,530** | **100%** |

**High-Cost Components (>15% of budget):**
- `elliott-wave-analysis.md` — 23.1%
- `optimize-prompt.md` — 15.8% *(meta-skill, not in main execution path)*
- `pinescript-validation-passes.md` — 13.4% *(just under threshold)*

---

## PHASE 2 — RUNTIME PHASE MAP

```
PHASE MAP
─────────────────────────────────────────────────────────────────────────
Phase 0  │ User input (TICKER + START DATE)          │ 0 LLM, 0 tool
Phase 1  │ Cache check (Read .wave file)             │ 0 LLM, 1 tool (Read)
Phase 1a │  [HIT]  → skip to Phase 3                │ 0 LLM, 0 tool
Phase 1b │  [MISS] → proceed to Phase 2              │ 0 LLM, 0 tool
Phase 2  │ Elliott Wave Analysis subagent            │ 1 LLM (Opus), N WebFetch
         │   Step 1–4: fetch OHLC, identify pivots   │
         │   Step 4.5: subwave identification         │
         │   Step 5–8: counts, projections, gates     │
Phase 2w │ Write .wave cache file                    │ 0 LLM, 1 tool (Write)
Phase 3  │ Pine Script Generation subagent           │ 1 LLM, 1 tool (Write)
Phase 4  │ Validation subagent                       │ 1 LLM, 1 tool (Read + Write)
Phase 5  │ Final output (timer + filename)           │ 0 LLM, 0 tool
─────────────────────────────────────────────────────────────────────────
TOTAL (cache miss): 3 LLM calls, N+3 tool calls
TOTAL (cache hit):  2 LLM calls, 3 tool calls
```

**Critical Path (cache miss):** Phase 2 → Phase 2w → Phase 3 → Phase 4
All three LLM subagent calls are strictly sequential — no parallelization possible given data dependencies.

**Hotspots:**
- Phase 2 dominates: multiple sequential WebFetch calls (one per fetch + potential fallback), plus the full `elliott-wave-analysis.md` rule set (~950 tokens) injected into the subagent context
- Phase 3 receives the full pivot table AND loads two full skill files (`pinescript-generation-rules.md` + `pinescript-visual-style.md`)
- Phase 4 re-reads the generated `.pine` file from disk, then loads the full `pinescript-validation-passes.md` (~550 tokens)

---

## PHASE 3 — WEAK SPOT IDENTIFICATION

| # | File | Section | Type | Severity | Description |
|---|------|---------|------|----------|-------------|
| 1 | `pinescript-generation-rules.md` L21 vs `pinescript-visual-style.md` L32 | `size=size.small` | 3A — Rule duplication | **High** | `size=size.small` CRITICAL rule stated verbatim in both generation rules (L21) and visual style (L32) |
| 2 | `pinescript-generation-rules.md` L22 vs `pinescript-visual-style.md` L43 | `textalign` | 3A — Rule duplication | **High** | `textalign=text.align_center` stated as CRITICAL in both files. Note: generation rules says `text.align_center` while validation pass D says `text.align_left` — **this is a direct contradiction** |
| 3 | `pinescript-validation-passes.md` D L112 vs `pinescript-generation-rules.md` L22 | `textalign` | 3C — Correctness risk | **Critical** | Generation rules L22 says `NEVER text.align_left`; visual style L43 says `text.align_center`; but validation pass D L112 says `textalign=text.align_left`. Three-way contradiction. The generated `.pine` file uses `text.align_left` throughout — meaning the visual style rule is being silently ignored or overridden by validation. |
| 4 | `pinescript-generation-rules.md` L23 vs `pinescript-visual-style.md` L44 | Label style (up/down) | 3A — Rule duplication | **Medium** | Label style (up for lows, down for highs) is stated in both files |
| 5 | `pinescript-generation-rules.md` L21 vs `pinescript-validation-passes.md` D L111 | `size=size.small` | 3A — Rule duplication | **Medium** | `size=size.small` rule repeated in generation rules AND validation scan D |
| 6 | `main prompt` L60 + L92 | OUTPUT RULE | 3A — Redundant repetition | **Low** | "Do not output the script as chat text" stated twice: once inline before generation subagent (L60) and once in the OUTPUT section (L92) |
| 7 | `elliott-wave-analysis.md` L93–101 | STEP 8 HARD STOP | 3B — Subagent over-context | **Medium** | The 5 HARD STOP checks are defined in the wave analysis skill AND implicitly re-enforced by the main prompt's subagent delegation instruction at L55 ("Execute the skill in full, including all 8 steps… and the PIVOT ACCEPTANCE GATE"). The main prompt adding "all 8 steps" is unnecessary since the skill defines what those steps are. |
| 8 | `main prompt` L55 | Subagent delegation | 3B — Main-agent re-derivation | **Medium** | Main prompt itemizes the subagent's steps ("data fetching, pivot identification, Fibonacci validation, and the PIVOT ACCEPTANCE GATE") — these are already fully defined in the skill. This redundant enumeration in the main prompt wastes tokens and creates a maintenance drift risk. |
| 9 | `pinescript-generation-rules.md` | Full file | 3B — Subagent over-context | **Low** | Generation subagent receives both `pinescript-generation-rules.md` and `pinescript-visual-style.md` in full. Visual style file restates 3 rules from generation rules (size, textalign, label style). Net: ~30 tokens of pure duplication injected per generation call. |
| 10 | `elliott-wave-analysis.md` L163–194 | VALIDATION AT THREE STAGES | 3D — Cross-file rule fragmentation | **Medium** | The PIVOT ACCEPTANCE GATE (6 checks) is defined once at L150–161, then re-stated as "Stage 1", referenced again in "Stage 2", and re-run as "Stage 3". This triple application of the same gate within one file inflates the skill by ~150 tokens. |
| 11 | `main prompt` L87 | Bug fix comment | 3D — Maintainability | **Low** | `<!-- Bug fix protocol moved to skill: bug-fix-protocol -->` — this HTML comment is dead weight in the main prompt context; it adds no runtime value and the skill is already self-documented. |
| 12 | `pinescript-validation-passes.md` L112 | `textalign=text.align_left` | 3C — Reliability risk | **High** | Validation scan D actively enforces `text.align_left` — but generation rules and visual style both mandate `text.align_center`. Whichever runs last "wins", creating non-deterministic output depending on execution order. This is a latent bug that silently overrides intended visual style on every run. |
| 13 | `elliott-wave-analysis.md` L109–113 | PRE-ANALYSIS CHECKLIST | 3B — Subagent over-context | **Low** | The checkbox-formatted PRE-ANALYSIS CHECKLIST at L109–113 is a formatting artifact — the subagent won't interact with checkboxes; the same constraints are already enforced by the PIVOT ACCEPTANCE GATE and AUTONOMOUS FETCH RULE. ~30 tokens of redundant instruction. |
| 14 | `pinescript-generation-rules.md` L42 | Legend confidence % | 3C — Implicit output coupling | **Medium** | Rule says to read confidence % from `.wave` file headers — but there is no explicit schema for how the `.wave` file header line is formatted (it's defined as part of the compact output in `elliott-wave-analysis.md`). If the wave analysis skill ever changes the header format, the generation rule silently produces wrong legend text. |

---

## PHASE 4 — PRIORITIZED OPTIMIZATION STRATEGIES

### TIER 1 — HIGH IMPACT

**T1-A: Resolve the `textalign` three-way contradiction** *(Issues #2, #3, #12)* ✅ SOLVED

This is the most impactful fix because it's an active bug, not just inefficiency.

- **Decide the canonical value**: `text.align_left` (current `.pine` output) or `text.align_center` (generation rules + visual style intent)
- **Set it once** in `pinescript-visual-style.md` as the authoritative source
- **Remove** the contradicting statement from `pinescript-generation-rules.md` L22
- **Update** `pinescript-validation-passes.md` D L112 to match the canonical value
- Estimated fix: eliminates silent per-run style override; ensures deterministic output

**T1-B: Eliminate cross-file `size=size.small` and label style duplication** *(Issues #1, #4, #5)* ✅ SOLVED

- Keep authoritative label style rules in `pinescript-visual-style.md` only
- In `pinescript-generation-rules.md`: replace the three duplicated label rules (size, textalign, style) with a single line: *"For all label style, size, and textalign constraints — see `pinescript-visual-style.md` (authoritative source)"*
- In `pinescript-validation-passes.md` D: remove the `size=size.small` and `textalign` checks (they're enforced at generation time by the authoritative source); keep only the coordinate-related checks unique to validation
- Estimated savings: ~60–80 tokens removed from cross-file duplication

**T1-C: Consolidate PIVOT ACCEPTANCE GATE triple-application** *(Issue #10)* ✅ SOLVED

- In `elliott-wave-analysis.md`: define the gate ONCE (it already is, at L150–161)
- Replace the Stage 1 / Stage 2 / Stage 3 restatements with single-line back-references: *"Re-apply the PIVOT ACCEPTANCE GATE (defined above) to all pivots at this stage"*
- Estimated savings: ~100–120 tokens

---

### TIER 2 — MEDIUM IMPACT

**T2-A: Trim main prompt subagent step enumeration** *(Issues #7, #8)* ✅ SOLVED

- In the main prompt's Wave Analysis delegation block (L55), replace the itemized step list with: *"Execute the `/elliott-wave-analysis` skill in full."*
- The skill already defines its own steps — the main prompt re-listing them is a maintenance liability
- Estimated savings: ~40 tokens; eliminates drift risk

**T2-B: Make the `.wave` file header schema explicit** *(Issue #14)* ✅ SOLVED

- Add a one-line schema comment to `elliott-wave-analysis.md`'s COMPACT OUTPUT FORMAT section: *"Header format (consumed by generation rules): `PRIMARY COUNT (X% confidence)` — the integer X is parsed by the legend rule in `pinescript-generation-rules.md`"*
- This makes the coupling explicit and survives future format changes
- Cost: ~20 tokens; prevents silent legend breakage

---

### TIER 3 — LOW IMPACT / MAINTAINABILITY

**T3-A: Remove dead HTML comment** *(Issue #11)* ✅ SOLVED

- Deleted `<!-- Bug fix protocol moved to skill: bug-fix-protocol -->` from main prompt; ~8 tokens saved

**T3-B: Remove duplicate OUTPUT RULE** *(Issue #6)* ✅ SOLVED

- Removed inline OUTPUT RULE before generation delegation; authoritative copy kept in OUTPUT section; ~15 tokens saved

**T3-C: Remove PRE-ANALYSIS CHECKLIST checkboxes** *(Issue #13)* ✅ SOLVED

- Replaced 4-checkbox list in `elliott-wave-analysis.md` with a single sentence; ~30 tokens saved

**T3-D: Add rule ownership table to README** *(General maintainability)* ✅ SOLVED

- Added rule ownership table to `README.md` documenting which file owns each rule domain

---

## PHASE 5 — REWRITE TARGETS

Ordered by recommended edit sequence:

1. `.claude/skills/pinescript-visual-style.md` — **Establish as authoritative source** for all label style rules (size, textalign, style direction); add explicit canonical `textalign` value (T1-A, T1-B) ✅ DONE

2. `.claude/skills/pinescript-validation-passes.md` — Remove `size=size.small` and `textalign` checks from section D (now owned by visual style); update `textalign` value in remaining D checks to match canonical (T1-A, T1-B) ✅ DONE

3. `.claude/skills/pinescript-generation-rules.md` — Remove duplicated label rules (size, textalign, style direction); replace with single reference to visual style file; remove duplicate OUTPUT RULE (T1-B, T3-B) ✅ DONE

4. `.claude/skills/elliott-wave-analysis.md` — Collapse Stage 1/2/3 restatements of PIVOT ACCEPTANCE GATE to back-references (T1-C); simplify PRE-ANALYSIS CHECKLIST (T3-C); add `.wave` header schema note (T2-B) ✅ DONE

5. `Elliot Wave TradingView PineScript.prompt.md` — Trim Wave Analysis delegation step enumeration (T2-A); remove dead HTML comment (T3-A) ✅ DONE

6. `README.md` — Add rule ownership table (T3-D) ✅ DONE

---