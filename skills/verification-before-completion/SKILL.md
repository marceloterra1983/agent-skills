---
name: verification-before-completion
description: Use before claiming success, committing, publishing, or handing off; fresh evidence must support every completion claim
---

# Verification Before Completion — Lite

Evidence precedes every success claim.

Before claiming a change is complete:

1. Identify the exact command or observation that proves the claim.
2. Run the full check freshly in the correct environment.
3. Read the complete result, exit code, and failure count.
4. Compare the evidence to the requirement, not merely to the implementation.
5. Report the verified result and any unverified limitation accurately.

Lint does not prove a build, a build does not prove behavior, and a passing test does
not prove every requirement. For regression tests, verify the red and green states.
Never substitute confidence, old output, partial checks, or another agent's report for
fresh evidence.
