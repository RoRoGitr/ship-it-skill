---
name: core-code-review
description: One-pass code review of a branch before it is merged. Use when the user asks for a code review, or once before opening a pull request that changes code. Not for docs-only changes.
---

# Code review (one pass)

Find the real problems in the changes, report them once, then stop.

## Steps
1. Find everything that changed: the whole branch against main (`git diff main...HEAD`),
   uncommitted changes (`git diff HEAD`) and new files (`git status`).
2. Read each changed file around the change, not just the diff lines.
3. Run the project's existing checks once (tests, build or type check named in its
   README or AGENTS.md). Don't install new tools.
4. Look for, in this order:
   - Critical: crashes, breaks a feature, loses or corrupts data, leaks a secret, security hole.
   - Important: wrong result in a real case, missing error handling where data comes in
     or goes out, a test that passes for the wrong reason.
   - Skip: style, naming, "could be cleaner", anything a formatter or linter would catch.
5. Check each finding before you report it: point to the exact line and say what input
   or situation makes it fail. Drop anything you can't back up.

## Report
- Start with the verdict: "OK to merge" or "Fix first".
- Then the findings, worst first: file:line, what breaks, the fix in one line.
- 10 findings at most. Plain English.

## Rules
- One pass. Don't review your own fixes in a loop.
- Don't change code during the review. If the user asks you to fix the findings, fix them,
  run the checks once, and say what changed. A second review only if the user asks.
- Don't call other review skills, plugins, subagents or outside review tools.
