# OneShot files

Every piece of work lives in its own folder inside the project, `docs/oneshot/<name>/`. `<name>` is a short kebab-case slug of the title, e.g. `export-audit-log`. If that folder already belongs to other work, add `-2`, `-3` and so on.

The files are plain markdown, so they're readable in any editor, in a PR, and a month later. They're meant to be committed with the code; the user does the committing. OneShot skills edit these files directly. Keep their structure exactly as below, because every skill (and `/oneshot:start` when continuing) reads it.

```
docs/oneshot/<name>/
  plan.md        what and why: the plan
  decisions.md   grill-me questions and answers
  tasks.md       how: the tasks, each with its own test, plus "Done when"
  review.md      review results: plan + tasks, then code
  progress.md    running log: what happened, what's next
```

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
```

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

## T1: Add audit:export permission
- status: todo          # todo | doing | done | skipped
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
- 2026-11-13 14:10 · plan validated (8 decisions)
- 2026-11-13 14:30 · tasks written (6), reviewed, approved
- 2026-11-13 15:05 · T1 done: permission added, test passes. Next: T2
```
Append one line per step, newest last, each starting with the date and time. The last lines say where things stand and what's next; continuing a flow starts from them.

## Where a flow stands
Read the front matter of `plan.md` and `tasks.md`, then the end of `progress.md`:

| State | Next step |
|---|---|
| no `plan.md`, or plan `drafting` | plan |
| plan `validated`, no `tasks.md` | tasks |
| tasks `draft` | review (plan + tasks) |
| tasks `reviewed` | approval |
| plan `approved`, tasks `approved` or `building` | build (next task that isn't done) |
| all tasks done, no code review yet | review (code) |
| tasks `done` | finished: hand over for commit |
