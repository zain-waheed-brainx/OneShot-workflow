---
name: review
description: Review a OneShot piece of work. "plan" mode reviews the plan and its tasks together before any code - coverage, extra scope, weak tests, order and file overlaps, risks - with a fresh-eyes subagent. "code" mode runs after the build - a ticket review of the whole change against the plan and its done-when items, then a code review (brainx:code-review, else code-review, else a careful read). Fixes go in only with the user's approval. Results go in docs/oneshot/<name>/review.md. Use for "/oneshot:review", "review the plan and tasks", "review the code".
argument-hint: "<flow name> plan|code"
---

# OneShot: review

Read `<this skill's base directory>/../../FORMAT.md` first. Results go in `docs/oneshot/<name>/review.md`, one dated section per review, and every finding ends as `fixed`, or as `accepted`/`skipped` with the user's reason.

If no mode is given: use `plan` when `tasks.md` is `draft`, or `code` when every task is done.

## plan: the plan and its tasks together (before any code)
1. **Fresh eyes.** For small work (3 tasks or fewer, all in one area) you may skip the subagent to save tokens; say so in `review.md`. Otherwise launch one read-only subagent with only `plan.md`, `tasks.md` and the project path. It hasn't seen the grilling. Ask it to check the points below and return its findings, each with where it is and a suggested fix. Do your own check at the same time.
2. **What to check:**
   - **Coverage:** every Approach step and every "Done when" item is covered by at least one task.
   - **Nothing extra:** no task does something the plan's Scope leaves out.
   - **Decisions:** the tasks follow every decision in the plan and in `decisions.md`.
   - **Tests:** each task has a test, and its command runs only that test. It proves the task's behaviour, not something easier, and could actually fail before the code. Manual tests have a real reason.
   - **Order:** dependencies make sense, there are no cycles, tasks that share files run in sequence, and `parallel: yes` only where it's safe.
   - **Clarity:** each `How` is specific enough to build without guessing, and names the files.
   - **Risks:** the plan's risks are handled by a task or marked as accepted.
3. **Merge** your findings with the subagent's, dropping duplicates, and write them to `review.md` as `[open]`.
4. **Fix with approval.** Show the findings as a numbered list, each with its fix in one line. Ask with AskUserQuestion (multiSelect) which fixes to apply. Options: **All of them** (Recommended), individual items, or **None**.
   - A finding that changes **what** gets built gets its own grill-style question, and the answer is logged in `decisions.md`.
   - Apply the chosen fixes to `plan.md` and `tasks.md`.
   - Mark each finding `[fixed]`, or `[accepted]` with the user's reason.
5. Set `status: reviewed` in `tasks.md` and add a `progress.md` line. The flow then goes to approval: `oneshot:start` asks, or, standalone, offer `/oneshot:start <name>`.

## code: after the build
1. **Ticket review first**, by you, without a review skill. Read the whole change (`git diff` plus new files, limited to the task files) and check it against the plan:
   - Goal, Scope (in and out) and Decisions;
   - anything missing, half done or built beyond scope, including gaps between tasks;
   - each "Done when" item: tick it in `tasks.md` (and `plan.md`) if you actually verified it by running a test or command, and list the ones you couldn't verify.

   Fix the gaps the user agrees to, then re-run the affected task tests.
2. **Stage only this flow's files by name:** every task's `files`, plus files the reviews accepted as belonging to the work, plus `docs/oneshot/<name>/`. Use `git add -- <file> …`, never `git add -A` or `git add .`.
3. **Your own code review** of the staged diff (`git diff --cached`): bugs, missed edge cases, error handling, leftover debug code, and lint if the project has it.
4. **Then the review skill**, on the staged changes only. Use the first of these that's available:
   1. `brainx:code-review`;
   2. `code-review`;
   3. a careful read of the diff.

   If a skill asks whether to review staged or all changes, choose staged only.
5. **Record and fix.** Write every finding to `review.md` under `## Code review · <date>`, naming the reviewer. Fix every finding, or skip or accept it with a reason the user agrees to. Re-stage after fixes, and re-run the affected task tests and the full test command.
6. Add a `progress.md` line. If every finding is closed, set `status: done` in `tasks.md`.
7. **Re-stage the flow's files** (`git add -- docs/oneshot/<name>/`), because `review.md`, `progress.md` and `tasks.md` changed after step 2. Then check `git diff --cached --stat`, so the hand-over shows what's really staged.

**Never commit.** The hand-over (in `oneshot:start`) gives the user the staged files and a commit message.
