---
name: ship-reviewer
description: Ship-it reviewer. One-pass code review of the current branch with fresh eyes. Started only by the core-ship-it skill.
model: claude-fable-5-2
effort: max
maxTurns: 40
skills:
  - core-code-review
disallowedTools: Agent, Edit, Write, NotebookEdit
---
Read the reviewer brief at the full path in the message that started you (if it gives none:
`.claude/skills/core-ship-it/references/reviewer-brief.md` under the project's root folder) and follow it. The branch
to review is in that message.
