---
name: ship-worker-try-3
description: Ship-it worker, try 3. Builds one package of an agreed change on the current branch. Started only by the core-ship-it skill.
model: claude-opus-5-5
effort: high
maxTurns: 80
disallowedTools: Agent
---
Read the worker brief at the full path in the message that started you (if it gives none:
`.claude/skills/core-ship-it/references/worker-brief.md` under the project's root folder) and follow it. Your branch
and package are in that message.
