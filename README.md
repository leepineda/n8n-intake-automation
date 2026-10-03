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

1. In n8n, choose **Import from file** and select `intake-workflow.json`.
2. Connect your own credentials on the MongoDB, Google Sheets, and Gmail nodes.
   The export contains no credentials.
3. In the Google Sheets node, set your own spreadsheet URL (the file has a
   `YOUR_SHEET_ID_HERE` placeholder). The sheet needs the headers
   `Name`, `Email`, `Report`, `Timestamp`.
4. Send the example request above and check that a document, a sheet row, and
   an email appear.

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
