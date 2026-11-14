# Ship it

Say "ship it" to your coding agent about a change you've agreed on. Your current model plans and coordinates.
Claude Code builds with the newest Sonnet at medium effort, retries with Sonnet at extra high, then uses Opus at
high. Codex follows the same steps with Sol, Sol, then Astra. Fresh reviewers and workers for the single fix round
use the newest Fable for Claude or Astra for Codex, both at max effort. The coordinator then ships the round.

## How it works

![Each Ship It step: plan and coordinate on your current model; build with Sonnet/Sol at medium, Sonnet/Sol at extra high, then Opus/Astra at high; review with Fable/Astra at max; one fix round with Fable/Astra at max; ship with the coordinator. Model pairs are Claude/Codex.](docs/ship-it-model-steps.svg)

The current conversation writes the plan. In BB, a fresh coordinator inherits that conversation's model and reasoning
effort, divides each round into up to six packages, and delegates implementation. Independent packages run in parallel;
packages that need the same files run sequentially. The coordinator checks the workers' results and commits their work.

## Which model does each agent use?

| Agent / attempt | Claude Code | Codex | Reasoning effort |
|---|---|---|---|
| Planning conversation | Your current model | Your current model | Your current setting |
| Coordinator | Inherits your model | Inherits your model | Inherits your setting |
| Worker: attempt 1 | Newest Sonnet | Newest Sol | `medium` |
| Fresh worker: attempt 2 | Newest Sonnet | Newest Sol | `xhigh` |
| Fresh worker: attempt 3 | Newest Opus | Newest Astra | `high` |
| Design-question worker | Newest Opus | Newest Astra | `high`; starts at attempt 3 |
| Fresh reviewer | Newest Fable | Newest Astra | `max` |
| Review-fix worker | Newest Fable | Newest Astra | `max`; one fix round |

**Build retries are per package.** A failed or off-scope attempt goes to a fresh worker with the previous handoff and a
tighter brief. A context-only handoff keeps the same attempt level. A package that fails its third attempt is marked
blocked; independent packages can still proceed. Design questions start at the third build tier and compare at least
two designs.

**Review is independent.** One fresh reviewer checks the whole round using the included `core-code-review` skill.
Critical or Important findings go to fresh fix workers at `max` effort for one fix round, without the build's retry
ladder. A context handoff keeps the same fix-worker model and effort. A second review runs only if the user asks.
The coordinator runs the checks and then opens and merges the pull request when it is mergeable with no failing
checks; unresolved Critical issues block shipping. Review and fixing are separate steps in the diagram, even though
they use the same model families and max effort.

**The model names depend on the environment.** In BB, the skill reads `bb provider models <provider> --json` once per
run and selects the newest model within each requested family, with the exact effort for its role. For Codex it uses
plain Sol and Astra IDs, excluding suffix variants such as `-pro`. An unavailable build-worker attempt counts as
failed; an unavailable reviewer or fix worker blocks shipping.

Outside BB, the initiating conversation coordinates. Native presets set both model and effort: `ship-worker`,
`ship-worker-try-2`, `ship-worker-try-3`, `ship-reviewer`, and `ship-fix-worker`. They currently pin Sonnet 5.5, Opus 5.5,
Fable 5.2, GPT-6.1-Sol and GPT-6-Astra for the roles above. Keep those files updated when newer versions in the same
families become available. Select the matching preset for each attempt or role.

**40% context handoffs.** The coordinator checks context after each round; workers check it when running their package's
validation. At the skill's 40% threshold, they write progress or a handoff for a fresh agent, keeping the work resumable.
In BB, usage comes from `bb thread context --self`; outside BB, the skill asks agents to estimate. It instructs agents
not to compact their threads.

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
The files under `skills/` and `agents/` match the current Harness source byte for byte, including line endings, except
for explicit privacy substitutions: personal names, personal workspace paths, a private project example, and a
personal capability statement. Model choices, effort levels, workflow instructions and the 40% handoff rule are
preserved. The README, workflow diagram, license and Git attributes are public-package files.

`<home>` means your home folder. The source skill assumes that `.ship-it/` is excluded from Git and that the project
has global Git publishing rules. Before using this standalone package, add `.ship-it/` to the exclude file returned
by `git rev-parse --git-path info/exclude`, and provide your project's rules for pushing branches, opening and merging
pull requests, and updating local main. Those setup notes stay here so the copied skill remains unchanged.
