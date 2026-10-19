# Group board

Arrange the people taking part in an event into groups, such as the working
groups of a forum. Each person is a card. Cards sit in group columns or in an
**Unassigned** tray, and you drag them between columns. You can add, edit and
delete people and groups, mark one lead per group, search, undo mistakes and
save the board to its JSON file.

The tool is one self-contained HTML file, `group-board.html`. It has no build
step, loads nothing from the network and holds no data of its own: every board
lives in its own JSON file.

## Open it

- Download `group-board.html` and open it in any modern browser. No server or
  installation is needed.
- Keep each board's JSON file with the event it belongs to and open it
  through **Load**.

The page opens on a blank board, titled "New board", with two empty groups,
ready to edit. **New** starts another blank board. To continue an existing
board, select **Load** (or drop a `.json` file onto the page).

## Load and save

- **Load** uses the browser's file picker. The whole file is checked before
  anything is shown. The board is rejected if it:
  - belongs to another tool,
  - comes from a newer format version,
  - is not valid JSON, or
  - breaks any rule below.

  The message names the problem and the current view stays as it was. Nothing
  is coerced, dropped or partly loaded.
- Nothing is written until you select **Save**. A page opened from a local file
  cannot write a file silently.
  - Where the browser offers a save dialog, Save opens it with the name of the
    file you loaded. Choose the project folder.
  - Otherwise the browser downloads the file under that name (or
    `group-board.json` if nothing was loaded). Move it to its project folder.
- There is no browser autosave. The page warns before you close it with unsaved
  changes.
- Saving keeps identifiers, order and any fields the tool does not recognise,
  at every level. The file is written as JSON indented by two spaces.
  - Saving without edits gives the same content as the loaded file.
  - Saving after edits differs only by those edits.

## Editing

- **Add person** opens an empty form for name, organisation, role, tags
  (separated by commas), note, group and lead.
- **Add people…** adds a list, one person per line, to **Unassigned**.
  - Separate organisation and role with a tab, as when pasting from a
    spreadsheet, or a semicolon: `Name; Organisation; Role`.
  - Blank lines are ignored. Lines without a name are skipped and counted.
- Drag a card to another column to move that person. A person is in exactly
  one group, or in **Unassigned**.
- Click a card, or focus it and press Enter, to edit that person, change their
  group or lead role, or delete them. Dragging is never required.
- **Add group** adds a group with the next palette colour and opens its form.
  - A group's **Edit** button changes its name, colour, note and lead.
  - **◀ Earlier** and **Later ▶** reorder the groups.
  - **Delete** removes the group and moves its members to **Unassigned**.
    Nobody is deleted.
- A group has at most one lead, marked ★. Only a member can be lead. Moving the
  lead out of the group, or deleting them, leaves the group without a lead.
- Each column shows how many people it holds and how often each tag appears.
- **Search** highlights people whose name, organisation, role, note or tag
  contains the text and dims the rest. The search never changes the board.
- **Title…** renames the board.
- **Undo** reverses edits one at a time, up to 150 steps. It also undoes
  loading a file, **New** and **Reset**. **Reset** returns to the file as last
  loaded.
- Printing gives a clean page of the board without the toolbar.

## JSON format, version 1

This section is the normative reference for the format.

```json
{
  "tool": "ngo-tools-group-board",
  "version": 1,
  "title": "Synthetic Forum working groups",
  "groups": [
    { "id": "research", "label": "Research", "colour": "#2f5d8a", "note": "Fictional priorities", "lead": "p1" },
    { "id": "funding", "label": "Funding", "colour": "#8a4f7d" }
  ],
  "people": [
    { "id": "p1", "name": "Alex Example", "organisation": "Example University", "role": "Researcher", "tags": ["science"], "group": "research" },
    { "id": "p2", "name": "Mar Ficticia", "note": "Available on day 2 only" }
  ]
}
```

| Field | Rule |
| --- | --- |
| `tool` | Exactly `"ngo-tools-group-board"`. |
| `version` | Whole number. `1` is read; a higher number is rejected as made by a newer version of the tool. |
| `title` | Non-empty text. |
| `groups` | Ordered list, possibly empty. Each group has `id` and `label` (non-empty text) and `colour` (`#rrggbb`). It may have `note` (text) and `lead` (the id of a person whose `group` is this group). |
| `people` | Ordered list, possibly empty. Each person has `id` and `name` (non-empty text). They may have `organisation`, `role` and `note` (text), `tags` (a list of unique, non-empty texts) and `group` (an id from `groups`). |

- A person without `group` is in **Unassigned**.
- Identifiers are unique within `groups` and within `people`.
- New people get the next free id of the form `person-N` and new groups
  `group-N`. Both are appended to their list.
- Clearing an optional text field in a form removes the field rather than
  saving empty text.

Any other field, at any level, is kept unchanged.

## Example

`example.json` is a synthetic forum board. It has four working groups, three
of them with a lead, and fourteen fictional people with tags and notes. Three
of the people are unassigned. It contains no real event data.
