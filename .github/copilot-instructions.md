# Elliott Wave PineScript Generator — Workspace Instructions

## Agentic Silence Rule

In any agentic workflow in this workspace, **never narrate intermediate steps in the chat window**. This applies to all tool calls, file reads, cache checks, API fetches, analysis steps, code generation, and validation passes.

**Permitted chat output:**
- Asking the user for required inputs (before work begins)
- One-line status strings explicitly defined by the active prompt
- The final output block defined by the active prompt

**Forbidden at all times during execution:**
- Cache hit/miss reports
- Bar counts, price summaries, or date ranges
- Pivot candidates, Fibonacci values, wave labels, or degree reasoning
- "Working memory" summaries or bullet-point status updates
- Pine Script lines, variable names, or validation fix descriptions
- Any sentence that describes what the agent is about to do, is doing, or just did
- `manage_todo_list` calls after work has started
- Validation scan category labels, error descriptions, fix rationale, or before/after comparisons

All intermediate output goes to files (`tmp/`, `output/`). The chat window receives only the permitted lines above.