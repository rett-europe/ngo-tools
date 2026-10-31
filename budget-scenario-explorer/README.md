# Budget scenario explorer

A what-if explorer for a draft annual budget that includes one event, such as
a congress. The page has two tabs:

- **1 · Baseline** holds the budget as you know it today, in absolute
  amounts: the event's cost lines, the planned driver counts, the running
  income and costs, the membership fees, the sponsor and member reference
  prices and the starting cash. This is the **Plan**.
- **2 · New edition** starts from the Plan and lets you design how the next
  edition could look with **levers**: sponsor numbers and prices, a grant,
  member contributions, how many people come, a price change for cost of
  living or another country, contingency, and a Change % or an exact new
  amount for any cost category, income line, the running costs or a fee.
  The page recomputes the result, the end-of-period cash position, the event
  funding gap and the largest single funder. You can pin scenarios to compare
  them side by side.

It is a high-level tool: the example works in whole euros and every
percentage is a whole number.

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
4. Edit the Plan on the **Baseline** tab. Every scenario shares it:
   - **Event cost plan:**
     - Each line has an item, a quantity, up to two drivers and a unit cost.
       A line "Lunches, 2 × Participants × 32" costs 2 × 80 × 32 when 80
       participants are planned.
     - Lines are grouped into categories, for example accommodation,
       catering, meeting rooms, audiovisual, organisation and staff, and
       funded travel. You can rename, add, delete and move them.
     - A category without lines takes a lump sum instead. Once it has lines,
       its Plan is their sum; adding its first line turns the lump sum into
       that line.
     - A category can be deleted only when it has no lines.
     - The table ends with the subtotal, the planned contingency and the
       event's Plan total.
   - **Drivers (planned counts):** the counts behind the lines, such as
     participants, rooms, nights or funded travellers. A driver can be
     deleted only when no line uses it.
   - **Running income and costs:** add, rename or remove income categories,
     such as donations or online fundraising, each with its amount for the
     period. Running costs is one amount for the period.
   - **Membership fees:** the fee per member for each band, with the fees
     due if every member pays.
   - **Sponsors, members and cash:** the reference price of each sponsor
     package and member contribution (the New edition sliders show their
     change against these), the fixed contribution and the cash at the start
     of the period.
5. Design the next edition on the **New edition** tab, or start from a
   preset:
   - **Drivers:** for example 100 participants instead of 80. Every line that
     uses the driver follows.
   - **Price change:** a percentage on all event costs, for cost of living or
     a different country. **Contingency** is a percentage on the subtotal.
   - **Change % per cost category:** step (±5) or type a percentage against
     its Plan. The New edition column shows the resulting amount and
     Difference the change from Plan. **Reset categories to Plan** clears them
     all. Plan is read-only here; change it on the Baseline tab.
   - **Change % per income line, running costs and fee band:** stepping sets
     the New edition amount to Plan × (100 + percent) ÷ 100, rounded.
   - **New edition amounts:** type an exact amount next to a cost category,
     an income line, the running costs or a band's fee to replace its Plan in
     this scenario; Change then shows the percentage it implies. Leave it
     empty to keep the Plan.
   Levers and New edition amounts are for exploring and are not saved unless
   you pin them.
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
Amounts are whole numbers in **minor units** of one currency. With
`minorDigits` 0, as in `example.json`, a minor unit is a whole euro (or
dollar), so nothing on the page shows decimals. Use 0 for a high-level
budget; use 2 only if you need cents.

| Field | Meaning |
| --- | --- |
| `tool` | Always `"ngo-tools-budget-scenario-explorer"`. |
| `version` | Format version, `3`. Files in any other format version are refused. |
| `title`, `subtitle` | Page heading. `subtitle` is optional. |
| `periodLabel`, `eventLabel` | Names for the budget period and the event. |
| `currency` | One three-letter currency code, for example `"EUR"`. |
| `minorDigits` | Optional number of digits after the decimal point, 0–4. Default 2. The example uses 0 (whole units). |
| `numberFormat` | Optional. `"eu"` (default) shows 1.234,56; `"en"` shows 1,234.56. |
| `cash` | `start` amount, a `history` list of `{label, amount}` points, and `endLabel`. |
| `running` | `income`: a list of `{id, label, amount}` income categories. `costs`: the running costs for the period. |
| `eventCosts` | `drivers`: a list of `{id, label, plan}`, where `plan` is a whole number. `categories`: a list of `{id, label, plan?}`; `plan` is an amount, allowed only for a category without lines. `lines`: a list of `{id, category, item, quantity, per?, unitCost}`, where `quantity` is a whole number and `per` is an optional list of driver ids. |
| `sponsorTiers` | Exactly two tiers, each `{label, baselinePrice}`. |
| `memberTiers` | Exactly three member contribution tiers, each `{label, baselineAmount}`. |
| `grant` | `{label}` for an unconfirmed grant. |
| `fixedContribution` | `{label, amount}` that is always included in member funding. |
| `bands` | Membership fee bands, each `{id, label, fee}`, where `fee` is the current fee per member. |
| `members` | Each `{id, name, band, info?}`. Used to count the members in each band. |
| `groups` | A title and optional help text for each control group: `sponsors`, `grant`, `associations`, `fees`, `running`, `event`. |
| `controls` | `{label, min, max, step}` for each slider (see the list after this table). |
| `defaults` | A complete set of values: every slider, plus `drivers` and `overrides`. |
| `presets` | Each `{id, name, values}`. `values` overrides `defaults`; `drivers`, `categoryChange` and each `overrides` map are merged by id. |
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
- `categoryChange`: optional `{categoryId: percent}`, a whole-number change
  of −100 or more. A category that is not listed is unchanged.
- `overrides`: New edition amounts that replace a Plan figure. A Change %
  on an income line, the running costs or a fee band is saved as the amount
  it gives.
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
- Planned cost of a category = sum of its line amounts with drivers at plan,
  or its `plan` amount when it has no lines.
- Category cost = its scenario amount if set; otherwise the sum of its line
  amounts with the scenario's driver counts (or its `plan`) ×
  (100 + priceChange) × (100 + categoryChange) ÷ 10,000.
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
whole minor unit at that point. These steps are each category's changed
cost, the contingency, each band's fees paid and an amount set by a Change %
step. Everything else is exact integer arithmetic.

The largest-funder share and every percentage shown are whole numbers,
rounded the same way. The share reads "n/a" when there is no cost or no
funder. Shares and the chart axis labels are for display only.