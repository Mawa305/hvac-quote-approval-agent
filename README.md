# HVAC Quote Approval Agent

An n8n workflow that drafts HVAC quotes with AI, checks them with a reviewer agent, and sends them for human approval before anything reaches the customer.

## Flow

1. **Webhook** receives the customer request (name, email, service, area) and validates the fields. Invalid requests get a 400 response.
2. **Respond Accepted** returns 202 right away, so the caller is not kept waiting for human approval.
3. **Drafter Agent** writes a short quote. It never invents prices, so the price stays "to be confirmed after inspection" unless a price list is given.
4. **Reviewer Agent** checks the draft against the request and returns JSON with `approved` and `notes`.
5. **Revision loop:** if the reviewer rejects the draft, it goes back to the drafter with the feedback. Maximum 3 attempts, then the owner is alerted.
6. **Human Approval** sends an email with Approve and Reject buttons to the owner.
7. **Approve:** the quote is emailed to the customer and logged as `approved`.
8. **Reject:** the owner gets a notice and the case is logged as `rejected`.
9. **Revision limit reached:** the owner gets a manual review email and the case is logged as `escalated`.

All outcomes are logged in a Google Sheet with timestamp, customer, area, service, outcome, revisions, and last reviewer note.

## Required Credentials

- OpenAI API
- Gmail (OAuth2)
- Google Sheets (OAuth2)

## Setup

1. In n8n, go to **Import from file** and select `hvac-quote-approval-agent.json`.
2. Re-select the credentials for OpenAI, Gmail, and Google Sheets.
3. Replace `OWNER_EMAIL` in the **Human Approval**, **Escalate Rejected Quote**, and **Escalate Revision Limit** nodes with your email.
4. Replace `YOUR_GOOGLE_SHEET_ID` in the three Google Sheets nodes with your sheet's ID.
5. Create a Google Sheet with a tab named `Quotes` and these headers in row 1:
   `timestamp | customer_name | area | service | outcome | revisions | last_note`
6. Publish the workflow, or run it with **Listen for test event** while testing.

## Test

Send a POST request to the webhook:

```json
{
  "customer_name": "Sara Ahmed",
  "customer_email": "test@example.com",
  "service": "Annual maintenance for 1.5-ton window AC and gas leak check",
  "area": "Gulberg III, Lahore"
}
```

Then check the approval email in your inbox and click Approve or Reject.

## Notes

- The workflow never sends a quote to a customer without human approval.
- Built as part of an n8n automation portfolio.
