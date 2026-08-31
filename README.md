# Daily School Lunch

This GitHub Actions workflow retrieves the daily lunch menu for Liberty Middle
School from MealViewer and sends it by SMS. It supports Gmail's email-to-SMS
gateway, TextBee (via G's Android phone gateway), and a test mode.

## Delivery Modes

Choose the mode with the `delivery_mode` workflow input:

| Value | Mode | Required secrets | Result |
| --- | --- | --- | --- |
| `1` | Email-to-SMS | `SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_FROM`, `RECIPIENT_PHONE_NUMBER` | Sends through Gmail SMTP to Verizon's `vtext.com` gateway. |
| `2` | TextBee | `TEXTBEE_API_KEY`, `RECIPIENT_PHONE_NUMBER` | Sends through a paired Android phone using the TextBee API. |
| `3` | No-op test | None | Scrapes and prints a menu preview without sending a message. This is the default. |

The workflow accepts comma-separated 10-digit phone numbers, for example:

```text
7015551234, 7015555678
```

For TextBee, the phone must be paired and enabled in the TextBee dashboard.
The workflow converts each number to international format before sending.

## GitHub Secrets

Add secrets under **Settings > Secrets and variables > Actions**.

### Email-to-SMS

- `SMTP_USERNAME`: Gmail address used to authenticate
- `SMTP_PASSWORD`: Gmail App Password, not the normal Gmail password
- `SMTP_FROM`: sender address, normally the same Gmail address
- `RECIPIENT_PHONE_NUMBER`: comma-separated 10-digit Verizon phone numbers

### TextBee

- `TEXTBEE_API_KEY`: API key created in the TextBee dashboard
- `RECIPIENT_PHONE_NUMBER`: comma-separated 10-digit phone numbers

The workflow never prints these secret values. Do not place an API key or
personal access token directly in the workflow file or the cron-job.org URL.

## Scheduling With cron-job.org

The workflow is manually triggered with GitHub's `workflow_dispatch` event.
cron-job.org can provide the weekday schedule by calling GitHub's workflow
dispatch API.

1. Create a GitHub token that can dispatch workflows in this repository. A
	fine-grained token should have access to this repository with **Actions:
	Read and write** permission. Store it securely; it is separate from the
	repository secrets above.
2. In cron-job.org, create a job with this URL:

	```text
	https://api.github.com/repos/gpsayers/DailySchoolLunch/actions/workflows/daily-sms.yml/dispatches
	```

3. Set the request method to `POST`.
4. Add these request headers:

	```text
	Accept: application/vnd.github+json
	Authorization: Bearer YOUR_GITHUB_TOKEN
	X-GitHub-Api-Version: 2022-11-28
	Content-Type: application/json
	```

5. Set the request body to select the desired mode. For TextBee, use:

	```json
	{
	  "ref": "main",
	  "inputs": {
		 "delivery_mode": "2"
	  }
	}
	```

	Use `"1"` for email-to-SMS or `"3"` for a no-op test.
6. Set the cron-job.org timezone and schedule. GitHub Actions and the workflow
	run in UTC, so convert the desired local delivery time to UTC and select
	Monday through Friday.
7. Enable the job and review both cron-job.org's HTTP result and the GitHub
	Actions run history after the first execution.

GitHub returns HTTP `204 No Content` when the workflow dispatch is accepted.
This only confirms that the workflow was queued; the Actions job log confirms
whether menu retrieval and SMS delivery succeeded.

## Manual Test

Open **Actions > Daily SMS**, choose **Run workflow**, select a delivery mode,
and run it. Mode `3` is the safest way to verify menu scraping without sending
an SMS.