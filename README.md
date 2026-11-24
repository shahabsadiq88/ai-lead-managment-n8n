# AI Lead Management Automation (n8n)

A prototype automation that receives leads from a website form, classifies them with an LLM, logs them to Google Sheets, alerts the team about high-priority leads, and sends every lead a personalized auto-reply.

Built as a hands-on learning project while studying AI automation, using a realistic agency scenario (a digital agency, NEXKODE, that gets service inquiries through a contact form).

> Status: working prototype, not hardened for production. See "Limitations" below.

## Screenshots

All screenshots are in the `screenshots/` folder:

- Full workflow canvas
- The contact form, filled in
- Google Sheet with the logged leads (priority and summary columns)
- High-priority alert email received by the admin
- Auto-reply email received by the lead
- n8n Executions tab showing successful runs

## What it does

1. A contact form sends the lead (name, email, company, service, message) to an n8n webhook.
2. An LLM reads the message and returns structured JSON: service category, priority (High / Medium / Low), and a one-line summary.
3. The lead and the AI result are saved as a new row in Google Sheets.
4. If the priority is **High**, an alert email goes to the admin.
5. A second LLM call writes a short, personalized auto-reply, which is emailed to the lead.
6. The webhook returns a success response to the form.

## Workflow overview

```
Webhook
  -> Edit Fields (flatten and clean lead data)
  -> Basic LLM Chain + Structured Output Parser (classify lead)
  -> Edit Fields "Combine Lead" (merge lead data + AI result)
  -> Google Sheets (Append Row)
  -> IF priority == High
        True  -> Send Email (admin alert) -> Reply Writer
        False -----------------------------> Reply Writer
  -> Reply Writer (LLM Chain)
  -> Send Email (auto-reply to lead)
  -> Respond to Webhook
```

## Tech stack

- n8n (self-hosted with Docker)
- Groq-hosted LLM (via the Groq Chat Model node)
- Google Sheets (storage)
- SMTP (emails)
- Plain HTML/JavaScript form

## Setup

1. Import `workflow.json` into n8n.
2. Create three credentials in n8n: **Groq**, **Google Sheets (OAuth2)**, and **SMTP**.
3. Create a Google Sheet with these headers in row 1:
   `name | email | company | service_selected | service_category | priority | summary | message | received_at`
4. In the workflow, point the Google Sheets node at your sheet and replace `YOUR_ADMIN_EMAIL` in the alert email node.
5. In the Webhook node, set **Respond** to `Using 'Respond to Webhook' Node`, and check the CORS settings if you call it from a browser.
6. Publish the workflow and copy the **Production URL**.
7. Open `form/index.html` and set `WEBHOOK_URL` to that URL.

## What I learned building this

- Webhook data arrives nested under `body`, so flattening it first keeps later nodes simple.
- Use temperature 0 for classification and a higher value (0.5) for writing emails.
- A structured output parser makes LLM output safe to use in later nodes.
- A string-vs-number mismatch can silently break an IF condition.
- Browser calls to a webhook need CORS handled.
- Network glitches happen, so nodes that call external services need **Retry On Fail**.
- Keep the admin alert address and the lead's address as separate settings.

## Limitations (what I would add before using this for a client)

- No spam protection yet (a honeypot field or similar check is needed for a public form).
- No duplicate-lead detection.
- Runs locally in Docker, so it needs proper hosting (n8n Cloud or a VPS) to run 24/7.
- Auto-replies go to whatever email is typed in the form, so email verification or rate limiting would be needed.
- No error-alert workflow if a step fails.

## Files

```
workflow.json
form/index.html
screenshots/
README.md
```
