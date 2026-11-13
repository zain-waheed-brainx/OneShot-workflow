---
name: view
description: Open a live, read-only progress page for this project's OneShot flows. It shows a card per flow, a task board, Now, recent activity, review status and the done-when checklist, and updates by itself whenever docs/oneshot changes. A small local script draws it, so it costs almost no tokens. Use for "/oneshot:view", "show progress", "open the progress page", "visualize my flow".
argument-hint: ""
---

# OneShot: view

Start the viewer and give the user the link. Don't read the flow files or describe the progress yourself: the page does that, and doing it here would only cost tokens.

1. Run `oneshot-view` from the project root with Bash and `run_in_background: true`. The plugin's `bin/` is on PATH; if it isn't found, use `node "<this skill's base directory>/../../bin/oneshot-view"`.
   - It opens the page in the default browser by itself.
   - If a viewer for this project is already running, it just opens that one.
2. Read the first lines of its output for the link (`http://localhost:<port>`). If the session has a built-in browser (for example the desktop app's browser pane), you may also open the link there.
3. Reply in one or two lines: the link, and that the page updates by itself while it's open, stops after 2 hours with no file changes and no page loads, and is read-only.

If `docs/oneshot/` doesn't exist yet, say there are no flows yet, and offer `/oneshot:start`.
