# The Stripe Payout Reconciliation Hell

> *"I started a SaaS company to build software. I did not start a SaaS company to become a forensic accountant investigating my own payment processor."*
> — Every bootstrapped founder, 11:47 PM on the last day of the month

## 💀 The Nightmare: The Bank Says $8,432. Stripe Says "Good Luck."

It's the last day of the month. The most stressful day of my entire calendar. Not because of sprint deadlines. Not because of investor updates. Because of reconciliation.

My business bank account shows an incoming wire: **$8,432.17** from Stripe. Clean. Simple. One number. I can see it. My bank can see it. The universe agrees on this number.

So I open Stripe's Dashboard. I navigate to Payouts. I click "Export." I select "All payout events." I download the CSV. I open it.

And I am immediately confronted with a spreadsheet that has more columns than I have brain cells.

I'm not exaggerating. The Stripe Payout export — the default, out-of-the-box, "here's your financial data" export — contains over 50 columns. Let me list a few of the gems you'll find scrolling horizontally through this monument to data over-engineering:

| Column Name | What It Contains | Do I Need It? |
| :--- | :--- | :--- |
| `id` | `po_1Nabc2DEF3ghi...` — a Stripe-internal object ID | No |
| `object` | The literal string "payout" | No |
| `arrival_date` | Unix timestamp. (Requires a converter) | Maybe |
| `automatic` | TRUE or FALSE | No |
| `balance_transaction`| Another internal ID | No |
| `created` | Another Unix timestamp | No |
| `currency` | `usd` — yes, I operate in USD, thank you | Maybe |
| `metadata` | Always blank unless you've instrumented your codebase | No |
| `reconciliation_status`| Blank or some internal flag | No |

I need, at most, 7 columns. I am being handed 50+. The signal-to-noise ratio of this CSV is approximately 14%.

**And here's the real kicker: I still can't find the $8,432.17 anywhere in this file.**

I sum the `amount` column. It gives me $8,619.42. I sum the `fee` column. Subtract it. I get $8,501.08. Still not $8,432.17. I try filtering by date range. Nothing matches. The number that my bank is showing me is nowhere to be found in Stripe's export without some combination of filtering, grouping, and arithmetic that is nowhere documented.

## 🕳️ Hidden Trap #1: The Gross / Fee / Net Triangle of Lies

In a sane universe, financial reconciliation follows a simple equation:
**Gross − Fee = Net**

In Stripe's payout export, this equation is routinely violated.

### Scenario A: Refunds
A customer charges $100. Stripe takes a $3.10 fee. You receive $96.90. Two weeks later, you refund the customer $40. Stripe refunds you $1.24 of the fee (proportional). The refund appears as a separate line item:

| Description | Gross | Fee | Net |
| :--- | :--- | :--- | :--- |
| Payment from customer | 100.00 | -3.10 | 96.90 |
| Refund to customer | -40.00 | 1.24 | -38.76 |

Now, if you sum the Gross column: 60.00. If you sum the Fee column: -1.86. And 60.00 - 1.86 = 58.14. Great, the math works!

**But wait** — the refund row's Fee column shows a *positive* 1.24. Is that a fee *charged* to you or a fee *refunded* to you? The sign convention is inconsistent between charge rows and refund rows. If you're not paying extremely close attention, you will double-count or miss fees entirely.

### Scenario B: Cross-Border Transactions and FX
You're a US-based company. A customer in Germany pays you €89.00. Stripe converts it to USD at their exchange rate (which includes a 1% FX fee that is not always clearly itemized). The payout line shows:

| Description | Gross | Fee | Net | Currency |
| :--- | :--- | :--- | :--- | :--- |
| Payment from DE customer | 97.23 | -4.17 | 93.06 | usd |

But the original charge was €89.00. Where did 97.23 come from? Where is the 1% FX fee? It's embedded in the conversion rate. Your Gross is already net of the FX spread. Your accountant asks: "What was the actual revenue?" You can't answer from this CSV alone. 

## 🕳️ Hidden Trap #2: Metadata Columns — 40 Columns of Nothing

Every Stripe object supports a `metadata` field. In the payout export, Stripe expands metadata into individual columns for *every possible key* that has ever been set across your entire account.

The result? If your engineering team has ever set `metadata["utm_source"]` or `metadata["plan_id"]`, every single one of those becomes a column in your CSV. And for 99% of rows, these columns are completely empty.

I've seen Stripe payout exports with 40+ columns of metadata. These columns:
*   Serve no reconciliation purpose — your accountant doesn't care about `utm_source`.
*   Break CSV parsers that aren't robust to large numbers of empty trailing columns.
*   Inflate file size — I've seen 10MB payout CSVs that are 90% empty metadata cells.

## 🕳️ Hidden Trap #3: Transfers and Transactions in the Same Table

This is the trap that makes you question your sanity.

Stripe's payout export includes both actual financial transactions AND internal transfers in the same flat CSV, with no clear visual distinction between them. You'll see rows with `type` values like `transfer`, `payout`, `payment`, `refund`, `adjustment`.

If you naively `SUM()` the Gross column, you're summing revenue events + internal transfers + adjustments + refunds, and the result is a number that doesn't correspond to any financial reality.

## 🔧 The Solution: Extract 7 Columns. Ignore the Rest.

After months of suffering, I've arrived at a simple truth: Stripe reconciliation requires exactly 7 fields. Not 50. Not 30. Seven.

### The Sacred Seven

1.  **Date**: `arrival_date` (converted from Unix timestamp to YYYY-MM-DD). Cash-basis accounting lives and dies on this field.
2.  **Description**: `description` + `type`. You need a human-readable explanation.
3.  **Gross**: `amount` (before fees). This is your top-line revenue.
4.  **Fee**: `fee` (as a negative number). This is your cost of revenue.
5.  **Net**: `net`. `SUM(Net)` across a payout must equal the wire amount in your bank statement.
6.  **Currency**: `currency`.
7.  **Customer**: `customer_email`.

### The Filtering Rules

1.  **Filter 1: Remove Transfer Rows**. Any row where `type` equals `transfer`, `payout`, or `network_cost` should be removed.
2.  **Filter 2: Normalize Sign Conventions**. Normalize everything to a consistent convention (Revenue: Gross positive, Fee negative. Refunds: Gross negative, Fee positive).
3.  **Filter 3: Convert Timestamps to Dates**. Every Unix timestamp must be converted to `YYYY-MM-DD` in your local timezone.

After extraction and filtering, your reconciliation table should look like this:

| Date | Description | Gross | Fee | Net | Currency | Customer |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 2026-01-31 | Charge for Pro Plan | 99.00 | -3.17 | 95.83 | USD | alice@acme.com |
| 2026-01-31 | Refund for Basic Plan | -29.00 | +0.93 | -28.07 | USD | carol@agency.com |

Every row is self-contained. `SUM(Net)` equals the bank deposit. Period. This is the table you import into QuickBooks. This is the table that lets you close your books in 15 minutes instead of 5 hours.

## Final Thought

Stripe is an extraordinary product. Their developer experience is why they've won the payments market. But their financial reporting exports were clearly designed by engineers for engineers. The payout export is a data dump, not a financial report. 

You shouldn't need to understand Stripe's internal object model to know how much money you made this month.

---

## 🛠️ The Local Solution (Instant QuickBooks Ready Ledger)

Stop playing hide-and-seek with your Stripe fees in Excel.

We built **[dataprep.dev](https://dataprep.dev)** — a browser-based toolkit that extracts the exact financial ledger from Stripe's chaotic exports.
* Strips away 40+ useless metadata columns automatically.
* Extracts only the core 7 financial fields (Gross, Fee, Net, Date, Description).
* Generates a clean CSV ready for QuickBooks or Xero import in 1 second.

*Zero uploads. 100% Privacy. Your financial data never leaves your device.*

👉 **[Try the Stripe Payout Formatter Free](https://dataprep.dev/tools/stripe-formatter)**
