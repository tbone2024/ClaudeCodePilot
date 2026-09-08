# AGENTS.md

## Project context

This workspace contains a single-file, browser-only IT project management demo for an internal PMO board. The app is implemented in [index.html](index.html) with vanilla HTML, CSS, and JavaScript only. It is intended to run by double-clicking the file in a browser; no build step, framework, or local server is required.

## Working conventions

- Keep the app as a single-file static page unless a user explicitly asks for a different structure.
- Preserve the vanilla HTML/CSS/JS approach: no React, Vue, jQuery, Tailwind, npm packages, or bundlers.
- Keep the board state in the in-memory `state` object and re-render from state. Do not add persistence mechanisms such as `localStorage`, `sessionStorage`, IndexedDB, cookies, or any storage API.
- Reuse the existing seeded demo data model and keep the board reset-on-refresh behavior intentional.
- Maintain the UOB-style branding constraint: neutral wordmark only, corporate blue palette, and no real UOB logos or trademarks.
- Use system fonts and inline SVG / Unicode icons only; do not add external scripts, CDNs, fonts, or images.
- Preserve the requirement that the page works without a server and without navigation away from the page.

## Critical implementation notes

- Form submissions must use the FormSubmit AJAX JSON endpoint pattern; do not replace it with a plain HTML POST form.
- Keep the FormSubmit configuration in one clearly marked constant near the top of the script so the email address can be updated in one place.
- Ensure failed FormSubmit requests never break the board; the card should remain in the UI and show a warning toast instead.
- Escape all user-supplied strings via `escapeHtml()` before injecting them into HTML.
- Use the drag-and-drop API for card movement, and keep a keyboard-accessible fallback via the “Move ▸” selector.
- Keep the delete flow as an inline confirm pattern instead of `confirm()`.
- Validate user input client-side and show inline field errors without `alert()`.
- Keep the summary strip and filter bar live and derived from the current in-memory state.

## Validation steps for AI agents

- Check the page in a real browser or browser automation by opening [index.html](index.html) directly.
- Prefer small, targeted edits over broad refactors.
- If changing task logic, verify the following behaviors still hold:
  - board counts update correctly
  - filters work client-side over the in-memory array
  - drag/drop and keyboard move controls both update task status
  - overdue badges and priority colors remain accurate
  - form validation and toast messaging still behave as expected

## Preferred file to modify

- Primary implementation: [index.html](index.html)

## Notes for future work

This is a demo/training app, not a production system. Keep code simple, readable, and static. If a feature requires persistence or backend processing, it should be treated as a deliberate architectural change rather than a default assumption.
