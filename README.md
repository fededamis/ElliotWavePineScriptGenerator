# ElliotWavePineScriptGenerator
Generates a .pine script to be imported into TradingView with Elliot Wave Analysis for a given stock and date range.

## Rule Ownership

| Rule Domain | Authoritative File |
|---|---|
| Elliott Wave methodology (steps, gates, pivot rules) | `.claude/skills/elliott-wave-analysis.md` |
| Label size, textalign, style direction | `.claude/skills/pinescript-visual-style.md` |
| Color scheme, display inputs, script structure | `.claude/skills/pinescript-visual-style.md` |
| Generation constraints (array handling, coord scale) | `.claude/skills/pinescript-generation-rules.md` |
| Post-generation validation passes (A–E) | `.claude/skills/pinescript-validation-passes.md` |
| Bug fix protocol | `.claude/skills/bug-fix-protocol.md` |

When editing a rule, update it **only in the authoritative file** listed above. Do not restate it elsewhere.

## Pending Tasks
- ADD INTERNAL COUNTS ON LARGE TRENDS
- ADD CONFIDENCE ON COUNTS
- RUN OPTIMIZATION SKILL ON PROMPT
- FIX BUG ON SCREENSHOT
- Check for Node or Python installations (numpy)
