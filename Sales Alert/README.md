# Sales Alert (n8n)

Checks Technology sales in a Google Sheet every day at 18:00 and sends a Telegram alert when the total is below 50,000.

## Flow

1. **Schedule Trigger**: cron `0 18 * * *`.
2. **Google Sheets**: reads all rows of the sales sheet (columns used: `Category`, `Sales`).
3. **Code**: sums `Sales` for rows where `Category` is `Technology`.
4. **If**: `totalTechSales < 50000`.
5. **Telegram**: sends the alert on the **true** branch (below target).

## Behavior to know

- The code sums every Technology row in the sheet. It does not filter by date, so the sheet should only contain the period you want to evaluate (for example today's rows), or the code needs a date filter.
- `Sales` is parsed with `parseFloat`, so malformed values can give wrong totals. Keep the sheet clean.
- The threshold is hardcoded in the If node. Change it there.

## Setup

- Import `Sales Alert.json` and set credentials for Google Sheets and Telegram.
- Point the Sheets node at your own spreadsheet and set your Telegram chat ID.
- Run it manually once with test rows below and above the threshold. Only the below-target case should send a message.

## Ideas for later

- Filter rows by date and set the workflow timezone explicitly.
- Validate amounts and stop on invalid data.
- Multiple categories and thresholds, email or Slack as extra channels.
