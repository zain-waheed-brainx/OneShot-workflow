---
name: plan
description: Plan a piece of work for the OneShot workflow by grilling the user (grill-me) on what's new: it remembers the user's preferences, applies the settled ones and shows them on one confirm screen, and always asks about the work itself, or validate an existing plan and grill only on its gaps. Writes docs/oneshot/<name>/plan.md and decisions.md. Usually run by /oneshot:start; can be used alone. Use for "/oneshot:plan", "grill me on this", "validate this plan".
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
Interview the user until you both understand the work the same way, asking only what you need. Start with the decisions others depend on.

### Before the first question
1. **Read the project's rules:** `CLAUDE.md`, `CLAUDE.local.md` and `.claude/CLAUDE.md`, plus any standards files they point to. Rules they settle (where validation goes, how queries are written, comments, locking) are decided: apply them, never ask about them.
2. **Read the code and find what already exists** before planning anything new: helpers, traits, base classes, models and their scopes, services, shared components, and constants for shared lists. The plan extends these instead of adding a parallel version. Never ask what the repo already answers, like the test framework or the folder layout.
3. **Read the preferences:** `docs/oneshot/preferences.md` and `~/.claude/oneshot/preferences.md` (FORMAT.md, "Preferences"). Either may be missing.
4. **Size the work and set a budget** of questions: small (a few files, one area) about 3, medium about 6, large as needed but still grouped. Questions about the work itself are never dropped to meet the budget; small points become "Assumed" lines instead (below).

### Sort every question (FORMAT.md, "Kinds of question")
- **Work facts** (what it does, scope, edge cases, data): always asked. Never assumed, never stored.
- **Settled preference** (`✅ auto`): don't ask. Apply it and keep it for the confirm screen.
- **New or unsettled preference** (no row, `💬 ask`, `🔀 always ask`): ask. For `💬 ask`, the stored answer is the first option, marked "(Recommended)" and "(your usual)".
- **Small points** that don't change the plan (a file name, a log message, a default limit): don't ask. Add them to the plan's **Assumed** section as one line each, which the user can edit.

### Asking
- **Batch related questions:** up to 4 per AskUserQuestion call, each with 2-4 concrete options and the recommended one first, marked "(Recommended)". Ask a question on its own when later ones depend on its answer.
- **Log every answer** in `decisions.md`, as `## Q<n>. <question>`, then `**Answer:** <answer>. Why: <one line>`.
- **Keep the plan current.** After each batch, and at the end, rewrite `plan.md` so it fills in as you go. Keep `status: drafting` until the end, and update `updated:`.
- **"No preference":** take the recommended option, mark it *(Claude's call)* in both files, and keep going. It is never approval, and it is never stored as a preference.

### The confirm screen
When nothing important about **what** to build is left open, show one message:
- **Used your usual choices:** a numbered list of every `✅ auto` preference applied, e.g. `1. Validation in Form Request classes`.
- **Assumed:** the plan's Assumed lines.

Then one AskUserQuestion: **Accept all** (Recommended) or **Change some**. Skip the screen if both lists are empty.
- **Change some:** ask which ones (multiSelect, up to 4 per screen), then ask each picked one as a normal question.
- Log each applied preference in `decisions.md` as `**Answer:** <answer>. From your preference (<last flow>).`, and each changed one as a normal answer.

### After the confirm screen
1. **Update the preferences** by the confidence rules (FORMAT.md, "Preferences"): every preference question asked or applied in this plan. Code style, libraries and patterns go in the project file; build and run modes in `~/.claude/oneshot/preferences.md`. Never store work facts or *(Claude's call)* answers.
2. Set `status: validated`, and update the views (FORMAT.md, "Keeping the views current"), e.g. log `🧭 Plan validated: 8 decisions (3 from your preferences)`.

## The plan
Keep it short. A builder should be able to work from it a month from now with nothing else:
- Goal / why
- Scope (in / out)
- Decisions, each with a one-line why
- Approach: the steps, with the files and areas they touch
- Code approach: reuse, queries, writes and locking, where validation and auth live (FORMAT.md). Filled from the code and the project's rules, not from questions
- Assumed: small points decided without asking, one line each
- Risks & open questions
- Done when: checkable items

## After
Show a 5-line summary. If `oneshot:start` invoked this, it carries on. Otherwise offer `/oneshot:tasks <name>`. Don't ask for approval here: the plan is approved together with its tasks, after the review.
