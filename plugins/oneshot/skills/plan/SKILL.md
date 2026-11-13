---
name: plan
description: Plan a piece of work for the OneShot workflow by grilling the user one question at a time (grill-me), or validate an existing plan and grill only on its gaps. Writes docs/oneshot/<name>/plan.md and decisions.md. Usually run by /oneshot:start; can be used alone. Use for "/oneshot:plan", "grill me on this", "validate this plan".
argument-hint: "<flow name> [details | path to an existing plan]"
---

# OneShot: plan

Read `<this skill's base directory>/../../FORMAT.md` first, for the file formats.

Work in `docs/oneshot/<name>/`. If no name is given, run `oneshot:start` instead, which creates the flow.

## Choose how to plan
- **Details only:** grill (below), starting from the details.
- **An existing plan** (file or pasted): ask **Validate it** (Recommended: find the gaps, grill only on those) or **Use it as is**.
  - **Use it as is:** write it into `plan.md` in the plan format, keeping its content, with `status: validated` and `source: imported`.
  - **Validate it:** write it into `plan.md` with `status: drafting` and `source: imported`. List its gaps, risks and contradictions in one short message, then grill only on those.
- **Tasks only** (from `oneshot:start`): write a short plan from them (Goal, Scope, Approach as the task list, Done when). Ask one question: **Looks right** or **Change something**.

## Grilling
Interview the user until you both understand the work the same way. Walk each branch of the design one decision at a time, starting with the decisions others depend on.
- **Read first, then ask.** Before asking, read the relevant code and docs (the README, CLAUDE.md, and the files the work touches). Never ask what the code already answers.
- **One question at a time,** with AskUserQuestion: 2-4 concrete options specific to this work, the recommended one first, marked "(Recommended)".
- **Log every answer** in `decisions.md`, as `## Q<n>. <question>`, then `**Answer:** <answer>. Why: <one line>`.
- **Keep the plan current.** Every 2-3 answers, and at the end, rewrite `plan.md` so it fills in as you go. Keep `status: drafting` until the end, and update `updated:`.
- **"No preference":** take the recommended option, mark it *(Claude's call)* in both files, and keep going. It is never approval.
- **When to stop:** when nothing important about **what** to build is left open. Set `status: validated`, and update the views (FORMAT.md, "Keeping the views current"), e.g. log `🧭 Plan validated: 8 decisions`.

## The plan
Keep it short. A builder should be able to work from it a month from now with nothing else:
- Goal / why
- Scope (in / out)
- Decisions, each with a one-line why
- Approach: the steps, with the files and areas they touch
- Risks & open questions
- Done when: checkable items

## After
Show a 5-line summary. If `oneshot:start` invoked this, it carries on. Otherwise offer `/oneshot:tasks <name>`. Don't ask for approval here: the plan is approved together with its tasks, after the review.
