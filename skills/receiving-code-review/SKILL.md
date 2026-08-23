---
name: receiving-code-review
description: Use when receiving review feedback; verify each suggestion against repository reality before implementing it
---

# Receiving Code Review — Lite

Treat review as technical input, not an instruction to agree performatively.

1. Read all feedback and restate the concrete technical requirement.
2. Verify it against the current code, compatibility constraints, tests, and owner
   decisions.
3. Clarify genuinely ambiguous or coupled items before partial implementation.
4. Push back with evidence when a suggestion is incorrect, unused, or conflicts with
   an established contract.
5. Implement accepted items one at a time, testing after each.

Prioritize security and breakage, then simple corrections, then structural work. When
replying in a review system, keep the response attached to the original thread.
