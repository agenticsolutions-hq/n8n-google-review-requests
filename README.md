# Post-visit Google review requests for n8n

An n8n workflow for small service businesses (clinics, salons, studios, contractors) that log visits in Google Sheets. After a visit is marked completed, it emails the client a link to your Google review page. Each client is asked once, emails wait until quiet hours are over, and every email has a signed unsubscribe link.

The same workflow is published on the n8n template library: [n8n.io/workflows/21138](https://n8n.io/workflows/21138-send-post-visit-google-review-requests-from-google-sheets-via-gmail/?utm_source=github&utm_medium=template&utm_campaign=ex077&utm_content=repo_readme_f1). The JSON in this repo is the version published there.

## What it does

1. Runs every hour and reads the `Visits` tab. It picks rows with `status = completed` whose `visit_end` was at least `delay_hours` ago (default 3) and no more than `max_age_days` old (default 14), that haven't been asked yet and haven't opted out.
2. Holds emails during quiet hours in your business timezone and sends them the next morning.
3. Claims each row before sending. It writes `sending <execution ID>` into `review_status`, waits 15 seconds, reads the sheet again and only sends if the row still holds this run's claim. If any check fails, the whole batch stops before anything is sent and you get an alert email.
4. Emails the client a link to your Google review page, then marks the row `sent` with `review_requested_at` and reads that write back. Each client is asked once, across all their visits and on re-runs.
5. If the email server fails, the attempt is recorded on the row, the run is marked failed and you get an alert email. The row is retried on later runs, up to 3 recorded attempts.
6. The unsubscribe link is signed and doesn't contain the email address. It opens a confirm page, and the opt-out is saved only after the person confirms. It then sets `opt_out = TRUE` on every row for that email. Tampered links change nothing.

## Setup

1. Create a Google Sheet with a `Visits` tab and these columns: `visit_id, client_name, email, visit_end, timezone, status, review_requested_at, opt_out, review_status`.
2. Import `post-visit-google-review-request.json` into n8n and connect your Google Sheets and SMTP credentials on the nodes that ask for them.
3. In **Config**, set the sheet URL, timezone, business name, sender email, owner email (for alerts), review link, delay and quiet hours.
4. Copy the Production URL of the **Unsubscribe link** node into `optout_url` and type a random `optout_secret` of 16 or more characters. Nothing is sent until both are set.
5. Set the same timezone in the workflow settings.
6. While the workflow is inactive, test with a row that uses your own email, `status = completed` and a `visit_end` 4 hours ago. You should get exactly one email, and a second run should send nothing. Then activate it. Don't start manual runs while the schedule is active.

## Requirements

- Self-hosted n8n (tested on 2.43.7 and 2.44.6), core nodes only
- A Google account (Google Sheets)
- An SMTP mailbox
- No credentials are included in the JSON

## Testing and limits

This version passed all 29 of its automated tests on n8n 2.44.6 on Oct 26, 2026, with a mock standing in for Google Sheets and email. The same workflow logic passed them on n8n 2.43.7 on Oct 22, 2026. In those tests, three runs started at once sent exactly one email. Its email was also delivered through Gmail SMTP (587/STARTTLS) to a real inbox on Oct 22, 2026.

Google Sheets was tested against mock data only. A live Sheets test is still pending.

Google Sheets has no row locking, so the duplicate protection is best effort. Don't add, delete or sort rows in `Visits` while a run is in progress.

## If you need more than reviews

This workflow is one of five in our paid ops pack for n8n. The pack adds appointment reminders, lead intake, invoice follow-ups and a weekly KPI digest. It costs $39 one-time, with one license per business: [5 n8n workflows for service businesses](https://agenticasolutions.gumroad.com/l/ops-pack-n8n?utm_source=github&utm_medium=template&utm_campaign=ex077&utm_content=repo_readme_p1).

If late invoices are the bigger problem, there's also a single workflow for that. It checks a Google Sheet of your own invoices once a day, sends reminders at 3, 7 and 14 days overdue and stops when an invoice is marked paid. It costs $29: [automatic invoice reminders for n8n](https://agenticasolutions.gumroad.com/l/invoice-chaser-n8n?utm_source=github&utm_medium=template&utm_campaign=ex077&utm_content=repo_readme_p2).

Both use self-hosted n8n with Google Sheets and an SMTP mailbox.

---

n8n and Google are trademarks of their respective owners. Agentica is not affiliated with or endorsed by them.
