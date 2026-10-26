# Budget scenario explorer

A what-if explorer for a draft annual budget that includes one event, such as
a congress. You edit the event's cost lines, the running income and the
membership fees. You then move sliders for sponsor numbers and prices, a
grant, member contributions, fee changes and cost adjustments. The page
recomputes the result, the end-of-period cash position, the event funding gap
and the largest single funder. You can pin scenarios to compare them side by
side.

The tool is one self-contained HTML file. It needs no network, installs
nothing and holds no data of its own.

## Use it

1. Download `budget-scenario-explorer.html` and open it in any modern
   browser. No server or installation is needed.
2. The page opens on a made-up starting budget, so you can explore straight
   away. Choose **Save JSON…** to keep it as your own file, and keep that
   budget file **with the project it belongs to**.
3. To open an existing budget, choose **Load JSON…**, or drop the JSON file
   onto the page.
4. Edit the budget lines. Every scenario shares these lines:
   - **Event costs:** open **Edit cost lines** under the event budget.
     - Each line has an item, a quantity and a unit cost.
     - Lines are grouped into categories, for example accommodation,
       catering, meeting rooms, audiovisual, organisation and staff, and
       funded travel.
     - You can rename, add and delete categories and lines, and move a line
       to another category.
     - A category can be deleted only when it has no lines.
   - **Running income and costs:** add, rename or remove income categories,
     such as donations or online fundraising. Each category has an absolute
     amount for the period. Running costs is one amount for the period.
   - **Membership fees:** set a base fee per member for each band.
5. Move the sliders, or start from a preset. Slider settings are for
   exploring and are not saved.
6. To keep a scenario, name it and choose **Pin current settings**. Pinned
   scenarios are written into the JSON when you save.
7. Choose **Save JSON…**. Browsers that support the save dialog suggest the
   name of the file you loaded and ask where to save, so choose the project
   folder. Other browsers save to Downloads; move the file to its project
   folder.

A page opened from a file cannot write a file silently, so saving is always
an explicit action, either through the save dialog or as a download. There is
no autosave; the JSON file is the durable record. The browser warns before
you close the page with unsaved edits or pinned scenarios.

## Data format (version 2)

The file is one JSON object. Unknown fields are kept unchanged when you save.
Amounts are whole numbers in **minor units** (for example cents) in one
currency.

| Field | Meaning |
| --- | --- |
| `tool` | Always `"ngo-tools-budget-scenario-explorer"`. |
| `version` | Format version, `2`. Files in any other format version are refused. |
| `title`, `subtitle` | Page heading. `subtitle` is optional. |
| `periodLabel`, `eventLabel` | Names for the budget period and the event. |
| `currency` | One three-letter currency code, for example `"EUR"`. |
| `minorDigits` | Optional number of digits after the decimal point, 0–4. Default 2. |
| `cash` | `start` amount, a `history` list of `{label, amount}` points, and `endLabel`. |
| `running` | `income`: a list of `{id, label, amount}` income categories. `costs`: the running costs for the period. |
| `eventCosts` | `categories`: a list of `{id, label}`. `lines`: a list of `{id, category, item, quantity, unitCost}`, where `quantity` is a whole number. |
| `sponsorTiers` | Exactly two tiers, each `{label, baselinePrice}`. |
| `memberTiers` | Exactly three member contribution tiers, each `{label, baselineAmount}`. |
| `grant` | `{label}` for an unconfirmed grant. |
| `fixedContribution` | `{label, amount}` that is always included in member funding. |
| `bands` | Membership fee bands, each `{id, label, fee}`, where `fee` is the base fee per member. |
| `members` | Each `{id, name, band, info?}`. Used to count the members in each band. |
| `groups` | A title and optional help text for each control group: `sponsors`, `grant`, `associations`, `fees`, `running`, `event`. |
| `controls` | `{label, min, max, step}` for each slider (see the list after this table). `categoryAdjust` is `{min, max, step}` and must allow 100. |
| `defaults` | A complete set of values: every slider plus `categoryAdjust`. |
| `presets` | Each `{id, name, values}`. `values` overrides `defaults`, and `categoryAdjust` is merged by category. |
| `startPreset` | Optional preset `id` shown after loading. |
| `scenarios` | Pinned scenarios, each `{name, values}` with a complete set of values. |
| `notes` | Optional list of caveats shown at the foot of the page. |

`controls` has one entry for each of these sliders:

- `tier1Count`, `tier1Price`, `tier2Count`, `tier2Price`
- `member1Count`, `member1Amount`, `member2Count`, `member2Amount`,
  `member3Count`, `member3Amount`
- `grant`, `feeChange`, `payShare`
- `generalUplift`, `contingency`

`feeChange` and `generalUplift` are percentage changes and may go as low as
−100. All other sliders start at 0 or more.

`categoryAdjust` is an object `{categoryId: percent}`. It gives each event
cost category's share of its planned cost. A category that is not listed is at
100%, its planned cost.

Every value must lie within its control's range and on its step. Otherwise
the tool refuses the file and names the field. Version 1 files must be
converted by hand:

- Turn the event costs into cost lines.
- Turn running income into income lines.
- Give each band one base fee.
- Replace `feeOption` with `feeChange`.

## Calculation

Event cost:

- Planned cost of a category = sum of quantity × unit cost over its lines.
- Category cost = planned × categoryAdjust × (100 + generalUplift) ÷ 10,000.
- Subtotal = sum of the category costs.
- Contingency = subtotal × contingency ÷ 100.
- Event cost = subtotal + contingency.

Funding:

- Sponsors = count × price, for each tier.
- Member funding = fixed contribution + members × contribution, for each of
  the three member tiers.

Fees, for each band:

- Scenario fee = base fee × (100 + feeChange) ÷ 100.
- Fees due = scenario fee × members in the band.
- Fees paid = fees due × payShare ÷ 100. These are summed over the bands.

Results:

- Running result = fees paid + sum of income lines − running costs.
- Event result = sponsors + member funding + grant − event cost.
- Result = running result + event result.
- Cash at end = cash start + result.

Each step that involves a division is rounded **half away from zero** to a
whole minor unit at that point. These steps are each category cost, the
contingency, each band's scenario fee and each band's fees paid. Everything
else is exact integer arithmetic.

The largest-funder share is shown to a tenth of a percent, rounded the same
way. It reads "n/a" when there is no cost or no funder. Shares and the chart
axis labels are for display only.