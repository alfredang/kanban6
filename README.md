# kanban6

A single-file IT PMO Kanban board for demos and training, built with plain HTML, CSS and JavaScript. To run it, open `index.html` in a browser. There's no build step and no server.

- Four columns: Backlog, In Progress, Blocked and Done. Move cards by drag and drop or with the keyboard-friendly "Move ▸" control.
- An Add Task form with inline validation, plus filters, a live summary strip and overdue badges.
- Tasks live only in memory. Refreshing the page resets the board to the demo data.
- New tasks send an email notification through [FormSubmit](https://formsubmit.co). To choose the recipient, set the address in `FORMSUBMIT_ENDPOINT` at the top of the script. FormSubmit sends a one-time activation email on the first submission, and nothing is delivered until the link in it is clicked.
