---
name: core-ship-it
description: The user says "ship it" (or "ship this") about an agreed change or a brainstormed plan. Write the plan, then a fresh coordinator builds it in rounds with cheaper-model workers in parallel, a fresh reviewer per round, and merges. Use only on those words from the user.
---

# Ship it

Two roles. The thread the user said "ship it" in writes the plan and starts a fresh coordinator (Part A). The
coordinator runs the plan (Part B): it thinks, splits, checks and decides; workers and the reviewer are separate
agents, and they type. Nobody asks the user what can run in parallel, which model to use, or whether to start the next
round.

## Where a ship-it lives
Every ship-it gets its own copy of the repo (a git worktree), `<home>\ship-it\<project>-<short-name>` (the copy), where
`<home>` is the user's home folder (`%USERPROFILE%` on Windows, `~` elsewhere). Its plan and brief files go in the
copy's `.ship-it\` folder. Nothing goes in the project's own folder, so two ship-its in one repo never touch each
other's files or branches. Everyone in a ship-it works only in its copy: `cd` there first, and name files by full path.
The copy is deleted when it's done.

## Part A: the thread the user said "ship it" in
1. Make the copy, from the project's folder: `git fetch origin main`, then
   `git worktree add -b ship/<short-name>-r1 <home>\ship-it\<project>-<short-name> origin/main`.
   If that folder already exists, another ship-it has the name: pick another.
   Keep `.ship-it/` out of git: if the project's `.git\info\exclude` doesn't list `.ship-it/`, add that line (this
   file is local and never committed).
2. Write the plan to `<copy>\.ship-it\plan.md`. It is all the coordinator will know, so put in: the goal; what was
   decided and why; what is out of scope; and the rounds. A round is a piece that works and can merge on its own, at
   most 6 worker jobs; later rounds build on earlier ones. A small change is one round. Also:
   - Anything the plan says must keep working: check that today's code really does it, and say how you checked. If
     you couldn't check, write "unverified" next to it; the coordinator checks it before anything depends on it.
   - Anything nobody knows how to do yet (will this approach work at all?) is a question, not a build job. Mark it
     "question" and say which jobs need its answer.
   End the file with an empty `## Progress` section.
3. Inside BB (`BB_THREAD_ID` is set), start the coordinator as your child thread. Leave out `--provider`, `--model`
   and `--reasoning-level`: a child with none of them inherits yours, and coordinating is thinking work. Write
   `<copy>\.ship-it\coordinator.md`:
   `Read <full path of this SKILL.md> and follow Part B as the coordinator. Your copy: <copy path>. The plan:
   <copy>\.ship-it\plan.md. If the plan leaves something unclear, read @thread:<your BB_THREAD_ID>; ask the user only
   if that doesn't settle it.`
   Then: `bb thread spawn --parent-self --project $BB_PROJECT_ID --environment $BB_ENVIRONMENT_ID --title "ship-it coordinator: <goal>" --prompt-file <copy>\.ship-it\coordinator.md --json`
   (PowerShell spells the variables `$env:BB_PROJECT_ID`.)
4. Tell the user in one line that the coordinator is running, then end your turn. BB wakes you when it finishes, fails
   or gets stuck. Then: relay its report in 3 lines; if it asks for a fresh coordinator, start one the same way with
   the same coordinator.md; if it failed, say what is blocking.
5. Outside BB you are the coordinator: do Part B yourself.

## Part B: the coordinator
Don't write the code yourself. `cd` to the copy. Do the plan's rounds in order, each one through steps 1 to 4.

### 1. Set up
1. Round 1's branch already exists in the copy. Each later round: `git fetch origin main`, then
   `git switch -c ship/<short-name>-r<round> origin/main`, in the copy. Workers share the copy and its branch.
2. Round 1 only: a new copy has no installed packages and no local config files. Install what the checks need, the way
   the project's README or package file says (e.g. `npm ci`). Copy any untracked config file the checks need (such as
   `.env.local`) from the project's folder into the copy. Never print what's in them.
3. Anything marked "unverified" in the plan: check it in today's code now. If it isn't true, the plan can't promise to
   keep it: drop that requirement and say so in your report.
4. Split the round into 1 to 6 packages. A package is one worker's job and says: the goal, the files it owns, and the
   check that proves it works (a test or command and what it should print). Two packages never own the same file. If
   they would have to, make them one package, or run the second after the first. A question from the plan is its own
   package (Build, step 4).
5. If nothing checks the change, a check (a test, or a command the user could run) is part of the first package.

### 2. Build
1. Start one worker per package, all at once (Delegation, below), each on try 1 of the table. Each gets the brief, the
   copy's path, the branch name and its package.
2. Wait for all of them, then read each report and run each package's check yourself, once.
3. A worker that failed or drifted off its package: start a FRESH worker one try further down the table, with a
   tighter package and the first one's handoff note. Move down on your own; never ask the user. A worker that only
   handed off because its thread got long hasn't failed: its replacement stays on the same try. Don't nudge a running
   worker along, and don't fix its work yourself.
4. A question package starts on try 3. Its job: try at least two designs, run each, and report which works, with what
   the runs printed. Packages that don't need its answer build alongside it; the ones that do wait for it.
5. A package is stuck when its try-3 worker fails, or a question finds no design that works. It holds back only the
   packages that need it: build and ship the rest, and put the stuck one in your report with what was tried.
6. Commit everything on the branch: one commit per package, plain-English message.

### 3. Review
1. Start ONE reviewer for the whole branch (Delegation). It has seen none of the above; don't tell it what you expect.
   Its message gives the copy's path and the full path of `core-code-review`'s SKILL.md in the project's folder.
2. Findings marked Critical or Important: a fresh worker per finding (one for all of them if they're small), given the
   finding and the reviewer's fix line. Run the checks once, commit. One round. A second review only if the user asks.
3. A finding you disagree with: leave it, and say so in your report.

### 4. Ship the round
In the copy: `git push -u origin <branch>`, open a pull request (`gh pr create --fill`), and merge it once it is
mergeable with no failing checks (`gh pr merge --squash --delete-branch`, or the merge style the project uses). Never
push straight to main and never force-push. Then update main in the project's own folder: if it has main checked out
and no uncommitted changes, `git pull --ff-only`; otherwise leave it and say so in your report. Add a line under
`## Progress` in the plan: the round, its PR link, anything skipped or stuck. Then start the next round without asking
anyone.

Stop and report instead of starting the next round only when: a round can't merge (its checks still fail after the fix
round), the reviewer found something Critical you couldn't fix, a stuck package that every remaining round needs, or
the plan turned out wrong in a way only the user can decide. Leave the copy in place when you stop: a fresh coordinator
picks up from its plan.

After the last round: from the project's folder, `git worktree remove --force <copy>`, then `git worktree prune` (if
the remove fails, delete the copy's folder, then prune). That deletes the plan too. Report in 3 lines: what shipped,
the PR links, anything stuck, left or skipped.

## Delegation

**Inside BB** (`BB_THREAD_ID` is set): workers and the reviewer are child threads of your thread, nested under it in
the user's sidebar, so they can watch and stop any of them.
1. Write the brief to a file, `<copy>\.ship-it\worker-1.md`. Line 1: `Read <full path> and follow it.` where the full
   path is this skill's folder (in the project's folder) + `references\worker-brief.md`, for example
   `C:\...\my-app\.agents\skills\core-ship-it\references\worker-brief.md`. Always full paths: a child thread may
   start anywhere in the project. Then the copy's path, the branch name and the package.
2. Start it, with your own provider (`claude-code` or `codex`) and the model and effort for its try from the table
   (always pass both: a child left to default inherits YOUR model and effort):
   `bb thread spawn --parent-self --project $BB_PROJECT_ID --environment $BB_ENVIRONMENT_ID --provider <yours> --model <model> --reasoning-level <effort> --title "ship-it worker 1: <goal>" --prompt-file <copy>\.ship-it\worker-1.md --json`
   Note the `id` it prints. Start all workers before waiting.
3. `bb thread wait <id> --timeout 2h`, then `bb thread output <id>` is the report. A worker still running after that:
   `bb thread stop <id>`, read what it did, start a fresh one.
4. The reviewer the same way, from `<copy>\.ship-it\reviewer.md` with line 1 pointing at `reviewer-brief.md`, title
   "ship-it reviewer".

**Outside BB**: the tool's own subagents, `ship-worker` and `ship-reviewer` (Claude Code: the Agent tool, in
parallel; Codex: spawn agents of those types). Same messages: the brief's full path, the copy's path, the branch name
and the package. Claude Code: for tries 2 and 3 pass model `opus` to the Agent tool (effort can't be set there). Codex:
a try you can't start counts as failed.

| Role | Claude Code model | Codex model | Effort (both) |
|---|---|---|---|
| Coordinator | yours (inherited) | yours (inherited) | yours |
| Worker, try 1 | `sonnet` | newest Luna | `medium` |
| Worker, try 2 | `opus` | newest Sol | `medium` |
| Worker, try 3 (and questions) | `opus` | newest Sol | `xhigh` |
| Reviewer | `opus` | newest Sol | `high` |

Always use the newest Luna and Sol, the way `sonnet` and `opus` always mean the newest Sonnet and Opus (Claude Code
moves those names itself). Codex has no such names, so you pick: once per ship-it, run
`bb provider models codex --json` and take the `gpt-<version>-luna` and `gpt-<version>-sol` with the highest version
(plain names only: skip any with more after `luna`/`sol`, such as `-pro`). On 2026-11-06 that was `gpt-6-luna` and
`gpt-6.1-sol`; if yours is newer, say so in your report.

The subagent route (`agents/`, outside BB) has fixed model names in its files; update them when a newer model ships.
If bb refuses a model name or effort, run `bb provider models <provider>` once, use the matching tier (for `xhigh`,
the level just above `high`), and tell the user.

## Long threads
- Nobody compacts: it loses the plan.
- Coordinator: after a round, if your thread is getting long (`bb thread context --self`), check that `## Progress` is
  up to date and end your turn with "Fresh coordinator needed after round <N>." Your parent starts one from the same
  coordinator.md. Outside BB, tell the user to start a new thread and point it at this skill, the copy and the plan.
- Workers stop and hand off too (their brief says how); you start a fresh one.
