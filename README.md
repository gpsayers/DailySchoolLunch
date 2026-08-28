# DailySchoolLunch

Sends the daily lunch menu by SMS on weekdays using GitHub Actions, MealViewer, Gmail SMTP, and Verizon's email-to-text gateway.

## Setup

In the repository, open **Settings > Secrets and variables > Actions** and add:

- `SMTP_USERNAME`: Gmail address used to send the message
- `SMTP_PASSWORD`: Gmail App Password
- `SMTP_FROM`: same Gmail address
- `RECIPIENT_PHONE_NUMBER`: comma-separated 10-digit Verizon numbers

Example:

```text
7015551234, 7015555678
```

Optionally add the repository variable `DAILY_SMS_MESSAGE`. The workflow currently sends the MealViewer menu from Liberty Middle School at 8:00 AM UTC, Monday through Friday.

## Test

Open **Actions > Daily SMS**, choose **Run workflow**, and select **Run workflow** again. Check the job log for the extracted menu and delivery status.