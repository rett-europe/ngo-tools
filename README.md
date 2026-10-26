# NGO tools

Small, self-contained tools that [Rett Syndrome Europe](https://rettsyndrome.eu)
uses to plan events, budgets and working groups. We share them as
**inspiration**: use them as they are, adapt them, or build something better
for your own organisation.

Each tool is a single HTML file. Open it in a browser and it works offline:
no server, no installation, no build step, no account and no network requests.
Your data stays in a JSON file that you load and save explicitly.

| Tool | Purpose |
| --- | --- |
| [Calendar builder](calendar-builder/README.md) | Plan a multi-day, multi-track agenda on a time grid. |
| [Budget scenario explorer](budget-scenario-explorer/README.md) | Explore what-if scenarios for a draft budget that includes one event, starting from an absolute current budget (event cost lines with drivers, income categories, membership fees) with levers and scenario amounts on top. |
| [Group board](group-board/README.md) | Arrange event participants as cards into working groups or teams. |

## Try a tool

1. Download the tool's HTML file (or clone this repository).
2. Open it in a modern browser.
3. Choose **Load** and pick the tool's `example.json` to see it with
   synthetic data, or start from the blank page.

## How the tools are built

- **One file per tool.** Plain HTML, CSS and JavaScript with no dependencies.
- **Your data in a file.** Every JSON file starts with `"tool"` and
  `"version"`. A tool refuses files for another tool, files from a newer
  format version and malformed data, and says why. Fields a tool does not
  know are kept when it saves.
- **Explicit saving.** There is no autosave and nothing leaves your machine.
  Browsers with a save dialog suggest the name of the file you loaded;
  others save to Downloads.
- **Exact money.** Amounts are whole numbers in minor units (for example
  cents), in one currency per file.
- **Documented formats.** Each tool's README is the reference for its JSON
  format, and every `example.json` contains only synthetic data.

## Ideas for developers

These tools solve real needs for a small patient organisation, but they are
deliberately simple. Some directions you might take them:

- Shared or collaborative editing, for example on top of a cloud drive.
- Import and export to spreadsheets or calendar formats (CSV, iCal).
- Translations and accessibility improvements.
- New tools for other NGO tasks: volunteer rotas, grant tracking, membership
  fees, travel bursaries.

Issues and pull requests are welcome. Please never include real personal or
financial data in examples, issues or tests.

## Licence

[MIT](LICENSE)
