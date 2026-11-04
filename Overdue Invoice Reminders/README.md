# Overdue Invoice Reminders

The inactive workflow contains exactly 25 nodes, including the sticky note, and 4 Code nodes. It uses one Google Sheet, Gmail, Telegram, and OpenRouter. DRY_RUN defaults true. No secrets are included.

## Setup

1. Import overdue-invoice-reminders.workflow.json into n8n. Leave it inactive until testing is complete.
2. Create the n8n credentials below and replace every matching credential placeholder. Select the existing OpenRouter account if available.

| Credential type | Suggested name | Access |
| --- | --- | --- |
| Google Sheets OAuth2 | Get Paid Lite googleSheetsOAuth2Api | Read and update the invoice sheet |
| Gmail OAuth2 | Get Paid Lite gmailOAuth2 | Approved sender account |
| Telegram API | Get Paid Lite telegramApi | Bot access to the owner chat |
| OpenRouter | OpenRouter account | openai/gpt-4o-mini |

3. Create a sheet named Invoices and import sample-invoices.csv. Required headers are invoice_no, customer_name, email, language, amount, amount_paid, currency, due_date, status, last_reminder_stage, last_reminder_date. Keep invoice_no unique and immutable. Format due_date as plain text YYYY-MM-DD. Use numeric amounts and last_reminder_stage. Use ar or en for language. Status is open, paid, or disputed. A partially paid invoice remains open with amount_paid populated.
4. Edit Config: DRY_RUN, BUSINESS_NAME, PAYMENT_DETAILS, OWNER_TELEGRAM_CHAT_ID, SHEET_ID, SHEET_NAME, SEND_WINDOW {start:9,end:20}, STAGE_DAYS {gentle:1,reminder:7,firm:14,escalate:30}, MAX_PER_RUN 30, and TIMEZONE Africa/Cairo. Replace every REPLACE_ value. Keep payment instructions short enough that the fallback body stays below 700 characters.
5. Keep Settings > Timezone at Africa/Cairo to match Config.TIMEZONE. Daily Run is 10:00 in the workflow timezone.
6. Save the workflow. Set Settings > Error workflow to this workflow. The Error Trigger runs through Config and routes to Error Alert. If the installed n8n UI disallows self-selection, a separate error workflow is required before activating error reporting. Error Trigger behavior must be tested using an active trigger; manual executions may not invoke it.
7. Check the Google Sheets Update Reminder column mapping in your installed UI. It matches invoice_no and writes only last_reminder_stage and last_reminder_date. Verify Telegram credentials, LLM parser output shape, and the installed node versions before activation.

## First DRY_RUN test

The sample dates use 2026-11-04 as today: stages 1, 2, 3, 4, paid, and a partial payment. Shift the dates together when testing later. Replace test emails with addresses you control before any live test.

Run Manual Run between 09:00 inclusive and 20:00 exclusive in Cairo with DRY_RUN true. Expect four customer drafts and one stage-4 owner alert on Telegram, followed by exactly one summary. Gmail and Update Reminder must not run. The partial-payment balance is 475 EGP. No sheet cells should change. Repeating a dry run repeats drafts because it stores no reminder state.

Force an LLM error to confirm the built-in Arabic or English fallback reaches the owner. Change an invoice to paid or change amount_paid before Re-read Invoice to confirm the draft is skipped. Add an invalid email/date to verify the invalid-row summary count. Outside the send window the workflow exits without alerts. With no eligible invoices and no invalid rows, it sends no summary.

For live testing, set MAX_PER_RUN to 1, use controlled recipients, and set DRY_RUN false. Verify Gmail success precedes the sheet update. A successful stage-4 owner alert updates its reminder fields without customer email. Then restore DRY_RUN true until acceptance is complete.

## Behavior and limits

Only the current stage is selected. last_reminder_stage greater than or equal to that stage prevents another attempt. Invalid open rows are counted once per row; paid and disputed rows are excluded without contributing to invalid counts. The cap counts invoices, including owner escalations. Deferred invoices are untouched.

Pick Invoices emits a no-invoice marker when necessary so invalid-only runs can still produce one summary. The marker never reaches customer or owner-draft nodes. The summary counts invalid rows once, even when several invoices carry the count.

Stages 1 to 3 use a Basic LLM Chain and structured parser. Validate checks invoice number, exact amount string, payment details, nonempty body, and length below 700. Provider/parser errors use one of six deterministic templates. An invalid fallback is marked failed without delivery.

Every stage, including stage 4, is re-read and fresh-checked before delivery. The fresh check requires the row to remain open and its rounded residual amount to match the selected invoice. Stage 4 skips draft validation and routes only to the owner in both dry-run and live modes. Google Sheets lookup assumes unique invoice numbers. It does not lock the row, so payment changes between the fresh check and Gmail cannot be eliminated. Concurrent runs are not protected by a database lock. Do not run concurrent live executions. If Gmail succeeds and the sheet update fails, the summary reports failure and the sheet can allow a later duplicate. Reconcile the Gmail result before retrying.

Gmail uses its error output to bypass sheet updates on send failure. Telegram and sheet-update errors continue into Record Outcome. No alert email fallback is present. The summary Telegram node runs once and reads the complete loop result directly. Record Outcome uses the standard n8n $prevNode.name binding to distinguish owner alerts from sheet updates; verify this behavior in the installed n8n runtime.

Quiet hours apply to invoice runs. Uncaught-error owner alerts bypass the quiet gate. The workflow does not contain an approval step. No source annotation or external log exists outside the sheet.

Local structural and Code-node checks do not establish successful n8n import, credential access, model responses, scheduling, or live delivery. Perform the UI tests before activation.
