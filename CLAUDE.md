# Notion Widgets

A collection of self-contained HTML widgets that are embedded in Notion pages. `notion-calendar/weekly-calendar.html` is the reference implementation: when in doubt, do what it does.

## Repository layout

- One folder per widget, holding one `.html` file: `<widget-folder>/<widget-name>.html`.
- Every widget gets its own subsection under **Widgets** in `README.md` (what it does, features, how its data is saved, any setup such as an ID constant).

## Hard constraints

A widget is pasted into a Notion HTML/embed block and runs inside Notion's **sandboxed iframe**. Anything that breaks there is a bug, even if it works when the file is opened directly.

- **One self-contained file.** All CSS and JS inline. No build step, no external scripts, stylesheets, fonts, images or network requests. Icons are inline SVG.
- **No form submission.** The sandbox blocks it silently: `Blocked form submission … 'allow-forms' permission is not set`. The submit handler never runs.
  - Never use `<form>`, `type="submit"`, `form.submit()` or `requestSubmit()`.
  - Buttons are `type="button"` with click handlers, and Enter-to-save is handled with an explicit `keydown` listener.
- **No `alert`, `confirm` or `prompt`.** Sandboxes without `allow-modals` block them. Use in-page dialogs (the calendar's `.overlay` / `.dialog` and its `askScope` pattern) and toasts.
- **No top-level navigation or reliance on popups.** Downloads use a temporary `<a download>` with a Blob URL.
- **Storage may be missing.** `localStorage` can throw or be denied.
  - Probe it once inside `try/catch` and keep working without it.
  - When it's unavailable, tell the user that changes won't survive a reload, and offer the snippet export (see Persistence).
- **The clipboard may be blocked.** Try `navigator.clipboard`, then fall back to selecting text in a textarea plus `execCommand('copy')`, and tell the user to copy manually if both fail.
- **Fill the iframe.** Use `html, body { height: 100% }`, a sensible `min-height`, and a usable layout down to about 360px wide. Scroll inside the widget, never the page sideways.

## Persistence

Use the calendar's model:

1. **`localStorage`** under a key built from a `WIDGET_ID` constant, e.g. `notion-cal:` + `CALENDAR_ID`. This constant must be easy to find and change at the top of the script, so several copies on one page don't share data. Document it in the README.
2. **Embedded data.** Put a `<script id="…-data" type="application/json">null</script>` tag in the page. A "Copy HTML snippet with data" action re-serializes the pristine page source with the current state baked in. This is the only way data reaches other devices and other viewers. Escape `<` as `<` when embedding.
3. Every save stamps `savedAt`. On load, **whichever copy (embedded or local) is newer wins**.
4. Pass all loaded data, whether embedded, local or imported JSON, through a `normalize()` function. It validates types and ranges, fills in defaults, and drops malformed entries. Old data must keep loading after the schema grows: new fields get defaults.
5. Listen for the `storage` event so open copies in other tabs stay in sync.
6. Offer **Backup & restore**: copy JSON, download JSON, load JSON, and copy the snippet with data.

## Look and feel

- Match Notion:
  - **Font stack:** `ui-sans-serif, -apple-system, BlinkMacSystemFont, "Segoe UI", …` at 14px.
  - **Text color:** `#37352f` in light mode.
  - **Lines:** subtle, around `#ededec`.
  - **Accent:** `#2383e2`.
  - **Corners and shadows:** small radii, Notion-like shadows.
- Define colors as CSS custom properties on `:root`.
  - Redefine them for dark mode under `@media (prefers-color-scheme: dark) { :root:not([data-theme="light"]) { … } }`, and again under `:root[data-theme="dark"]`.
  - Offer a Theme setting: system, light or dark.
  - Give `body` an explicit background.
- User-pickable colors use Notion's palette: gray, brown, orange, yellow, green, blue, purple, pink and red, each with light and dark background/accent pairs. See `COLORS` in the calendar.
- Follow the system locale for dates and times (`toLocaleDateString`, `Intl`). Offer a 12/24-hour setting wherever times are shown.

## Code conventions

- One `<script>` containing `(() => { 'use strict'; … })();`.
- Use small helpers (`$`, `pad`, `clamp`, `uid`, `esc`) and sections marked `// ---------- name ----------`.
- Comments are sparse and explain *why*.
- **Escape all user text** with `esc()` before it goes into `innerHTML`, or use `textContent`.
- Route state changes through one `commit(mutate)`, which snapshots for undo, mutates, saves and re-renders. Support Ctrl/Cmd+Z, and offer Undo in the toast after destructive actions.
- **Keyboard:**
  - Esc closes the topmost overlay.
  - Interactive elements are real buttons, or have `tabindex`, `role` and `aria-label`.
  - Shortcuts are ignored while the user is typing in an input.

## Testing (required before calling a change done)

Test in a real headless browser. Reading the code is not enough, and passing tests in an unsandboxed page proves nothing about Notion.

**Tooling.** Use Playwright with Chromium.
- If it isn't installed, look for a cached copy (`find ~/.npm/_npx -maxdepth 3 -name playwright -type d`) and `require()` it by path. Otherwise, ask before installing anything.
- Keep test scripts in the scratchpad directory, not in the repo.

**Harness.** Load the widget into an iframe that copies Notion's restrictions: `sandbox="allow-scripts allow-same-origin allow-popups"`, with **no `allow-forms` and no `allow-modals`**. Set it through `srcdoc` from a page created with `page.setContent(...)`:

```js
await page.setContent('<iframe id=f sandbox="allow-scripts allow-same-origin allow-popups" style="width:1150px;height:850px"></iframe>');
await page.$eval('#f', (f, html) => { f.srcdoc = html; }, html);
const frame = page.frames()[1];
```

In this setup `localStorage` is denied, so it also exercises the **no-storage path**. To test the storage path, load the file directly with `page.setContent(html)` as well.

**What to check:**
- The new or changed feature works end to end through real mouse and keyboard input (`page.mouse`, `frame.click`, `frame.press`), not by calling internal functions.
- Saving works through every entry point: buttons, Enter, shortcuts.
- Undo restores the previous state, and cancel/Escape leaves state untouched.
- Data round-trips: put JSON in the embedded data tag, reload, and check what renders. Include old-format data without the newest fields.
- Read state through the UI, e.g. the Backup dialog's JSON textarea, since `localStorage` isn't readable in the sandboxed harness.
- There are **no console errors** and **no `Blocked …` messages**. Collect them with `page.on('console')` and `page.on('pageerror')`.
- Screenshot the changed UI and look at it in **light and dark** themes, and at a narrow width (about 400px).

**Pitfalls:**
- Playwright bounding boxes are page coordinates.
- Sticky headers and footers inside the widget cover parts of scrolled content. Aim pointer actions well inside the visible area and confirm the target with `elementFromPoint` when an action seems to do nothing.

**Reporting.** Report results honestly: name what was tested, what passed, and anything that couldn't be tested.

## Finishing a change

- Update the widget's README subsection when behavior or features change.
- Commit or push only when asked.
- Users update a widget by pasting the new file over the old code in Notion. If a change affects stored data, keep `normalize()` backward compatible. Remind the user to copy the snippet with data first if their data lives in the embed.
