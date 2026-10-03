# n8n-intake-automation

A webhook-triggered [n8n](https://n8n.io) workflow that receives a report
submission, normalizes it into one consistent data shape, and then stores it,
logs it, and sends a confirmation email, all from a single request.

## What it does

```
POST /webhook/intake
        |
   Edit Fields   (extract name, email, report; add ISO timestamp)
        |
        +--> MongoDB        (insert document)
        +--> Google Sheets  (append row)
        +--> Gmail          (confirmation email to the sender)
```

All three downstream steps read from the same normalized fields, so the
extraction logic lives in one place instead of being repeated per destination.

## Example request

Fake data only:

```bash
curl -X POST "https://<your-n8n-host>/webhook-test/intake" \
  -H "Content-Type: application/json" \
  -d '{"name": "Juan Dela Cruz", "email": "juan@example.com", "report": "Sample report text"}'
```

Use `/webhook-test/intake` while the workflow is open in the editor, and
`/webhook/intake` once it is activated.

## Setup

1. **Import Workflow:** In n8n, click **Workflows** > **Import from file** and select `intake-workflow.json`.
2. **Configure Credentials:** Connect your credentials for the following nodes:
   - **MongoDB** (Specify your database name and ensure the collection is set to `n8n`).
   - **Google Sheets** (Authenticate your Google account).
   - **Gmail** (Authenticate your OAuth2/App Password for sending email).
3. **Set Up Google Sheet:**
   - Create a Google Sheet with column headers in row 1: `Name`, `Email`, `Report`, `Timestamp`.
   - Open the **Append row in sheet** node and replace `YOUR_SHEET_ID_HERE` with your actual Google Sheet ID (or paste your sheet URL).
   - Verify the tab name matches `Sheet1` (or select your active tab name).
4. **Test the Workflow:**
   - Click **Test step** or **Listen for Test Event** in n8n.
   - Run the example `curl` request in your terminal using `/webhook-test/intake`.
   - Verify that data appears in MongoDB, Google Sheets, and your inbox.
5. **Activate:** Toggle the workflow switch in n8n from **Inactive** to **Active** to start receiving production webhooks via `/webhook/intake`.

## Known limitations

This is a first version. Known gaps, in the order I plan to fix them:

- **The webhook is unauthenticated.** Anyone with the URL can post to it.
  Planned: header authentication on the Webhook node.
- **No input validation.** Empty fields and malformed emails are accepted.
  Planned: an IF node that validates required fields and email format and
  routes bad requests to a rejection response.
- **No failure handling.** If one destination fails, the others still run and
  nothing is reported. Planned: an error workflow.
- **The data is personal** (name and email). Do not commit real submissions.

## Planned

- Header authentication on the webhook
- Validation step before any data is stored
- An AI step that classifies and summarizes the report text, with the output
  constrained to a fixed set of categories and the input length-limited to
  reduce prompt-injection risk

## Author

Justine Lee G. Pineda, [github.com/leepineda](https://github.com/leepineda)
