# Notion Widgets

A collection of self-contained HTML widgets that can be embedded in a Notion page.

Each widget is a single `.html` file with its CSS and JavaScript inline, with no build step, no external dependencies and no server. Styling follows Notion's look (fonts, colors, light and dark mode), so the widgets sit naturally on a page.

## Using a widget

Either:

- **Paste the code** of the widget's `.html` file into an HTML embed block on your Notion page, or
- **Host the file** (for example with GitHub Pages) and add it to the page with Notion's `/embed` block, using the file's URL.

## Widgets

### Weekly Calendar

`notion-calendar/weekly-calendar.html`

A week-view calendar with time-blocked events and a per-day to-do list.

**Features**

- Week grid in 30-minute slots, with a current-time indicator and today highlighted
- Drag on the grid to create an event; click an event to edit its title, date, time, notes and color (Notion's color palette)
- Recurring events: repeat every day, every weekday, weekly, every 2 weeks, monthly or yearly, with an optional last day. Editing or deleting an occurrence asks whether to apply it to this event, this and following events, or all events. Dragging a single occurrence moves just that one.
- To-do list under each day, with checkboxes and a collapsible row
- Undo (`Ctrl/Cmd + Z`), plus keyboard shortcuts: `←` / `→` to change week, `T` to jump to today
- Settings: first day of the week, show/hide weekends, visible hour range, default scroll position, row height, 12/24-hour time, light/dark/system theme, default event color

**Saving data**

Events, to-dos and settings are saved in the browser's `localStorage`, so they stay on the device and browser where you entered them. To keep them in Notion itself, so they show on every device and for everyone who views the page, open **Backup & restore…** from the `⋯` menu, click **Copy HTML snippet with data**, and paste the result into the HTML block, replacing the old code. The data is stored in the `<script id="cal-data">` tag, and whichever copy (embedded or local) was saved most recently is used.

The same dialog can copy, download or load the data as JSON.

If the embed can't use browser storage, the calendar shows a banner and changes are lost on reload unless you save them as a snippet.

**Multiple calendars**

If you embed more than one calendar, change `CALENDAR_ID` near the top of the script in each copy so they don't share events.

### Habit Streak

`habit-streak/habit-streak.html`

A daily checklist of your habits with a strict all-or-nothing streak.

**Features**

- Each day shows your habits as tiles. Click one (or focus it and press `Space`) to mark it done: the tile turns solid green with a check mark in the middle. Click again to undo the check.
- When every habit for today is done, the widget turns orange and shows a flame with your current streak: the number of days in a row you completed **every** habit. **Show today's habits** goes back to the list, and the flame button in the toolbar returns to the streak.
- No cheat days. Missing even one habit on a day resets the streak to 0. Today only counts against you once it's over, so the streak shown during the day is the run up to yesterday, and finishing today adds one. Past days can't be checked off.
- Add habits from the box under the list, or rename, reorder and delete them in **Edit habits…** (`⋯` menu). A new habit counts from the day you add it. Deleting a habit stops it counting from today, so past days are still judged by the habits you had then.
- At midnight the list resets to unchecked for the new day.
- Undo (`Ctrl/Cmd + Z`), with an Undo button after deleting a habit
- Settings: light/dark/system theme

**Saving data**

Habits, the daily check-off history and settings are saved in the browser's `localStorage`. To keep them in Notion itself, so they show on every device and for everyone who views the page, open **Backup & restore…** from the `⋯` menu, click **Copy HTML snippet with data**, and paste the result into the HTML block, replacing the old code. The data is stored in the `<script id="habit-data">` tag, and whichever copy (embedded or local) was saved most recently is used. Because the streak depends on checking habits off every day, a tracker that relies on the embedded snippet needs to be re-pasted after each day's check-offs. Use it in a browser where storage works if you can.

The same dialog can copy, download or load the data as JSON.

If the embed can't use browser storage, the widget shows a banner and changes are lost on reload unless you save them as a snippet.

**Multiple trackers**

If you embed more than one tracker, change `HABITS_ID` near the top of the script in each copy so they don't share habits.
