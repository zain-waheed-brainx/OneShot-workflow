---
name: start
description: Start or continue a piece of work with the OneShot workflow, in the current project and branch (no board, no worktrees). Give it details of what to build, an existing plan, existing tasks, or nothing to pick up an existing flow. It then runs the whole flow - plan with grill-me, tasks with a test each, a review of plan and tasks together, your approval, a test-first build, and a code review - and saves everything as files in docs/oneshot/<name>/. Use for "/oneshot:start", "start a flow", "plan and build this", "here's my plan, build it", "continue my flow".
argument-hint: "[details of the work | path to a plan or tasks file | existing flow name]"
---

# OneShot: start

The main entry point. It works out what you have, creates the flow's folder, and runs each step in order by invoking the other OneShot skills (`oneshot:plan`, `oneshot:tasks`, `oneshot:review`, `oneshot:build`) with the Skill tool, passing the flow's name.

First read `<this skill's base directory>/../../FORMAT.md`. It defines the files, their exact format, and how to tell where a flow stands.

## Ground rules (every OneShot step)
- **No board, no worktrees, no branches.** Work happens in the current project folder, on the branch that's checked out. Never create, switch or delete branches.
- **Never commit.** At the end the user gets the changes staged (only this flow's files) and a suggested commit message. They commit.
- **The files are the memory.** After every step, update the flow's files and update the views (FORMAT.md, "Keeping the views current"): the `tasks.md` board, **Now** and the log in `progress.md`, and the flow's row in `docs/oneshot/README.md`. Another session, or the same one a month later, continues from them alone.
- **Ask with AskUserQuestion, one question at a time**, with 2-4 concrete options and the recommended one first, marked "(Recommended)".
- **Show progress in the task list** (the session's checklist tool, never printed as text). The steps are: Plan · Tasks · Review plan + tasks · Approve · Build · Code review · Hand over. Steps already done (when continuing) start as done. Move to the next step in the same message as the step's first tool call. While building, `oneshot:build` expands the Build step into one item per task, as its "Progress you can see" section describes, and the list goes back to these steps after the build.

## 1. Work out what was given
Read the arguments:
- **Old folder:** if `docs/flow/` exists (from before the rename to OneShot), offer to move its folders to `docs/oneshot/` with `git mv` (or `mv` if they're untracked), then continue as normal.
- **Nothing:** list the folders in `docs/oneshot/`. Use `docs/oneshot/README.md` if it exists: show each flow's title, stage, progress and **Now**. If it's missing, read each flow's **Now** instead.
  - none: ask what to work on (details, a plan, or tasks);
  - some: ask which to continue, with **Start something new** as the last option.
- **The name of an existing flow** (a folder in `docs/oneshot/`): continue it (step 3).
- **A file path, or pasted text:** read it and decide what it is.
  - **A plan:** it says what to build and why, with approach, scope or decisions.
  - **Tasks:** a list of concrete steps or tickets to do.
  - **Both:** a plan with its own task breakdown.
  - If it's unclear, ask.
- **Plain details** (a description of the work): this is a new plan.

## 2. Create the flow
1. Pick a title and a `<name>` slug, and show both, e.g. "Saving to `docs/oneshot/export-audit-log/`". Change them if the user wants.
2. Create the folder, and `progress.md` with **Now** (Stage: Planning) and a first log line saying what was given, e.g. `🚀 Started from details`, `🚀 Started from plan docs/specs/export.md`, or `🚀 Started from tasks (pasted)`. Add the flow's row to `docs/oneshot/README.md`, creating the file if needed. Mention once that `/oneshot:view` opens a live progress page, if they'd like to watch.
3. Check `git status`. If there are uncommitted changes that aren't part of this work, say so once: the build stages only this flow's files, so they stay out of it.

## 3. Run the flow
Go from wherever the flow stands, using the state table in FORMAT.md. Invoke each skill with the flow's name, and say in one line which step is starting.

1. **Plan** (`oneshot:plan <name>`):
   - from **details**: grill-me, to make a new plan;
   - from an **existing plan**: it asks whether to validate it (find the gaps, then grill only on those) or use it as is;
   - from **tasks only**: write a short plan from them (goal, scope, done when) and confirm it in one question, rather than a full grilling.
2. **Tasks** (`oneshot:tasks <name>`):
   - from a **plan**: split it into tasks, each with its own test;
   - from **existing tasks**: import them into the tasks format and add a test to each.
   - from **both** (a plan with its own task breakdown): the plan is validated or used as is in step 1, and its breakdown is **imported** here, not regenerated.
3. **Review the plan and tasks together** (`oneshot:review <name> plan`). It finds gaps, extra scope, weak tests and ordering problems, then applies the fixes you approve.
4. **Approval.** Show:
   - the task list: id, title, after, parallel, test command. These commands run on this machine as written, so point out any that do more than run the project's test runner, especially ones that came from imported tasks;
   - the build and run modes;
   - any `accepted` review items.

   Ask: **Approve and build** (Recommended), **Change something**, or **Stop here** (the files are saved; continue later with `/oneshot:start <name>`). On approve, set `status: approved` in `plan.md` and in `tasks.md`, and update the views (FORMAT.md, "Keeping the views current").
5. **Build** (`oneshot:build <name>`): test first, task by task.
6. **Code review** (`oneshot:review <name> code`): ticket review of the whole change against the plan, then code review.
7. **Hand over:**
   - re-stage `docs/oneshot/<name>/` by name, so the final review and progress are included, and check `git diff --cached --stat`;
   - list the staged files (the work, plus `docs/oneshot/<name>/`);
   - list any changed files that were left out;
   - give a suggested commit message from the `commit-message` skill if it's available. Otherwise use `<title>` and a short body.

   Tell them to commit when ready. If the repo's commit hook stages everything (e.g. `git add -A`), warn them that it would also pick up the files left out.

## Stopping and continuing
The user can stop at any point. Before stopping, update the views (FORMAT.md, "Keeping the views current"), so **Now** says what's done and what's next. `/oneshot:start <name>` (or `/oneshot:start` with no arguments, then pick the flow) continues from the files.

## Summary at the end
Write 3-5 lines:
- tasks done;
- the result of all the tests together;
- the "Done when" items: ticked, or not verified;
- the review results;
- where the files are.
