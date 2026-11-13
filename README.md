# Flow

A lightweight Claude Code workflow that takes a piece of work from idea to reviewed, tested code. It runs **inside your project, on your current branch**: no board, no worktrees, no extra tools. Everything it produces is saved as markdown files in `docs/flow/<name>/`, so you can stop at any point and continue later, even a month on.

```
/flow:start  ─►  plan (grill-me)  ─►  tasks (a test each)  ─►  review plan + tasks  ─►  you approve
             ─►  build (test first, task by task)  ─►  review code  ─►  staged + commit message  ─►  you commit
```

It's a lighter sibling of the Ticketboard plugin. It keeps the same planning, tasks, test-first building and reviews, but has no board, worktrees, branches or PR stage.

---

## Install
```bash
claude plugin marketplace add zain-waheed-brainx/flow-workflow
```
```bash
claude plugin install flow@flow-workflow
```
Then start a new Claude session in your project.

To try it from source without installing, for one session:
```bash
claude --plugin-dir "/path/to/flow-workflow/plugins/flow"
```

No setup is needed. It only requires git (and the project's own test runner, if it has one).

---

## Skills

| Command | What it does |
|---|---|
| **`/flow:start`** | **The main entry.** Give it details, an existing plan, or existing tasks, or nothing to continue a flow. It runs every step below in order |
| `/flow:plan <name>` | Grill-me planning, one question at a time, or validating an existing plan. Writes `plan.md` and `decisions.md` |
| `/flow:tasks <name>` | Splits the plan into tasks, each with its own test, or imports your tasks. Subagents study the code for bigger work. Writes `tasks.md` |
| `/flow:review <name> plan\|code` | **plan:** reviews the plan and tasks together before any code, with a fresh-eyes subagent. **code:** after the build, a ticket review, then a code review |
| `/flow:build <name>` | Builds task by task: test first, ticket review, self code review. Never commits |

### Starting
```
/flow:start Add CSV export of the audit log for compliance users
/flow:start docs/specs/audit-export.md        ← an existing plan: validate it, or use it as is
/flow:start @tasks.md                         ← existing tasks: imported, then each gets a test
/flow:start                                   ← pick an existing flow to continue
/flow:start export-audit-log                  ← continue that flow
```

---

## How a flow runs

1. **Plan.** Claude reads the code, then asks one question at a time, with concrete options and its recommendation first.
   - Each answer goes in `decisions.md`, and `plan.md` fills in as you go.
   - If you answer "No preference", Claude takes the recommended option and marks it *(Claude's call)*.
   - An existing plan can be **validated**: Claude lists its gaps, then grills you only on those.
2. **Tasks.** The plan is split into tasks, usually 3-10.
   - Each task has its files, its steps, and **its own test**: a command that runs only that test, plus what it proves.
   - Tasks that share files run in sequence.
   - You choose the **build** mode (one task at a time, or parallel subagents) and the **run** mode (pause after each task, or all in one go).
3. **Review: plan and tasks together.** Claude and a fresh-eyes subagent check:
   - every plan step and "Done when" item is covered;
   - there's no extra scope;
   - the tests really prove each task;
   - the order and file overlaps are safe;
   - the risks are handled.

   You pick which fixes to apply. Anything that changes *what* gets built is asked separately.
4. **You approve** the plan and its tasks together, or stop here; everything is saved.
5. **Build**, for each task:
   1. **Test first.** Write the test and run it: it must **fail**.
   2. **Code.** Write the code and run the test again: it must **pass**.
   3. **Ticket review:** the spec is done, nothing is out of scope, and the test proves the spec.
   4. **Self code review.**
   5. **Pause** or carry on, depending on the run mode.

   At the end, every task's test runs together, then the project's full test command.
6. **Review: code.**
   - **Ticket review** of the whole change against the plan's goal, scope, decisions and "Done when".
   - **Staging:** only the flow's files are staged, **by name**.
   - **Code review:** Claude's own check, then `brainx:code-review`, or `code-review` if that isn't installed, or a careful read.
   - Every finding is fixed, or skipped with a reason you agree to.
7. **Hand-over.** You get:
   - the staged files;
   - a list of changed files that were left out (for example generated junk);
   - a suggested commit message (from the `commit-message` skill when it's installed).

   **You commit.** Flow never commits, creates branches or switches branches.

---

## Files

```
docs/flow/<name>/
  plan.md        what and why: goal, scope, decisions, approach, risks, done when (status in front matter)
  decisions.md   every grill-me question and answer
  tasks.md       tasks with files, how, test command, status and test runs, plus "Done when"
  review.md      plan + tasks review, then code review: every finding and how it ended
  progress.md    one line per step; the last lines say what's next
```

These are meant to be committed with the code, so reviewers can see why the work was done this way. The full format is in [`plugins/flow/FORMAT.md`](plugins/flow/FORMAT.md).

**Continuing:** `/flow:start` with no arguments lists your flows and where each stands, read from these files. Pick one and it carries on from the next step.

---

## Notes
- **Commit hooks that stage everything** (for example `git add -A` before reviewing) would also commit the files Flow left out. Gitignore generated files, or remove them before committing. Flow's hand-over lists them.
- **Unrelated changes** that were already in your working tree are never staged.
- **Projects with no test runner:** the first task sets one up, with a smoke test.

## Repo layout
```
.claude-plugin/marketplace.json
plugins/flow/
  .claude-plugin/plugin.json
  FORMAT.md                      file formats shared by all skills
  skills/{start,plan,tasks,review,build}/SKILL.md
```
After changing a skill, bump `version` in `plugin.json` and push. Users update with `claude plugin update flow@flow-workflow`.
