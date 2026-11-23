# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately tiny to-do app: plain HTML, CSS and JavaScript in a single file, `index.html`. No frameworks, no build step, no dependencies, no tests, no linter. Keep it that way: don't introduce a framework, bundler or package.json unless asked. To run it, open `index.html` in a browser.

## Architecture (all in `index.html`)

- **State** is one array, `todos`, of `{id, text, category, priority, due, remind, reminded, done}`. Every mutation (`addTodo`, `saveEdit`, `toggleTodo`, `deleteTodo`) changes the array, calls `save()`, then `render()`, which rebuilds the whole list from scratch. There is no incremental DOM updating.
- **Persistence** is `localStorage` under the keys `todos` and `theme`. Every access is wrapped in try/catch, because storage can be unavailable (private windows, `data:` URLs, the IDE preview pane). The app must still work without it. Items saved by older versions are migrated at load: a missing priority becomes Medium and an unknown category becomes Others.
- **Rendering**: `render()` groups items into sections in the fixed order of `CATEGORIES` (Work, Personal, Others) and sorts each by `PRIORITY_ORDER`, then by due date (`byDue`). `buildItem` builds a row in either view or edit mode. Edit and delete use the `iconButton()` helper (✏️ and 🗑️).
- **Reminders**: `setInterval(checkReminders, 15000)` compares each item's `remind` datetime with the current time. The in-page alert banner (`renderAlerts`) is always shown. A desktop notification (`notify`) is added only if the Notification permission was granted (`askNotificationPermission`).
- **Theme**: CSS custom properties on `:root`, overridden by `:root[data-theme="dark"]`. The initial theme comes from the saved choice, falling back to `prefers-color-scheme`. `applyTheme()` sets the attribute and the toggle icon (🌙 or ☀️).
- Text is always inserted with `textContent`, never `innerHTML`.

## Gotchas

- Top-level `let theme` is a string, so it shadows the `#theme` element's auto-created global. Use `document.getElementById('theme')` to reach the button.
- When adding a field to the todo shape, also handle old saved items in the load-time migration.

## mockup.html is generated, not hand-edited

`mockup.html` shows the app in an iPhone 16 frame (light) and a Galaxy S25 frame (dark), using iframes whose `srcdoc` is an encoded copy of `index.html` with sample items injected. It is a snapshot: it does not follow changes to `index.html`. The generator is a PowerShell script that lives in the session scratchpad rather than in this folder, so it may be gone. If it is missing, recreate it rather than editing `mockup.html` by hand. The script strips the `localStorage` reads and writes from the embedded copies, so opening the mock-up cannot overwrite real saved data (file:// pages share storage in Chrome). Keep that behaviour.

## Environment

Windows, PowerShell 5.1: `&&` is not available, so chain with `;`. The folder is not a git repository.
