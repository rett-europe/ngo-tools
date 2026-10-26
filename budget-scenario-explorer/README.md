# Budget scenario explorer

A what-if explorer for a draft annual budget that includes one event, such as
a congress. You keep a **current budget** in absolute amounts: the event's
cost lines, the running income and costs, and the membership fees. On top of
it you pull **levers**: sponsor numbers and prices, a grant, member
contributions, how many people come, a price change for cost of living or
another country, contingency, and scenario amounts that replace a current
figure. The page recomputes the result, the end-of-period cash position, the
event funding gap and the largest single funder. You can pin scenarios to
compare them side by side.

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
4. Edit the current budget. Every scenario shares it:
   - **Event costs:** open **Edit cost lines and drivers** under the cost
     table.
     - Each line has an item, a quantity, up to two drivers and a unit cost.
       A line "Lunches, 2 × Participants × 32,00" costs 2 × 80 × 32,00 when
       80 participants are planned.
     - **Drivers** are the counts behind the lines, such as participants,
       rooms, nights or funded travellers. Each has a planned number.
     - Lines are grouped into categories, for example accommodation,
       catering, meeting rooms, audiovisual, organisation and staff, and
       funded travel. You can rename, add, delete and move them.
     - A category can be deleted only when it has no lines, and a driver only
       when no line uses it.
   - **Running income and costs:** add, rename or remove income categories,
     such as donations or online fundraising, each with its current amount
     for the period. Running costs is one amount for the period.
   - **Membership fees:** the current fee per member for each band.
5. Pull the levers, or start from a preset:
   - **Drivers:** for example 100 participants instead of 80. Every line that
     uses the driver follows.
   - **Price change:** a percentage on all event costs, for cost of living or
     a different country. **Contingency** is a percentage on the subtotal.
   - **Scenario amounts:** type an amount next to a cost category, an income
     category, the running costs or a band's fee to replace the current
     figure in this scenario. Leave it empty to keep the current figure.
   Levers and scenario amounts are for exploring and are not saved unless you
   pin them.
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

## Data format (version 3)

The file is one JSON object. Unknown fields are kept unchanged when you save.
Amounts are whole numbers in **minor units** (for example cents) in one
currency.

| Field | Meaning |
| --- | --- |
| `tool` | Always `"ngo-tools-budget-scenario-explorer"`. |
| `version` | Format version, `3`. Files in any other format version are refused. |
| `title`, `subtitle` | Page heading. `subtitle` is optional. |
| `periodLabel`, `eventLabel` | Names for the budget period and the event. |
| `currency` | One three-letter currency code, for example `"EUR"`. |
| `minorDigits` | Optional number of digits after the decimal point, 0–4. Default 2. |
| `numberFormat` | Optional. `"eu"` (default) shows 1.234,56; `"en"` shows 1,234.56. |
| `cash` | `start` amount, a `history` list of `{label, amount}` points, and `endLabel`. |
| `running` | `income`: a list of `{id, label, amount}` income categories. `costs`: the running costs for the period. |
| `eventCosts` | `drivers`: a list of `{id, label, plan}`, where `plan` is a whole number. `categories`: a list of `{id, label}`. `lines`: a list of `{id, category, item, quantity, per?, unitCost}`, where `quantity` is a whole number and `per` is an optional list of driver ids. |
| `sponsorTiers` | Exactly two tiers, each `{label, baselinePrice}`. |
| `memberTiers` | Exactly three member contribution tiers, each `{label, baselineAmount}`. |
| `grant` | `{label}` for an unconfirmed grant. |
| `fixedContribution` | `{label, amount}` that is always included in member funding. |
| `bands` | Membership fee bands, each `{id, label, fee}`, where `fee` is the current fee per member. |
| `members` | Each `{id, name, band, info?}`. Used to count the members in each band. |
| `groups` | A title and optional help text for each control group: `sponsors`, `grant`, `associations`, `fees`, `running`, `event`. |
| `controls` | `{label, min, max, step}` for each slider (see the list after this table). |
| `defaults` | A complete set of values: every slider, plus `drivers` and `overrides`. |
| `presets` | Each `{id, name, values}`. `values` overrides `defaults`; `drivers` and each `overrides` map are merged by id. |
| `startPreset` | Optional preset `id` shown after loading. |
| `scenarios` | Pinned scenarios, each `{name, values}` with a complete set of values. |
| `notes` | Optional list of caveats shown at the foot of the page. |

`controls` has one entry for each of these sliders:

- `tier1Count`, `tier1Price`, `tier2Count`, `tier2Price`
- `member1Count`, `member1Amount`, `member2Count`, `member2Amount`,
  `member3Count`, `member3Amount`
- `grant`, `payShare`
- `priceChange`, `contingency`

`priceChange` is a percentage change and may go as low as −100. All other
sliders start at 0 or more.

Besides the sliders, a set of values has:

- `drivers`: `{driverId: count}`. A driver that is not listed is at its plan.
- `overrides`: scenario amounts that replace a current figure.
  - `costs`: `{categoryId: amount}`, the final amount of an event cost
    category. Price change does not apply to it; contingency does.
  - `income`: `{incomeId: amount}`.
  - `fees`: `{bandId: fee per member}`.
  - `runningCosts`: optional amount.

Every value must lie within its control's range and on its step, and every
id must name an existing driver, category, income line or band. Otherwise the
tool refuses the file and names the field.

Older files must be converted by hand. For version 2:

- Set `version` to 3 and add `eventCosts.drivers` (it may be empty).
- Replace `generalUplift` with `priceChange`.
- Replace each `categoryAdjust` entry with drivers on the lines, or with
  `overrides.costs`.
- Replace `feeChange` with `overrides.fees`.

For version 1, first turn the event costs into cost lines, running income
into income lines and give each band one fee.

## Calculation

Event cost:

- Line amount = quantity × each of its drivers' counts × unit cost.
- Planned cost of a category = sum of its line amounts with drivers at plan.
- Category cost = its scenario amount if set; otherwise the sum of its line
  amounts with the scenario's driver counts × (100 + priceChange) ÷ 100.
- Subtotal = sum of the category costs.
- Contingency = subtotal × contingency ÷ 100.
- Event cost = subtotal + contingency. The plan column uses the planned costs
  and the default contingency.

Funding:

- Sponsors = count × price, for each tier.
- Member funding = fixed contribution + members × contribution, for each of
  the three member tiers.

Fees, for each band:

- Fee = scenario fee if set, otherwise the current fee.
- Fees due = fee × members in the band.
- Fees paid = fees due × payShare ÷ 100. These are summed over the bands.

Results:

- Running result = fees paid + income (scenario amount or current) − running
  costs (scenario amount or current).
- Event result = sponsors + member funding + grant − event cost.
- Result = running result + event result.
- Cash at end = cash start + result.

Each step that involves a division is rounded **half away from zero** to a
whole minor unit at that point. These steps are each category's price-changed
cost, the contingency and each band's fees paid. Everything else is exact
integer arithmetic.

The largest-funder share is shown to a tenth of a percent, rounded the same
way. It reads "n/a" when there is no cost or no funder. Shares and the chart
axis labels are for display only.