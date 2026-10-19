# Calendar builder

Plan a multi-day, multi-track event agenda. Each day is a column split into the
event's tracks on a time grid, with unscheduled blocks in a tray. You can drag,
resize, add, edit and delete blocks, see overlaps and per-track totals, undo
mistakes and save the agenda back to its JSON file.

The tool is one self-contained HTML file, `calendar-builder.html`. It has no
build step, loads nothing from the network and holds no data of its own: every
event lives in its own JSON file.

## Open it

- Download `calendar-builder.html` and open it in any modern browser. No
  server or installation is needed.
- Keep each event's JSON file with the project it belongs to and open it
  through **Load**.

The page opens on a blank event (one day, two tracks, 09:00–18:00) that is
ready to edit. Select **Event settings…** to set the title, days, tracks, hours,
grid and block types, one per line; lines match existing entries by position,
so renaming keeps blocks. Blocks on a removed day or track, or outside new
hours, move to the tray; a block type still in use cannot be removed. **New**
starts another blank event. To continue an existing event, select **Load** (or
drop a `.json` file onto the page).

## Load and save

- **Load** uses the browser's file picker. A file is checked in full before
  anything is shown. If it is for another tool, from a newer format version, not valid
  JSON or breaks any rule below, the message names the problem and the current
  view stays as it was. Nothing is coerced, clamped, dropped or partly loaded.
- **Save** is the only way anything is written, and only when you select it.
  A page opened from a local file cannot write a file silently. Where the
  browser offers a save dialog, Save opens it with the name of the file you
  loaded; choose the project folder. Otherwise the browser downloads the file
  under that name (or `calendar.json` if nothing was loaded) and you move it
  to its project folder.
- There is no browser autosave. The page warns before closing with unsaved
  changes.
- Saving keeps identifiers, order and any fields the tool does not recognise,
  at every level. A save without edits equals the loaded file; a save after
  edits differs only by those edits. The file is written as JSON indented by
  two spaces.

## Editing

- Drag a block to another time, day, track or the tray. Drag its top or bottom
  edge to resize it. Changes snap to the grid, stay within the time window and
  keep at least one interval. An all-tracks block keeps spanning all tracks
  when dragged; change its track in the form.
- Click a block (or focus it and press Enter) to edit its title, type, day or
  tray, track (including all tracks), start, end or duration, and note. Delete
  is in the same form. **Add block** opens an empty form. Dragging is never
  required.
- Moving or resizing a block removes its approximate marker. Editing only text
  keeps it.
- **Undo** reverses edits one at a time, up to 150 steps, including loading a
  file and reset. **Reset** returns to the file as last loaded.
- Two placed blocks on the same day overlap when their times intersect with
  positive length and they share a track; an all-tracks block shares every
  track. Both are outlined and marked ⚠. Blocks that only touch do not overlap.
- The totals table shows, per day and track, the time in each block type that
  counts towards totals. An all-tracks block counts in every track.

## JSON format, version 1

This section is the normative reference for the format.

```json
{
  "tool": "ngo-tools-calendar-builder",
  "version": 1,
  "title": "Synthetic Summit",
  "window": { "start": "09:00", "end": "18:00" },
  "interval": 30,
  "days": [{ "id": "d1", "label": "Day 1" }],
  "tracks": [{ "id": "a", "label": "Track A" }, { "id": "b", "label": "Track B" }],
  "types": [{ "id": "session", "label": "Session", "colour": "#3b6ea5", "counts": true }],
  "blocks": [
    { "id": "k1", "title": "Fictional keynote", "type": "session", "day": "d1", "track": "*", "start": "09:00", "end": "10:00", "approx": true },
    { "id": "t1", "title": "Unscheduled talk", "type": "session", "duration": 60, "note": "Speaker to confirm" }
  ]
}
```

| Field | Rule |
| --- | --- |
| `tool` | Exactly `"ngo-tools-calendar-builder"`. |
| `version` | Whole number. `1` is read; a higher number is rejected as made by a newer version of the tool. |
| `title` | Non-empty text. |
| `window.start`, `window.end` | Times `HH:MM` from `00:00` to `24:00`, end after start. The daily time window shown for every day. |
| `interval` | Optional grid step in whole minutes, default `30`. Must divide the window length exactly. If absent it stays absent when saved. |
| `days` | Ordered list, at least one: `id` and `label`, both non-empty text. |
| `tracks` | Ordered list, at least one: `id` and `label`, both non-empty text. `"*"` is reserved and cannot be a track id. |
| `types` | List: `id` and `label` (non-empty text), `colour` (`#rrggbb`) and `counts` (`true` or `false`, whether the type counts towards totals). |
| `blocks` | Ordered list of blocks, each placed or unplaced, as below. |

Every block has `id` and `title` (non-empty text) and `type` (an id from
`types`). It may have `note` (text) and `approx` (`true` or `false`; marks a
published time rounded to the grid).

- A **placed** block has `day` (an id from `days`), `track` (an id from
  `tracks`, or `"*"` for all tracks), and `start` and `end` (`HH:MM`). Both
  times are on the grid counted from `window.start`, inside the window, and
  `end` is after `start`. It has no `duration`.
- An **unplaced** block sits in the tray. It has `duration`, a positive whole
  number of minutes that is a multiple of `interval` and no longer than the
  window. It has none of `day`, `track`, `start` or `end`.

Identifiers are unique within each of `days`, `tracks`, `types` and `blocks`.
Times are local wall-clock times without a time zone, and blocks do not cross
midnight. Moving a block to the tray replaces its placement with `duration`
(its length); placing it again does the reverse. New blocks get the next free
id of the form `block-N` and are appended to `blocks`.

Any other field, at any level, is kept unchanged.

## Example

`example.json` is a synthetic three-day, two-track event with session,
workshop and non-counting break types, all-tracks blocks, approximate times
and two unscheduled blocks. It contains no real event data.
