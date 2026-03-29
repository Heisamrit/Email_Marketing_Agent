<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=200&section=header&text=📬%20AWS%20Mail%20Agent&fontSize=60&fontColor=ffffff&animation=fadeIn&fontAlignY=38&desc=Serverless%20Automated%20Email%20Marketing%20on%20AWS&descAlignY=60&descSize=18" width="100%"/>

<br/>

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![AWS Lambda](https://img.shields.io/badge/AWS_Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![Amazon S3](https://img.shields.io/badge/Amazon_S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![Amazon SES](https://img.shields.io/badge/Amazon_SES-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Amazon EventBridge](https://img.shields.io/badge/EventBridge-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![GoDaddy](https://img.shields.io/badge/GoDaddy-1BDBDB?style=for-the-badge&logo=godaddy&logoColor=white)

<br/>

![Status](https://img.shields.io/badge/Status-Live%20%26%20Running-brightgreen?style=flat-square)
![Serverless](https://img.shields.io/badge/Architecture-Serverless-blueviolet?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)
![Made with ❤️](https://img.shields.io/badge/Made%20with-❤️-red?style=flat-square)

<br/>
<br/>

> **A fully serverless, automated bulk email pipeline built on AWS.**  
> Reads contacts, personalizes HTML emails, and delivers them — all on autopilot, zero servers to manage.

<br/>

</div>

---

## 📌 Table of Contents

- [✨ Overview](#-overview)
- [🏗️ Architecture](#-architecture)
- [🧰 AWS Services Used](#-aws-services-used)
- [📁 Project Structure](#-project-structure)
- [📄 contacts.csv Format](#-contactscsv-format)
- [✉️ Email Template](#-email-template)
- [⚙️ Lambda Function](#-lambda-function)
- [🔐 IAM Policy](#-iam-policy)
- [🧪 Test Event](#-test-event)
- [🕐 EventBridge Scheduler](#-eventbridge-scheduler)
- [🌐 Domain & SES Setup](#-domain--ses-setup)
- [🚀 Deployment Guide](#-deployment-guide)
- [⚠️ Common Pitfalls](#-common-pitfalls)
- [📈 Future Improvements](#-future-improvements)
- [👤 Author](#-author)

---

## ✨ Overview

<div align="center">

| 📦 Zero Servers | ⚡ Fully Automated | 🎯 Personalized | 💸 Cost Effective |
|:-:|:-:|:-:|:-:|
| Lambda handles all compute — no EC2, no containers | EventBridge fires on schedule without human input | Each email is uniquely addressed per recipient | Pay only per Lambda invocation & SES email sent |

</div>

<br/>

This project was built to automate a **weekly newsletter** ("Tiny Tales") using **100% managed AWS services**. The system:

1. 📥 **Reads** a list of subscribers from a CSV file stored in S3
2. 🎨 **Fetches** a styled HTML email template from S3
3. 🧠 **Personalizes** the email by replacing `{{FirstName}}` for each contact
4. 📤 **Sends** the personalized email using Amazon SES
5. ⏰ **Repeats** automatically on a schedule via EventBridge

No manual work. No running servers. Just results in inboxes.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ⏰ Amazon EventBridge                                          │
│        (Cron / Rate Schedule)                                   │
│              │                                                  │
│              ▼                                                  │
│   ⚡ AWS Lambda Function                                         │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │                                                          │  │
│   │  Step 1 ──► Fetch contacts.csv  ────────────────────┐   │  │
│   │  Step 2 ──► Fetch email_template.html  ─────────┐   │   │  │
│   │                                                  │   │   │  │
│   │                          ┌───────────────────────┘   │   │  │
│   │                          │   📦 Amazon S3 Bucket      │   │  │
│   │                          │   mail-agent-heisamrit ◄──┘   │  │
│   │                                                          │  │
│   │  Step 3 ──► Personalize email per contact               │  │
│   │  Step 4 ──► Send via SES  ──────────────────────────┐  │  │
│   │                                                      │  │  │
│   └──────────────────────────────────────────────────────┘  │  │
│                                                              │  │
│              ┌───────────────────────────────────────────────┘  │
│              ▼                                                  │
│   📧 Amazon SES  ──────────────► 🌐 Recipient Inboxes          │
│   (noreply@sainicaraccessories.shop)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                             ▲
            🌐 GoDaddy DNS (SPF · DKIM · DMARC)
```

---

## 🧰 AWS Services Used

<div align="center">

| 🔧 Service | 🎯 Role | 💡 Why This Service |
|:---|:---|:---|
| ![S3](https://img.shields.io/badge/S3-569A31?style=flat-square&logo=amazons3&logoColor=white) **Amazon S3** | Stores `contacts.csv` & `email_template.html` | Durable, cheap, accessible from Lambda in milliseconds |
| ![SES](https://img.shields.io/badge/SES-232F3E?style=flat-square&logo=amazonaws&logoColor=white) **Amazon SES** | Sends personalized HTML emails | High deliverability, low cost ($0.10/1000 emails) |
| ![Lambda](https://img.shields.io/badge/Lambda-FF9900?style=flat-square&logo=awslambda&logoColor=white) **AWS Lambda** | Runs all business logic serverlessly | No idle cost — billed only when executing |
| ![EventBridge](https://img.shields.io/badge/EventBridge-FF4F8B?style=flat-square&logo=amazonaws&logoColor=white) **Amazon EventBridge** | Triggers Lambda on a cron schedule | Native AWS scheduler, no 3rd party cron needed |
| ![IAM](https://img.shields.io/badge/IAM-DD344C?style=flat-square&logo=amazonaws&logoColor=white) **AWS IAM** | Grants Lambda scoped permissions | Least-privilege security — only S3 read + SES send |
| ![GoDaddy](https://img.shields.io/badge/GoDaddy-1BDBDB?style=flat-square&logo=godaddy&logoColor=white) **GoDaddy DNS** | Manages domain & DNS records | SPF, DKIM, DMARC configured for email authentication |

</div>

---

## 📁 Project Structure

```
📦 mail-agent-heisamrit       ← S3 Bucket
 ┣ 📄 contacts.csv            ← Subscriber list (FirstName, Email)
 ┗ 📄 email_template.html     ← Branded HTML email with {{FirstName}} placeholder

📦 AWS Lambda
 ┗ 📄 lambda_function.py      ← Core orchestration logic (Python 3.x)

📦 AWS IAM
 ┗ 📄 LambdaMailPolicy        ← Custom policy: S3:GetObject + SES:SendEmail

📦 Amazon EventBridge
 ┗ 📅 MailScheduleRule        ← Triggers Lambda on your chosen cron schedule
```

---

## 📄 contacts.csv Format

The CSV file is the **source of truth** for all recipients. It must be uploaded to your S3 bucket.

```csv
FirstName,Email
Alice,alice@example.com
Bob,bob@example.com
Charlie,charlie@example.com
```

> **Column requirements:**
> - `FirstName` — Used to personalize each email
> - `Email` — Must be a valid address (verified in SES sandbox, or any address in production mode)

---

## ✉️ Email Template

The `email_template.html` file lives in your S3 bucket and supports the `{{FirstName}}` placeholder, which Lambda replaces dynamically before sending.

```html
<!DOCTYPE html>
<html>
  <body style="font-family: Georgia, serif; background: #fdf6ec; padding: 40px;">
    <div style="max-width: 600px; margin: auto; background: white; border-radius: 8px; padding: 32px;">

      <h1 style="color: #b5451b;">📖 Your Weekly Tiny Tales</h1>

      <p>Hi <strong>{{FirstName}}</strong>,</p>

      <p>
        Welcome to this week's edition — packed with bite-sized stories
        handpicked just for you.
      </p>

      <!-- ... rest of your email content ... -->

      <p style="color: #999; font-size: 12px;">
        You're receiving this because you subscribed at sainicaraccessories.shop
      </p>
    </div>
  </body>
</html>
```

> 💡 **Tip:** Keep your template fully self-contained with inline CSS — many email clients (Gmail, Outlook) strip external stylesheets.

---

## ⚙️ Lambda Function

**Runtime:** Python 3.x &nbsp;|&nbsp; **Handler:** `lambda_function.lambda_handler`

```python
import boto3
import csv

# Initialize AWS clients
s3_client  = boto3.client('s3')
ses_client = boto3.client('ses')

def lambda_handler(event, context):
    bucket_name = 'mail-agent-heisamrit'   # Your S3 bucket name

    try:
        # ── Step 1: Pull the contacts list from S3 ──────────────────────────
        csv_file = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
        lines    = csv_file['Body'].read().decode('utf-8').splitlines()

        # ── Step 2: Pull the HTML template from S3 ──────────────────────────
        email_template = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
        email_html     = email_template['Body'].read().decode('utf-8')

        # ── Step 3 & 4: Personalize & send for every contact ────────────────
        contacts = csv.DictReader(lines)
        for contact in contacts:
            personalized_email = email_html.replace('{{FirstName}}', contact['FirstName'])

            response = ses_client.send_email(
                Source='noreply@sainicaraccessories.shop',   # Your verified sender
                Destination={'ToAddresses': [contact['Email']]},
                Message={
                    'Subject': {
                        'Data'   : 'Your Weekly Tiny Tales Mail! 📖',
                        'Charset': 'UTF-8'
                    },
                    'Body': {
                        'Html': {
                            'Data'   : personalized_email,
                            'Charset': 'UTF-8'
                        }
                    }
                }
            )
            print(f"✅ Sent to {contact['Email']} | MessageId: {response['MessageId']}")

    except Exception as e:
        print(f"❌ Error: {e}")
        raise   # Re-raise so Lambda marks the invocation as failed
```

### 🔍 What each step does

| Step | Code | What Happens |
|:---:|:---|:---|
| 1 | `s3_client.get_object(... Key='contacts.csv')` | Downloads the subscriber list from S3 into memory |
| 2 | `s3_client.get_object(... Key='email_template.html')` | Downloads the HTML email body from S3 |
| 3 | `email_html.replace('{{FirstName}}', ...)` | Swaps the placeholder with each contact's real name |
| 4 | `ses_client.send_email(...)` | Fires off the personalized email via Amazon SES |

---

## 🔐 IAM Policy

The Lambda execution role has a **least-privilege custom policy** — it can only do what it absolutely needs to.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid"   : "AllowS3ReadForMailAssets",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::mail-agent-heisamrit/*"
        },
        {
            "Sid"   : "AllowSESSend",
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```

> ✅ **Security note:** The S3 permission is scoped only to the `mail-agent-heisamrit` bucket — Lambda cannot access any other bucket in your account.

---

## 🧪 Test Event

Use this in the Lambda console to **manually trigger** a run without waiting for the schedule:

```json
{
  "comment": "Generic test event for scheduled Lambda execution. The function does not use this event data.",
  "test": true
}
```

> The Lambda function doesn't read any fields from the event object — it fetches everything it needs from S3. This event simply triggers execution.

**How to test:**
1. Open your Lambda function in the AWS Console
2. Click **"Test"** → **"Create new test event"**
3. Paste the JSON above → Save → Click **"Test"**
4. Check the **Execution results** and **CloudWatch logs**

---

## 🕐 EventBridge Scheduler

Amazon EventBridge triggers the Lambda automatically using a cron expression. No manual intervention needed.

**Example schedules:**

| Schedule | Cron Expression | Description |
|:---|:---|:---|
| Every Monday at 9 AM UTC | `cron(0 9 ? * MON *)` | Weekly newsletter |
| Every day at 8 AM UTC | `cron(0 8 * * ? *)` | Daily digest |
| 1st of every month | `cron(0 9 1 * ? *)` | Monthly roundup |

**Setup path in AWS Console:**
```
Amazon EventBridge → Rules → Create Rule
  → Rule type: Schedule
  → Schedule pattern: Cron expression (paste from above)
  → Target: Lambda function → select your mail agent function
  → Create rule ✅
```

---

## 🌐 Domain & SES Setup

### DNS Records (configured in GoDaddy)

| Record Type | Name | Value | Purpose |
|:---:|:---|:---|:---|
| **TXT** | `@` | `v=spf1 include:amazonses.com ~all` | SPF — authorizes SES to send on your behalf |
| **CNAME** | `_domainkey` (SES provides this) | SES-generated value | DKIM — cryptographic email signing |
| **TXT** | `_dmarc` | `v=DMARC1; p=none; rua=mailto:you@domain.com` | DMARC — policy for handling spoofed emails |
| **MX** | `@` | `inbound-smtp.us-east-1.amazonaws.com` | (Optional) for receiving replies |

### Moving out of SES Sandbox

By default, SES is in **sandbox mode** — you can only send to verified addresses. To send to real subscribers:

1. Go to **AWS SES Console → Account Dashboard**
2. Click **"Request production access"**
3. Fill out the form explaining your use case
4. AWS typically approves within 24–48 hours ✅

---

## 🚀 Deployment Guide

```
Step 1 ─── Create S3 Bucket
            └── Upload contacts.csv
            └── Upload email_template.html

Step 2 ─── Verify Domain in SES
            └── Add DNS records in GoDaddy (SPF, DKIM, DMARC)
            └── Wait for SES verification (usually < 72 hours)
            └── Request production access if needed

Step 3 ─── Create Lambda Function
            └── Runtime: Python 3.x
            └── Paste lambda_function.py code
            └── Set timeout: 3–5 minutes (depending on contact list size)
            └── Memory: 128 MB is sufficient for most lists

Step 4 ─── Attach IAM Policy
            └── Go to Lambda → Configuration → Permissions
            └── Click the execution role → Attach policy
            └── Create and attach the custom S3 + SES policy

Step 5 ─── Test Manually
            └── Create test event in Lambda console
            └── Run it → verify emails arrive
            └── Check CloudWatch logs for any errors

Step 6 ─── Set Up EventBridge
            └── Create a new rule with your cron schedule
            └── Set target to your Lambda function
            └── Enable the rule → you're live! 🎉
```

---

## ⚠️ Common Pitfalls

<div align="center">

| ❌ Problem | ✅ Fix |
|:---|:---|
| `Email address not verified` error | Add recipient to SES verified emails (sandbox) or request production access |
| `Access Denied` on S3 | Check IAM policy — bucket name must match exactly |
| Lambda times out | Increase timeout in Configuration → General settings (max 15 min) |
| Emails land in spam | Set up SPF, DKIM, and DMARC records in GoDaddy DNS |
| CSV parse error | Ensure CSV uses UTF-8 encoding and has correct `FirstName,Email` headers |
| SES sending limit exceeded | Check your SES sending quota in the console — request an increase if needed |

</div>

---

## 📈 Future Improvements

- [ ] 📊 **Open & Click Tracking** — Integrate SES configuration sets for engagement analytics
- [ ] 🔕 **Unsubscribe Handling** — Add a one-click unsubscribe link and auto-remove from CSV
- [ ] 📎 **Attachment Support** — Use `send_raw_email` to include PDFs or images
- [ ] 📋 **DynamoDB Integration** — Replace CSV with a proper database for dynamic subscriber management
- [ ] 🔔 **SNS Alerts** — Get notified on Lambda failures via SNS + email/SMS
- [ ] 📬 **Bounce & Complaint Handling** — Use SES feedback notifications to keep your list clean
- [ ] 🖥️ **Admin Dashboard** — A simple web UI to manage contacts and preview templates before sending

---

## 💰 Cost Estimate

> Based on AWS free tier + pay-per-use pricing:

| Service | Usage | Est. Monthly Cost |
|:---|:---|:---:|
| AWS Lambda | < 1M invocations/month | **Free** (free tier) |
| Amazon S3 | < 5 GB storage | **~$0.01** |
| Amazon SES | 1,000 emails/month | **~$0.10** |
| EventBridge | 1 rule, weekly trigger | **Free** |
| **Total** | | **~$0.11/month** |

---

## 👤 Author

<div align="center">

**Amrit Saini**

Built as a serverless email automation project using AWS managed services.

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/)

</div>

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=100&section=footer" width="100%"/>

**⭐ If this project helped you, give it a star!**

*Built with ☁️ AWS + ❤️ Python*

</div>
