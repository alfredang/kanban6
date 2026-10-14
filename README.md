# kanban6

A single-file IT PMO Kanban board for demos and training, built with plain HTML, CSS and JavaScript.

**Live demo:** https://alfredang.github.io/kanban6/

## Features

- Four columns: Backlog, In Progress, Blocked and Done. Move cards by drag and drop or with the keyboard-friendly "Move ▸" control.
- An Add Task form with inline validation, plus filters, a live summary strip and overdue badges.
- Inline "Delete? Yes / No" confirmation on cards.

## Run it

Open `index.html` in a browser (double-clicking it works). There's no build step and no server.

Tasks live only in memory, on purpose. Refreshing the page resets the board to the demo data.

## Email notifications

New tasks send an email notification through [FormSubmit](https://formsubmit.co). To choose the recipient, set the address in `FORMSUBMIT_ENDPOINT` at the top of the script in `index.html`. FormSubmit sends a one-time activation email on the first submission, and nothing is delivered until the link in it is clicked. If the notification fails, the task is still added and a warning toast is shown.

## Tech

Vanilla HTML, CSS and JavaScript in a single file. No frameworks, libraries or other dependencies.

## Deployment

Every push to `main` deploys `index.html` to GitHub Pages through [`.github/workflows/pages.yml`](.github/workflows/pages.yml).

## Disclaimer

This is a fictitious demo. It isn't affiliated with, endorsed by or connected to any real bank.
