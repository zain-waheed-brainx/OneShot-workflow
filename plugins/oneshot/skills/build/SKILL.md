---
name: build
description: Build an approved OneShot piece of work task by task in the current project and branch. Each task is test-first (the test must fail before the code and pass after), then checked against its spec and reviewed for code quality. Independent tasks can run in parallel subagents. Pauses after each task or runs straight through, as chosen. Never commits. Progress is saved in docs/oneshot/<name>/, so it can stop and continue later. Use for "/oneshot:build", "build the flow", "do the next task".
argument-hint: "<flow name> [all|step]"
---

# OneShot: build

Read `<this skill's base directory>/../../FORMAT.md` first. Work in the current project folder and branch: no worktrees, no branch changes, and **never `git commit`**.

## 1. Get ready
1. **Check it's approved.** `plan.md` and `tasks.md` must be `approved` (or `tasks.md` `building`, when continuing). If not, say what's missing. Approval happens in `/oneshot:start` after the review.
2. **Load the context:** `plan.md`, `decisions.md`, `tasks.md` and **Now** in `progress.md`. Set `status: building` in `tasks.md`. If the files are in the older format, convert them first (FORMAT.md, "Older files").
3. **Check the test commands.** Each one runs on this machine as written. Before running a task's test the first time, make sure it only runs the project's test runner on that task's tests. If it does anything else, such as chaining commands, making network calls or deleting files, stop and ask.
4. **Check git:** run `git status`. Note any changes that existed before this build. They aren't this flow's work, and they never get staged.
5. **Set up the task list** from `tasks.md`, following "Progress you can see" below. Tasks already done show as done or folded, so a continued build starts where it left off.

## Progress you can see
Show progress in the session's task list: the checklist tool your environment has, such as TodoWrite, or the app's own plan or checklist tool. Never print the checklist as text. Keep it matching `tasks.md` at all times:
- **One item per task**, titled `T#: <title>`, in build order. Never group tasks into batches.
- **Many tasks (more than 8):** fold the finished ones into one item, `T1–T6 done`. Show the current task(s) and the next 3 on their own, then one item for the rest, e.g. `T11–T16 and the final test run`. Re-fold as you go.
- **Exactly one item is in progress: the current task.** For a parallel batch, that one item names them all: `T9: verify domain (+ T12, T13 in parallel)`.
- **Update at the moment a task starts or finishes,** in the same message as your next tool call, never as a message of its own. If the list ever disagrees with `tasks.md`, fix the list first.
- **The last item is always** `Run all tests together`.
- **Progress line:** after each task (or parallel batch) is done, write one line, with a 10-character bar: filled cells = done ÷ total × 10, rounded to the nearest whole cell:
  `▓▓▓▓░░░░░░ 6/16 · T6 done (26 tests pass) · now T9`
- Steps inside a task (writing the test, the failing run, the code) are not list items. Updating that often costs tokens and adds little.

## 2. Loop
Repeat until every task is `done` or `skipped`.

1. **Next batch.** Take the first task that isn't done and whose `after:` tasks are all done. If `build: parallel`, also take the other ready tasks that are `parallel: yes` and share no files with each other.
2. **Mark it `doing` (🔄)** and update the views (FORMAT.md, "Keeping the views current"). In the same message as the first step of the work, make it the in-progress item in the task list.
3. **Test first.** For each task:
   1. **Write the test exactly as the task's `test` describes, and nothing else.** Don't write any of the task's code yet.
   2. **Run its command. It must fail, and so must every case in it.**
      - A case that passes before the code doesn't test the new behaviour, e.g. an "it throws" case that passes only because the function doesn't exist yet. Fix it so it fails for the missing behaviour, and run it again.
      - If it fails for the wrong reason (a syntax error, or the runner can't start), fix that first.

      Record it in `test runs:`, e.g. `before: failed 14:02`.
   3. **Write the code**, following the task's `How` and the plan's decisions.
   4. **Run it again. It must pass.**
      - Record `after: passed 14:09`.
      - If it fails, fix the code, not the test (unless the test itself was wrong, which you then say).
      - If you can't make it pass, set the task to `blocked` (❌), update the views (FORMAT.md, "Keeping the views current") with a ⚠️ or ❌ log line saying why, and stop to ask.
   - **Manual tests:** describe how to check it, and ask the user to confirm that it works.

   **Parallel batch:** launch one subagent per task in a single message, all in this folder. Give each one its task, the plan's decisions, and the steps above. Tell it to change **only its own files**, not to stage or commit anything, and to report its before-run, what it changed, and anything it couldn't do. When they're back, run each task's test yourself, one at a time.
4. **Ticket review** (yourself, no skill). Read the task's diff:
   - **Spec:** does it do everything the task's `How` says, following the plan's decisions?
   - **Scope:** does it do nothing outside the plan? A file outside the task's `files` is fine only if it clearly belongs to the task: add it to `files` and say so.
   - **Test:** does the test prove the spec?
5. **Self code review:** bugs, edge cases, error handling, and leftover debug code. Run lint if the project has it. Fix what you find and re-run the task's test.
6. **Mark it `done` (✅)** and update the views (FORMAT.md, "Keeping the views current"). The log line is `✅ T# done: what changed (test result)`, plus a ⚠️ line for any gotcha. Tick it in the task list, make the next task in progress, and write the progress line in chat.
7. **Pause or carry on.** Use `all` or `step` from the arguments if given, otherwise the `run:` mode.
   - **step:** ask **Continue with T#** (Recommended), **Run the rest in one go**, **Stop here**, or **Change something first**.
   - **all:** carry on. The progress line is the update after each task.
   - **Either mode:** stop and ask when a test can't be fixed, when a task needs a decision the plan doesn't cover (log the answer in `decisions.md`), or when a change would go outside the plan.

## 3. All tasks done
1. **Run everything together:** every task's test in order, then the project's `test-command`. Everything must pass. A test that passed alone but fails together usually means two tasks interfere: find out why, and fix it. Never report done with failing tests.
2. Update the views (FORMAT.md, "Keeping the views current") with the result, e.g. `✅ All tests together pass (10/10)`.
3. **Next step:** if `oneshot:start` invoked this, it carries on with the code review. Otherwise offer `/oneshot:review <name> code`. The code review stages the flow's files and checks them. Nothing gets committed.

## Rules
- **Follow the plan.** A decision it doesn't cover goes to the user as one question, and the answer goes in `decisions.md`.
- **Never skip a task silently.** To skip one, ask first, then set `status: skipped` with the reason.
- **Never stage unrelated changes, and never commit.**
