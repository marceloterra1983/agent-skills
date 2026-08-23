---
name: test-driven-development
description: Use for behavior changes; write and observe a failing test before implementation, then make the smallest passing change
---

# Test-Driven Development — Lite

Use red, green, refactor for every behavior change.

1. Write one focused test that expresses the desired behavior.
2. Run it and confirm it fails for the intended missing behavior, not a setup error.
3. Implement the smallest production change that can pass.
4. Run the focused test and the relevant regression suite.
5. Refactor only while the suite remains green, then repeat for the next behavior.

A test that was never observed failing does not prove it can detect the defect. Prefer
real behavior over mocks, clear names over generic assertions, and production-facing
interfaces over test-only hooks. Generated artifacts and configuration-only changes
may use contract validation instead of a unit red/green cycle.
