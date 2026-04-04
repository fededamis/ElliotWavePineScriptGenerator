---
name: bug-fix-protocol
description: Apply a bug fix or adjustment to both the output file (.pine or .wave) AND the source prompt, so re-executing the prompt will not reproduce the same error.
---

When applying any bug fix or adjustment to a `.pine` or `.wave` output file:

1. **Fix the output file** — update the `.pine` or `.wave` file directly with the corrected logic.
2. **Fix the prompt source** — locate the section in the prompt file (or in any referenced validation/wave skill files) that produced the incorrect behavior and update it so that re-executing the prompt will not reproduce the same error.

Do not treat a fix as complete until both the output file and the prompt source have been updated (or the exemption above is documented).

3. **Optimize for tokens** — fixes must not increase token count unnecessarily. Prefer concise rewrites: remove redundant comments, collapse verbose logic, reuse existing variables. If a fix requires new code, offset it by trimming equivalent dead weight elsewhere. Apply token trimming only within the sections touched by the fix, not as a global pass.
