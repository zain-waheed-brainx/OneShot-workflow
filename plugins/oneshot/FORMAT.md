# OneShot files

Every piece of work lives in its own folder inside the project, `docs/oneshot/<name>/`. `<name>` is a short kebab-case slug of the title, e.g. `export-audit-log`. If that folder already belongs to other work, add `-2`, `-3` and so on.

The files are plain markdown, so they're readable in any editor, in a PR, and a month later. They're meant to be committed with the code; the user does the committing. OneShot skills edit these files directly. Keep their structure exactly as below, because every skill (and `/oneshot:start` when continuing) reads it.

```
docs/oneshot/
  README.md      index: every flow with its stage and progress, one row each
  <name>/
    plan.md        what and why: the plan
    decisions.md   grill-me questions and answers
    tasks.md       how: a status board on top, then the tasks, each with its own test, plus "Done when"
    review.md      review results: plan + tasks, then code
    progress.md    "Now" on top (where things stand), then the log, newest first
  preferences.md   this project's remembered answers (code style, libraries, patterns)

~/.claude/oneshot/preferences.md   workflow answers shared by all your projects (build mode, run mode)
```

**Icons** (used in the board, in task headings, and in the index): ✅ done · 🔄 in progress · 🔍 under review · ⬜ to do · ⏭️ skipped · ❌ blocked.

## plan.md
```markdown
---
title: Export audit log as CSV
status: drafting        # drafting | validated | approved
source: grill           # grill | imported | details
created: 2026-11-13
updated: 2026-11-13
---
# Export audit log as CSV
## Goal / why
## Scope
In: …
Out: …
## Decisions
- CSV plus a SHA-256 checksum file. Why: compliance needs integrity proof. *(Claude's call)* when the user had no preference
## Approach
1. … (steps, with the files and areas they touch)
## Assumed
- Export filename is `audit-log-<date>.csv`. Edit this line to change it.
## Risks & open questions
## Done when
- [ ] A user with audit:export can download a CSV and its checksum
```
`approved` is set only when the user approves the plan and its tasks together, after the plan + tasks review.

## decisions.md
```markdown
# Decisions: Export audit log as CSV
## Q1. Who can export?
**Answer:** Compliance role only. Why: least privilege.
## Q2. Where does request validation go?
**Answer:** Form Request classes. From your preference (user-import).
```
Each answer says where it came from: the user's answer (no note), *(Claude's call)* for "no preference", or `From your preference (<flow>)` for a preference applied and confirmed on the confirm screen, naming the flow where it was last answered (`From your workflow preference` for build and run modes).

## Preferences
Planning remembers the user's answers to **preference** questions, so it only asks about what's new. There are two files, same format:
- **`docs/oneshot/preferences.md`:** project preferences, e.g. code style, libraries, patterns, test style. It sits next to the flows and can be committed, so teammates share it.
- **`~/.claude/oneshot/preferences.md`:** workflow preferences shared by all the user's projects: build mode, run mode. Create the folder and file the first time one is saved. Its **Last flow** is always `-`: flow names can identify a client or project, so they never go in this shared file.

```markdown
# OneShot preferences

| Topic | Answer | State | Streak | Changes | Used | Last flow |
|---|---|---|---|---|---|---|
| Where request validation goes | Form Request classes | ✅ auto | 3 | 0 | 4 | user-import |
| Test style | Feature tests over unit tests | 💬 ask | 1 | 0 | 1 | export-audit-log |
| Error response shape | depends on the work | 🔀 always ask | 0 | 2 | 3 | export-audit-log |
```
- **Topic:** a short, stable name for the question, so the same question matches next time. Reuse an existing topic's wording rather than adding a near-duplicate.
- **State:**
  - `💬 ask`: asked every time, with **Answer** as the recommended option.
  - `✅ auto`: applied without asking, then shown on the confirm screen.
  - `🔀 always ask`: answers keep changing, so it depends on the work. Asked every time, with no option marked from the preference.
- **Streak:** how many times in a row the same answer was given or accepted. **Changes:** how many times the answer changed. **Used:** how many flows used it.

**Kinds of question.** Sort every question before asking it:
| Kind | Examples | What happens |
|---|---|---|
| Work facts | what it does, scope, edge cases, data, who can use it | Always asked. Never stored, never assumed. |
| Settled preference (`✅ auto`) | "validation in Form Request classes", "feature tests over unit tests" | Not asked. Applied, then listed on the confirm screen. |
| New or unsettled preference (no row, `💬 ask`, `🔀 always ask`) | anything not answered before, or that changes | Asked. For `💬 ask` the stored answer is the recommended option. |

A question the code already answers (the test framework, the folder layout) is neither: don't ask it, and don't store it.

**Confidence rules** (update the row after the confirm screen):
- **First answer:** add the row: `💬 ask`, Streak 1, Changes 0, Used 1.
- **Same answer again:** Streak +1, Used +1. At Streak 2 it becomes `✅ auto`.
- **Accepted on the confirm screen** (`✅ auto`): Streak +1, Used +1.
- **Different answer** (asked, or changed on the confirm screen): Answer is replaced, Streak 1, Changes +1, Used +1, back to `💬 ask`. When Changes reaches 2, it becomes `🔀 always ask` with Answer `depends on the work`, and stays that way until the user edits it.
- **`🔀 always ask` answered again:** only Used +1 and Last flow change. Answer stays `depends on the work`, and Streak and Changes stay as they are.
- **"No preference"** (*Claude's call*) is not an answer: don't add or update a row for it.
- Update **Last flow** to this flow whenever the row changes (project file only).

The user can edit the file by hand: change an answer, set a State, or delete a row to forget it. Skills read it fresh every time and respect those edits.

## tasks.md
```markdown
---
status: draft           # draft | reviewed | approved | building | done
build: sequential       # sequential | parallel
run: step               # step (pause after each task) | all (run straight through)
test-command: npm test  # the project's full test command, if it has one
updated: 2026-11-13
---
# Tasks: Export audit log as CSV

**Progress:** ▓▓▓▓▓░░░░░ 3/6 · **Now:** T4 Checksum file · **Updated:** 2026-11-13 15:40

| | Task | Test |
|---|---|---|
| ✅ | T1 Add audit:export permission | passes |
| ✅ | T2 CSV writer | passes |
| ✅ | T3 Export endpoint | passes |
| 🔍 | T4 Checksum file | passes, under review |
| ⬜ | T5 Rate limit | not run |
| ⏭️ | T6 Email the link | skipped: out of scope |

## ✅ T1: Add audit:export permission
- status: done          # todo | doing | review | done | skipped | blocked
- after: -              # ids this task depends on, e.g. T1, T2
- parallel: no          # yes only if it has no dependencies and shares no files with other tasks
- files: src/auth/permissions.ts, tests/auth/audit-export.test.ts
- test: `npx jest tests/auth/audit-export.test.ts` proves a role without audit:export gets 403 and compliance gets 200
- test runs: -          # filled in by build, e.g. before: failed 14:02 · after: passed 14:09
### How
1. …
Notes: follows the pattern in src/auth/roles.ts

## Done when
- [ ] A user with audit:export can download a CSV and its checksum
```
- Every task has its own `test`: a command that runs only that task's test, and one line on what it proves. Use `test: manual: <how to check> (why it can't be automated)` only when an automated test is impossible.
- `files` lists every file the task changes or adds, including its test file.
- **The board** sits right under the title: the progress line, then one row per task in build order. Each row has the icon, the id and title, and the test state (`not run` · `failing (…)` · `passes` · `passes, under review` · `manual` · `skipped: <why>`). The progress bar has 10 cells: filled = done ÷ total × 10, rounded to the nearest cell.
- **Each task heading** starts with the same icon as its `status`: `## 🔄 T3: Export endpoint`.

## review.md
```markdown
# Review: Export audit log as CSV

## Plan + tasks review · 2026-11-13
- [fixed] T4 had no test for the empty date range. Added to T4's test.
- [accepted] No rate limit on exports. User: out of scope for now.
- [open] …

## Code review · 2026-11-14
Reviewer: brainx:code-review
- [fixed] Missing null check in exportRows (src/audit/export.ts:42)
```
Every finding ends as `fixed`, or `accepted`/`skipped` with the user's reason. `open` is allowed only while the review is in progress.

## progress.md
```markdown
# Progress: Export audit log as CSV

## Now
- **Stage:** Build, 2/6
- **Working on:** T3 Export endpoint
- **Next:** T4, then code review
- **Blocked:** nothing
- **Updated:** 2026-11-13 15:40

## Log
### 2026-11-13
- 15:40 ✅ T2 done: CSV writer streams rows, 7 tests pass
- 15:05 ✅ T1 done: permission added, test passes
- 14:30 🔍 Plan + tasks reviewed: 2 findings, fixed. Approved
- 14:10 🧭 Plan validated: 8 decisions
- 13:50 🚀 Started from details
```
- **Now** is rewritten at every step, so it always says where things stand. Continuing a flow starts from it.
- **Log:** one line per event, **newest first**, under a `### <date>` heading for each day, each starting with the time and an icon: ✅ done · 🔄 started · 🔍 review · 🧭 decision or plan · ⚠️ problem or warning · ❌ blocked · 🚀 started the flow · 📦 handed over.

## docs/oneshot/README.md: the index
```markdown
# OneShot flows

| Flow | Stage | Progress | Now | Updated |
|---|---|---|---|---|
| [email-verification](email-verification/progress.md) | 🔄 Build | ▓▓▓▓░░░░░░ 6/16 | T9 verify domain | 2026-11-13 15:40 |
| [slugify-utility](slugify-utility/progress.md) | ✅ Ready to commit | ▓▓▓▓▓▓▓▓▓▓ 2/2 | - | 2026-11-13 18:20 |
```
One row per flow, newest update first. Stages: 🧭 Planning · 🧭 Tasks · 🔍 Review · ⏸️ Waiting for approval · 🔄 Build · 🔍 Code review · ✅ Ready to commit. Create the file if it's missing; update only that flow's row.

## Keeping the views current
Whenever a task starts or finishes, or the flow moves to another stage, update all of these in one go, before moving on:
1. **`tasks.md`:** the task's `status`, the icon in its heading, its board row, and the board's progress line (with **Updated**). Skip this until `tasks.md` exists.
2. **`progress.md`:** rewrite **Now**, and add a log line at the top of today's section (create the section if it's missing).
3. **`docs/oneshot/README.md`:** that flow's row.

These are small edits, a few lines each. Never rewrite whole files to do them.

**Older files.** A flow written before this format has no board, no Now and no icons, and its log is oldest first. Before its next update, convert it once: add the board and Now, add icons to the headings, and reverse the log into date sections.

## Where a flow stands
Read the front matter of `plan.md` and `tasks.md`, then **Now** in `progress.md` (in older files, the end of the log):

| State | Next step |
|---|---|
| no `plan.md`, or plan `drafting` | plan |
| plan `validated`, no `tasks.md` | tasks |
| tasks `draft` | review (plan + tasks) |
| tasks `reviewed` | approval |
| plan `approved`, tasks `approved` or `building` | build (next task that isn't done) |
| all tasks done, no code review yet | review (code) |
| tasks `done` | finished: hand over for commit |
