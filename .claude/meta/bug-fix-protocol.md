---
name: bug-fix-protocol
description: Apply a bug fix or adjustment to both the output file (.pine or .wave) AND the source prompt, so re-executing the prompt will not reproduce the same error.
---

When applying any bug fix or adjustment to a `.pine` or `.wave` output file:

1. **Fix the output file** — update the `.pine` or `.wave` file directly with the corrected logic.
2. **Fix the prompt source** — locate the section in the prompt file (or in any referenced validation/wave skill files) that produced the incorrect behavior and update it so that re-executing the prompt will not reproduce the same error. Any text added or modified in the prompt must explicitly enforce EW rules — it must not permit, omit, or loosen any of the three absolute EW rules or the Degree Proportion Rules defined in step 3.

Do not treat a fix as complete until both the output file and the prompt source have been updated (or the exemption above is documented).

3. **Enforce Elliott Wave rules** — after every fix, re-verify that the full pivot set in the output still satisfies **all three absolute EW rules** (at every wave degree present, including subwaves):
   - Wave 2 never retraces more than 100% of Wave 1
   - Wave 3 is never the shortest among Waves 1, 3, and 5
   - Wave 4 never overlaps Wave 1's price territory (except in diagonals)

   Also re-verify the **Degree Proportion Rules**:
   - No wave spans ≤4% or ≥86% of the total sequence duration
   - No wave is more than 10× the duration of any sibling wave

   If any rule is violated after the fix, the fix is incomplete — correct the pivot set before proceeding. A fix that resolves one bug while introducing an EW rule violation is **not acceptable**.

4. **Optimize for tokens** — fixes must not increase token count unnecessarily. Prefer concise rewrites: remove redundant comments, collapse verbose logic, reuse existing variables. If a fix requires new code, offset it by trimming equivalent dead weight elsewhere. Apply token trimming only within the sections touched by the fix, not as a global pass.
