# Ship it

Say "ship it" to your coding agent about a change you've agreed on. It writes a plan, then a coordinator builds it in
rounds: cheaper-model workers in parallel, one fresh reviewer per round, then a pull request and merge.

## Install (per project)
Claude Code:
- `skills/core-ship-it` and `skills/core-code-review` go in `<project>/.claude/skills/`
- `agents/claude/*.md` go in `<project>/.claude/agents/`

Codex:
- the same two skill folders go in `<project>/.agents/skills/`
- `agents/codex/*.toml` go in `<project>/.codex/agents/`

## Needs
- git, a GitHub remote with a `main` branch, and the GitHub CLI (`gh`) logged in.
- Works best inside bb (each worker is its own thread you can watch). Outside bb it uses the tool's subagents.
- It merges to `main` on its own once checks pass. Only use it where that's OK.

## Where things go
Each ship-it works in its own copy of the repo under `~/ship-it/` (`%USERPROFILE%\ship-it\` on Windows), and deletes
it when done.

## License
MIT. See [LICENSE](LICENSE).

## Source fidelity
The skill, briefs, review skill, and agent definitions preserve the original files, including line endings,
except for personal details and machine-specific references. Names and personal paths use generic equivalents;
local sync, global Git rules, and harness maintenance references have their necessary instructions included here.
The coordinator and workers retain the original 40% context handoff rule, model choices, and workflow.
