---
name: context-optimizer
description: "Improves context window efficiency by ranking, deduplicating, and compressing documents. Use when: a prompt or document is too long, context window is filling up, you need to reduce token usage without losing information, you want to trim boilerplate or redundant content from instructions/prompts."
argument-hint: "Paste or reference the document to optimize, and describe the task it must serve."
---

# Context Optimizer

Reduces token usage in documents while preserving all facts and logic. Runs three sequential passes: Relevance Ranking, Redundancy Detection, and Compression.

## When to Use
- A prompt file, skill, or instruction document is growing too large
- Context window pressure is causing truncation or degraded output
- You want to audit a document for bloat before publishing or sharing
- A user asks to "trim", "compress", "optimize", or "reduce" a document

## Inputs Required
1. **Document** — the text or file to optimize
2. **Task description** — what the document must support (determines relevance threshold)

## Procedure

### Pass 1 — Relevance Ranker
1. Read the task description carefully.
2. Split the document into logical sections (headings, paragraphs, list blocks).
3. Score each section 0–1 for relevance to the task:
   - `1.0` — directly required to complete the task
   - `0.5–0.9` — supporting context that aids correctness
   - `0.1–0.4` — tangential; useful only in edge cases
   - `0.0` — unrelated, introductory filler, or purely cosmetic
4. Drop all sections scoring below **0.4** (adjustable threshold).
5. Output a keep/cut decision table with one-line justification per section.

### Pass 2 — Redundancy Detector
1. Scan kept sections for:
   - Duplicate facts stated in multiple places
   - Overlapping examples that test the same concept
   - Boilerplate headers or transition sentences with no informational content
2. For each duplicate cluster, nominate the **single best instance** to keep.
3. Merge complementary but non-overlapping fragments into one consolidated block.
4. Output a list of merges and removals with brief rationale.

### Pass 3 — Compressor
1. Take the surviving, deduplicated sections.
2. For each section, rewrite using these rules:
   - Remove filler phrases ("It is important to note that…", "As mentioned above…")
   - Convert prose explanations of lists into bullet points
   - Replace wordy qualifications with precise conditionals (`if X → Y`)
   - Preserve every distinct fact, rule, constraint, and example — nothing substantive may be dropped
3. Target ≥30% token reduction vs. the pre-compression section length.
4. Flag any section where compression would require dropping a fact (do not drop; flag instead).

## Output Format

```
## Relevance Decisions
| Section | Score | Decision | Reason |
|---------|-------|----------|--------|

## Redundancy Removals
- [Section A ¶2] merged into [Section B ¶1]: both define X
- [Section C header] removed: zero informational content

## Compressed Document
<full optimized text here>

## Token Summary
Original: ~N tokens | Optimized: ~M tokens | Reduction: P%
```

## Thresholds (defaults, override if user specifies)
| Parameter | Default |
|-----------|---------|
| Relevance cutoff | 0.4 |
| Target compression | ≥30% |
| Fact-drop policy | Flag, never drop |
