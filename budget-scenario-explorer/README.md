# Budget scenario explorer

A slider-driven what-if explorer for a draft annual budget that includes one
event. You change sponsor numbers and prices, a grant, member contributions,
membership fees, giving and event cost uplifts. The page recomputes the
result, the end-of-period cash position, the event funding gap and the
largest single funder. Pin scenarios to compare them side by side.

The tool is one self-contained HTML file. It needs no network, installs
nothing and holds no data of its own.

## Use it

1. Download `budget-scenario-explorer.html` and open it in any modern
   browser. No server or installation is needed.
2. Keep the budget JSON file **with the project it belongs to**. The page
   opens on a made-up starting budget, so
   you can explore straight away. Choose **Save JSON…** to keep it as your own
   file in the project folder, then edit its figures.
3. To open an existing budget, choose **Load JSON…**, or drop the JSON file
   onto the page.
4. Move the sliders, or start from a preset. Slider changes are for
   exploring and are not saved.
5. To keep a scenario, name it and choose **Pin current settings**. Pinned
   scenarios are written into the JSON when you save.
6. Choose **Save JSON…**. Browsers that support the save dialog suggest the
   name of the file you loaded and ask where to save; choose the project
   folder. Other browsers save to Downloads; move the file to its project
   folder.

A page opened from a file cannot write a file silently, so saving is
always an explicit action through the save dialog or a download. There is no
autosave. The JSON file is the durable record. The browser warns before you
close the page with unsaved pinned scenarios.

## Data format (version 1)

The file is one JSON object. Unknown fields are kept unchanged when you save.
Amounts are whole numbers in **minor units** (for example cents), in one
currency.

| Field | Meaning |
| --- | --- |
| `tool` | Always `"ngo-tools-budget-scenario-explorer"`. |
| `version` | Format version, `1`. Files from a newer format version are refused. |
| `title`, `subtitle` | Page heading. `subtitle` is optional. |
| `periodLabel`, `eventLabel` | Names for the budget period and the event. |
| `currency` | One three-letter currency code, for example `"EUR"`. |
| `minorDigits` | Optional digits after the decimal point, 0–4. Default 2. |
| `cash` | `start` amount, `history` list of `{label, amount}` points, and `endLabel`. |
| `running` | `otherIncome`, `costs`, `givingBaseline` and `givingLabel` for day-to-day running. |
| `eventCosts` | Baseline `venue`, `other` and `travel` costs and the baseline number of `travellers`. |
| `sponsorTiers` | Exactly two tiers, each `{label, baselinePrice}`. |
| `memberTiers` | Exactly three member contribution tiers, each `{label, baselineAmount}`. |
| `grant` | `{label}` for an unconfirmed grant. |
| `fixedContribution` | `{label, amount}` that is always included in member funding. |
| `feeOptions` | Labels for the fee levels. |
| `bands` | Membership fee bands, each `{id, fees}` with one fee per fee option. |
| `members` | Each `{id, name, band, info?}`; used to count members per band for membership fees. |
| `groups` | Title and optional help text for each control group: `sponsors`, `grant`, `associations`, `fees`, `running`, `event`. |
| `controls` | `{label, min, max, step}` for each slider: `tier1Count`, `tier1Price`, `tier2Count`, `tier2Price`, `member1Count`, `member1Amount`, `member2Count`, `member2Amount`, `member3Count`, `member3Amount`, `grant`, `payShare`, `giving`, `generalUplift`, `venueUplift`, `contingency` and `travellers`. |
| `defaults` | A complete set of values: every slider and `feeOption` (index). |
| `presets` | Each `{id, name, values}`; `values` overrides `defaults`. |
| `startPreset` | Optional preset `id` shown after loading. |
| `scenarios` | Pinned scenarios, each `{name, values}` with a complete set of values. |
| `notes` | Optional list of caveats shown at the foot of the page. |

Every value must lie within its control's range and on its step; the tool
refuses the file and names the field otherwise.

## Calculation

With `g = 100 + generalUplift`, `v = 100 + venueUplift` and
`c = 100 + contingency`:

- Venue cost = venue × g × v × c ÷ 1,000,000.
- Other cost = other × g × c ÷ 10,000.
- Travel cost = travel × g × travellers × c ÷ (10,000 × baseline travellers).
- Event cost = venue cost + other cost + travel cost.
- Sponsors = count × price for each tier.
- Member funding = fixed contribution + members × contribution for each of
  the three member tiers.
- Fees = for each band, band fee for the chosen option × members in the band
  × payShare ÷ 100, then summed over the bands.
- Giving = givingBaseline × giving ÷ 100.
- Running result = fees + giving + otherIncome − running costs.
- Event result = sponsors + member funding + grant − event cost.
- Result = running result + event result; cash at end = cash start + result.

Each line marked with a division is rounded **half away from zero** to a whole
minor unit at that point: the three cost lines, each band's fees and giving.
Everything
else is exact integer addition. The largest-funder share is shown to a tenth
of a percent, rounded the same way, and reads "n/a" when there is no cost or
no funder. Shares and the chart axis labels are display only.
