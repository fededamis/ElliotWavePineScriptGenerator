---
name: optimize-prompt
description: Analyze a target prompt for runtime performance and token budget, identify weak spots, and produce a prioritized optimization plan with concrete rewrite strategies.
---

## PROMPT OPTIMIZATION SKILL

Given a **TARGET PROMPT** (a `.prompt.md` file or inline prompt text), perform the analysis below and output a structured optimization report.

---

### PHASE 1 — INVENTORY & TOKEN BUDGET BREAKDOWN

Read the target prompt and every file it references (skills, validation passes, generation rules, etc.).

For each component, record:

| Component | Source File | Approx. Tokens | % of Total |
|-----------|-------------|---------------|------------|
| System preamble / role definition | | | |
| Cache check logic | | | |
| Input collection / routing logic | | | |
| Subagent delegation blocks | | | |
| Each referenced skill file (list individually) | | | |
| Output / file-writing instructions | | | |
| **TOTAL** | | | **100%** |

Estimate token counts using ~4 characters per token as a baseline. Flag any component exceeding **15% of total budget** as a **High-Cost Component**.

---

### PHASE 2 — AVERAGE RUNTIME ANALYSIS

Model the execution path as a sequence of phases. For each phase estimate the number of LLM calls, tool calls, and approximate latency contribution:

```
PHASE MAP
─────────────────────────────────────────────────────────────────
Phase 0  │ User input collection          │ 0 LLM calls, 0 tool calls
Phase 1  │ Cache / state check            │ 0 LLM, N tool calls (Read)
Phase 2  │ Input validation / routing     │ 0 LLM, 0 tool (logic only)
Phase 3  │ Primary analysis subagent      │ 1 LLM call, N tool calls
Phase 4  │ Intermediate output write      │ 0 LLM, 1 tool (Write)
Phase 5  │ Generation subagent            │ 1 LLM call, 1 tool (Write)
Phase 6  │ Validation subagent            │ 1 LLM call, N tool calls
Phase 7  │ Final output                   │ 0 LLM, 0 tool
─────────────────────────────────────────────────────────────────
TOTAL: variable LLM calls, variable tool calls
```

Adapt the phase map to the actual prompt structure.

Identify the **Critical Path** — the sequence of phases that cannot be parallelized and dominates total latency. Highlight any phase where:
- Multiple sequential external calls could be batched or cached
- A subagent receives more context than it needs to produce its output
- The main agent re-processes or re-echoes content already produced by a subagent

---

### PHASE 3 — WEAK SPOT IDENTIFICATION

Scan the target prompt and all referenced skill files for the following anti-patterns. For each issue found, record: **location** (file + section), **type**, **severity** (Critical / High / Medium / Low), and **description**.

#### 3A — TOKEN WASTE PATTERNS
- **Redundant repetition**: Rules or constraints stated in both the main prompt AND a skill file (double-loaded)
- **Verbose gate descriptions**: Multi-check gates that appear in full in a skill but are also repeated or summarized in the main prompt unnecessarily
- **Over-specified output formats**: Full markdown table headers repeated inline that could be delegated entirely to a subagent skill
- **Inline rule duplication**: Rules defined in one skill file that also appear verbatim in another (e.g. generation rules restated in validation rules)
- **Dead instructions**: Instructions for behaviors already guaranteed by a lower-level skill (e.g. the main prompt restating validation rules when the validation subagent already runs the full skill)

#### 3B — RUNTIME INEFFICIENCY PATTERNS
- **Sequential fetch loops**: External API calls performed one at a time when a single bulk call could cover all requests
- **Subagent over-context**: A subagent receives full upstream artifacts AND full rule text when only the artifacts are needed (rules are already embedded in the skill it executes)
- **Main-agent re-derivation**: The main prompt instructs the main agent to re-check or re-apply rules that the subagent already ran — unnecessary token spend and latency
- **Blocking sequential subagents when parallelism is possible**: Phases that could run concurrently but are serialized by prompt ordering

#### 3C — RELIABILITY / CORRECTNESS RISKS
- **Ambiguous fallback paths**: Instructions that say "if X fails, try Y" without specifying what constitutes failure or what state to restore
- **Implicit assumptions about subagent output format**: Main agent parses subagent output with no schema validation — if the subagent deviates from the expected format, parsing silently fails
- **Gate enforcement gaps**: Checks defined but no explicit mechanism to block continuation if they fail (relying on LLM compliance rather than hard structural stops)
- **Accumulated arithmetic drift**: Values computed with repeated offset arithmetic — rounding errors compound over many iterations
- **Model routing fragility**: Subagent delegation specifies a model via advisory text only — model version drift could silently degrade quality

#### 3D — MAINTAINABILITY RISKS
- **Cross-file rule fragmentation**: A single logical rule defined in multiple files — if one is updated, the others drift
- **Skill boundary ambiguity**: Overlapping responsibility between two skill files with unclear authoritative source
- **Bug fix protocol scope gaps**: A fix protocol that doesn't specify which skill file owns which rules, making it hard to locate the right file to patch

---

### PHASE 4 — PRIORITIZED OPTIMIZATION STRATEGIES

For each weak spot identified in Phase 3, provide a concrete mitigation strategy. Group by impact tier:

#### TIER 1 — HIGH IMPACT (implement first)

**T1-A: Eliminate cross-file rule duplication**
- Audit all skill files for verbatim or near-verbatim rule overlap
- For each duplicated rule: keep the authoritative version in the primary source file, replace the secondary copy with a single-line reference to the authoritative file
- Estimated savings: depends on duplication density

**T1-B: Remove main-prompt restatement of subagent rules**
- Strip any instruction in the main prompt that merely repeats a rule already enforced by the delegated skill
- Estimated savings: depends on restatement density

**T1-C: Consolidate repeated gate / checklist definitions**
- Define multi-check gates ONCE at the top of the relevant skill file; replace all inline repetitions with a back-reference (e.g. "Re-apply the [GATE NAME] defined above")
- Estimated savings: depends on gate size and repetition count

#### TIER 2 — MEDIUM IMPACT

**T2-A: Batch external API fetches**
- Fetch the entire required data range in ONE API call at the start of the relevant phase; store the response in memory; all subsequent lookups index into this cached response — do NOT make additional calls for individual items
- Collapses N external calls into 1, reducing latency proportionally

**T2-B: Make subagent output schema explicit**
- Add a compact schema definition (JSON or table) at the top of each subagent's output format section, defining exact column names, types, and allowed values
- The consuming agent should validate the received data against this schema before proceeding — if any required field is missing, issue a HARD STOP
- Eliminates silent parse failures

**T2-C: Add a pre-delegation input validation step**
- Before launching any subagent, the main agent should verify required inputs are present and valid (non-empty strings, correct formats, no future dates if applicable)
- If validation fails, prompt the user immediately without launching any subagent — avoids a full LLM call on bad input

#### TIER 3 — LOW IMPACT / MAINTAINABILITY

**T3-A: Establish a single authoritative file per rule domain**
- Document in `README.md` which file owns which rule domain
- All other files reference the authoritative file rather than restating rules

**T3-B: Strengthen model routing**
- Replace advisory model-routing text with a structural parameter in the subagent delegation block if the framework supports it
- If structural enforcement isn't available, add a fallback assertion: "If this subagent was not run on the required model class, output a HARD STOP: MODEL ROUTING FAILURE — re-run on the correct model"

**T3-C: Standardize repeated arithmetic patterns**
- Replace accumulated offset arithmetic with a named constant defined once at the top of the relevant section
- Prevents compounding rounding errors in long sequences

---

### PHASE 5 — REWRITE TARGETS

List every file that should be modified to implement the above strategies, in recommended edit order. For each file, summarize which optimizations apply.

Example format:
1. `primary-analysis-skill.md` — consolidate gate (T1-C), batch fetch rule (T2-A), output schema (T2-B)
2. `validation-skill.md` — remove rules duplicated from generation file (T1-A)
3. `generation-rules.md` — confirm as authoritative source; remove content that belongs in validation only
4. `main.prompt.md` — strip restatements of subagent rules (T1-B), add input validation block (T2-C), strengthen model routing (T3-B)
5. `README.md` — add rule ownership table (T3-A)

---

### OUTPUT FORMAT

After completing all five phases, output the following in order:

1. **Token Budget Table** (Phase 1)
2. **Runtime Phase Map** (Phase 2) with Critical Path highlighted
3. **Weak Spot List** (Phase 3) — table format: File | Section | Type | Severity | Description
4. **Optimization Strategies** (Phase 4) — grouped by tier, with estimated token savings where applicable
5. **Rewrite Target List** (Phase 5) — ordered file list with one-line summary of changes

Do not apply any changes to the files during this skill. Output the analysis only. If the user confirms they want changes applied, proceed with edits in the order specified in Phase 5.
