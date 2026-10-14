# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file Kanban board for a fictitious bank's IT PMO, built as an internal demo and training tool. Everything is in `index.html`: the markup, one `<style>` block and one `<script>` block.

## Hard constraints (from the original brief — do not break)

- Vanilla HTML/CSS/JS only. No frameworks, libraries, build step, npm, CDN scripts, web fonts or image files. Use the system font stack and Unicode or inline SVG for icons.
- It must run by double-clicking `index.html` (`file://`). No server.
- **No persistence.** Do not use localStorage, sessionStorage, IndexedDB or cookies. A refresh resets the board to the seed data on purpose, and the header note says so.
- The only network call is the FormSubmit AJAX POST in `notifyNewTask()`. No other backend.
- No `alert()`, `confirm()` or `!important`. Card deletion uses the inline "Delete? Yes / No" toggle.
- Branding: show only the neutral "IT PMO" text wordmark in a corporate blue palette. Do not add a real bank's logo, trademarks or anything that imitates its systems. The `UOB-ITPM-####` ID prefix and the `[UOB IT PMO]` email subject were specified in the brief.

## Commands

There is no build, lint or test tooling.
- Run: open `index.html` in a browser (on Windows: `Start-Process index.html`).
- Quick JS syntax check (needs Node): extract the script with `sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/k.js && node --check /tmp/k.js`
- Constraint check: `grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|alert\(|confirm\(|!important|https?://' index.html` should only match the `FORMSUBMIT_ENDPOINT` line.

## Architecture (inside the `<script>` block)

- **Config at the top:** `FORMSUBMIT_ENDPOINT` is the one place to change the notification email address. The HTML comment above `<script>` explains FormSubmit's one-time activation email.
- **Single source of truth:** `state = { tasks, filters, pendingDeleteId }`. `pendingDeleteId` controls which card shows the inline delete confirmation.
- **Render from state:** every action (`addTask`, `moveTask`, `deleteTask`, `setDeleteConfirm`) changes `state` and then calls `renderBoard()`. `renderBoard()` rebuilds every column from strings returned by `renderCard()`, then calls `renderSummary()`. Do not change card DOM anywhere else. Toasts are the only DOM built directly.
- **Escaping:** every user-supplied string must go through `escapeHtml()` before it is put into an HTML string.
- **Event delegation:** columns and cards are recreated on every render, so drag-and-drop (`initDragAndDrop`) and card controls (`initCardControls`: the Move ▸ select and the delete/confirm buttons) are bound once to `#board`, not to individual cards. After a re-render, focus is put back with `focusInCard(id, selector)`.
- **Counts:** the header summary counts all tasks. Column badges count only the tasks that pass `applyFilters()`.
- **Dates** are local `YYYY-MM-DD` strings compared as strings (`todayISO()`, `isOverdue()`). Seed due dates are set relative to today with `addDays()` so some cards are always overdue.
- **Add Task flow (`handleSubmit`):** validate (inline errors, `aria-invalid`), then `addTask()` straight away so the card appears at once, reset the form and show a success toast. After that, `notifyNewTask()` runs with the button showing "Sending…". If it fails, show the warning toast and keep the card. A FormSubmit failure must never break the board. `notifyNewTask` also treats `success: "false"` in a 200 response as a failure, which happens while the address isn't activated yet.
- **Accessibility:** keep `<label for>` on every field, `aria-label` on icon-only buttons, the `aria-live` toast region, the text on priority pills (colour alone must not carry meaning), and the modal's focus trap and Esc-to-close.
- **Styling:** colours and spacing use CSS custom properties on `:root`. The columns sit in a 4-column grid, drop to 2 columns between 768px and 1100px, and stack below 768px.
