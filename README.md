<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f2027,50:203a43,100:2c5364&height=220&section=header&text=📬%20AWS%20Mail%20Agent&fontSize=65&fontColor=ffffff&animation=fadeIn&fontAlignY=40&desc=Serverless%20%7C%20Automated%20%7C%20Personalized%20Email%20Delivery&descAlignY=62&descSize=17&descColor=a8d8ea" width="100%"/>

<br/>

<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
<img src="https://img.shields.io/badge/AWS_Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white"/>
<img src="https://img.shields.io/badge/Amazon_S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white"/>
<img src="https://img.shields.io/badge/Amazon_SES-232F3E?style=for-the-badge&logo=amazonaws&logoColor=FF9900"/>
<img src="https://img.shields.io/badge/EventBridge-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white"/>
<img src="https://img.shields.io/badge/IAM-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white"/>
<img src="https://img.shields.io/badge/GoDaddy-1BDBDB?style=for-the-badge&logo=godaddy&logoColor=white"/>

<br/><br/>

<img src="https://img.shields.io/badge/🚀%20Status-Live%20%26%20Operational-00c853?style=flat-square&labelColor=1b5e20"/>
<img src="https://img.shields.io/badge/☁️%20Architecture-100%25%20Serverless-7c4dff?style=flat-square&labelColor=311b92"/>
<img src="https://img.shields.io/badge/💸%20Monthly%20Cost-~%240.11-ff6f00?style=flat-square&labelColor=e65100"/>
<img src="https://img.shields.io/badge/📧%20Powered%20by-Amazon%20SES-FF9900?style=flat-square&labelColor=232F3E"/>
<img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square"/>

<br/><br/>

<table>
<tr>
<td align="center"><b>📦 Zero Servers</b><br/>No EC2. No containers.<br/>Lambda does it all.</td>
<td align="center"><b>⚡ Fully Automated</b><br/>EventBridge fires<br/>on your schedule.</td>
<td align="center"><b>🎯 Personalized</b><br/>Every email addressed<br/>by first name.</td>
<td align="center"><b>💸 Near-Free</b><br/>Pay per use only.<br/>~$0.11/month.</td>
</tr>
</table>

<br/>

> 🔥 **A fully serverless, scheduled, personalized bulk email system — built entirely on AWS managed services.**  
> Upload your contacts. Write your template. Let AWS do the rest — forever.

</div>

---

## 📌 Table of Contents

| # | Section |
|:---:|:---|
| 1 | [✨ What This Project Does](#-what-this-project-does) |
| 2 | [🏗️ Full Architecture Diagram](#-full-architecture-diagram) |
| 3 | [🔵 Service Deep-Dive: Amazon S3](#-service-deep-dive-amazon-s3) |
| 4 | [🟠 Service Deep-Dive: AWS Lambda](#-service-deep-dive-aws-lambda) |
| 5 | [📧 Service Deep-Dive: Amazon SES](#-service-deep-dive-amazon-ses) |
| 6 | [🩷 Service Deep-Dive: Amazon EventBridge](#-service-deep-dive-amazon-eventbridge) |
| 7 | [🔴 Service Deep-Dive: AWS IAM](#-service-deep-dive-aws-iam) |
| 8 | [🌐 Service Deep-Dive: GoDaddy DNS](#-service-deep-dive-godaddy-dns) |
| 9 | [📁 Project File Structure](#-project-file-structure) |
| 10 | [📄 contacts.csv — Subscriber List](#-contactscsv--subscriber-list) |
| 11 | [✉️ email_template.html — Email Design](#-email_templatehtml--email-design) |
| 12 | [⚙️ Lambda Function — Full Code](#-lambda-function--full-code) |
| 13 | [🔐 IAM Policy — Permissions](#-iam-policy--permissions) |
| 14 | [🧪 Test Event — Manual Trigger](#-test-event--manual-trigger) |
| 15 | [🕐 EventBridge — Scheduling](#-eventbridge--scheduling) |
| 16 | [🚀 Step-by-Step Deployment Guide](#-step-by-step-deployment-guide) |
| 17 | [⚠️ Troubleshooting & Common Pitfalls](#-troubleshooting--common-pitfalls) |
| 18 | [💰 Cost Breakdown](#-cost-breakdown) |
| 19 | [📈 Roadmap & Future Improvements](#-roadmap--future-improvements) |
| 20 | [👤 Author](#-author) |

---

## ✨ What This Project Does

This project automates the end-to-end delivery of a **personalized weekly newsletter** ("Tiny Tales") using **zero-server AWS infrastructure**.

### 🔄 The Flow — Step by Step

```
 ┌──────────────────────────────────────────────────────────────────────┐
 │                                                                      │
 │  ① EventBridge fires a cron trigger on your chosen schedule         │
 │         │                                                            │
 │         ▼                                                            │
 │  ② Lambda wakes up — reads contacts.csv from S3                     │
 │         │                                                            │
 │         ▼                                                            │
 │  ③ Lambda reads email_template.html from S3                         │
 │         │                                                            │
 │         ▼                                                            │
 │  ④ For each contact → replaces {{FirstName}} with real name         │
 │         │                                                            │
 │         ▼                                                            │
 │  ⑤ SES delivers the personalized email to the recipient's inbox     │
 │         │                                                            │
 │         ▼                                                            │
 │  ⑥ Lambda logs results to CloudWatch → sleeps until next trigger    │
 │                                                                      │
 └──────────────────────────────────────────────────────────────────────┘
```

**No servers. No cron jobs. No manual clicks. Just emails arriving in inboxes — automatically.**

---

## 🏗️ Full Architecture Diagram

```
                     ┌─────────────────────────┐
                     │   ⏰ Amazon EventBridge  │
                     │   cron(0 9 ? * MON *)   │
                     └────────────┬────────────┘
                                  │  triggers
                                  ▼
                     ┌─────────────────────────┐
                     │   ⚡ AWS Lambda          │
                     │   (Python 3.x)          │
                     │                         │
                     │  1. get contacts.csv ───┼──────────────┐
                     │  2. get template.html ──┼──────────┐   │
                     │  3. personalize email   │          │   │
                     │  4. send via SES ───────┼──┐       ▼   ▼
                     └─────────────────────────┘  │   ┌──────────────────┐
                                                  │   │  📦 Amazon S3    │
                                                  │   │  Bucket:         │
                                                  │   │  mail-agent-     │
                                                  │   │  heisamrit       │
                                                  │   │                  │
                                                  │   │  • contacts.csv  │
                                                  │   │  • template.html │
                                                  │   └──────────────────┘
                                                  │
                                                  ▼
                                    ┌─────────────────────────┐
                                    │   📧 Amazon SES         │
                                    │   noreply@              │
                                    │   sainicaraccessories   │
                                    │   .shop                 │
                                    └────────────┬────────────┘
                                                 │
                              ┌──────────────────┼──────────────────┐
                              ▼                  ▼                  ▼
                       alice@mail.com     bob@mail.com     charlie@mail.com
                       📥 Inbox           📥 Inbox          📥 Inbox

                              ▲
              ┌───────────────────────────────┐
              │  🌐 GoDaddy DNS               │
              │  SPF  · DKIM  · DMARC        │
              │  (Email Authentication)       │
              └───────────────────────────────┘

              ┌───────────────────────────────┐
              │  🔐 AWS IAM                   │
              │  Grants Lambda permission to  │
              │  read S3 + send via SES only  │
              └───────────────────────────────┘

              ┌───────────────────────────────┐
              │  📊 AWS CloudWatch            │
              │  Logs every send result and   │
              │  captures any errors          │
              └───────────────────────────────┘
```

---

## 🔵 Service Deep-Dive: Amazon S3

<img src="https://img.shields.io/badge/Amazon_S3-Object_Storage-569A31?style=for-the-badge&logo=amazons3&logoColor=white"/>

**Amazon Simple Storage Service (S3)** is AWS's object storage — like a hard drive in the cloud, accessible from anywhere.

### Why S3 for this project?

- ✅ **Always available** — Lambda can fetch files from S3 in milliseconds, any time of day
- ✅ **Free tier friendly** — First 5 GB of storage is free
- ✅ **Easy updates** — Update `contacts.csv` or `email_template.html` anytime without touching code
- ✅ **Durable** — AWS stores your data across multiple Availability Zones (99.999999999% durability)

### What's stored in S3?

| File | Purpose | When It's Read |
|:---|:---|:---|
| `contacts.csv` | Subscriber list with `FirstName` and `Email` columns | Every Lambda execution |
| `email_template.html` | Styled HTML email body with `{{FirstName}}` placeholder | Every Lambda execution |

### How Lambda reads from S3

```python
# Lambda fetches the file using the bucket name and file key (path)
csv_file = s3_client.get_object(Bucket='mail-agent-heisamrit', Key='contacts.csv')
content  = csv_file['Body'].read().decode('utf-8')
```

> 💡 **Pro tip:** You can update your subscriber list or redesign the email template anytime by re-uploading the files to S3 — Lambda always reads the latest version on the next run.

---

## 🟠 Service Deep-Dive: AWS Lambda

<img src="https://img.shields.io/badge/AWS_Lambda-Serverless_Compute-FF9900?style=for-the-badge&logo=awslambda&logoColor=white"/>

**AWS Lambda** is a serverless compute service — you upload code, and AWS runs it for you. No servers to provision, patch, or manage. You pay only for the milliseconds your code actually runs.

### Why Lambda for this project?

- ✅ **No idle cost** — Lambda only runs when triggered. Between sends, you pay nothing
- ✅ **Scales automatically** — handles 1 contact or 10,000 contacts without configuration changes
- ✅ **Native AWS integration** — directly talks to S3 and SES using `boto3` (the AWS Python SDK)
- ✅ **Simple deployment** — paste code in the console, done

### Lambda Configuration Used

| Setting | Value | Reason |
|:---|:---|:---|
| Runtime | Python 3.x | `boto3` is built in — no external packages needed |
| Handler | `lambda_function.lambda_handler` | Entry point AWS calls |
| Timeout | 3–5 minutes | Enough time to loop over all contacts |
| Memory | 128 MB | Reading CSVs and sending HTTP requests is lightweight |
| Execution Role | Custom IAM Role | Grants permission to access S3 + SES |

### What Lambda does internally

```
Lambda Execution Flow:
━━━━━━━━━━━━━━━━━━━━
 [START]
    │
    ├── boto3.client('s3')   → creates S3 connection
    ├── boto3.client('ses')  → creates SES connection
    │
    ├── get_object → contacts.csv      → parse with csv.DictReader
    ├── get_object → email_template.html → load as string
    │
    ├── FOR each contact in CSV:
    │       ├── replace {{FirstName}} → contact['FirstName']
    │       └── ses.send_email(...)   → deliver to contact['Email']
    │
    └── [END] → CloudWatch logs all results
```

---

## 📧 Service Deep-Dive: Amazon SES

<img src="https://img.shields.io/badge/Amazon_SES-Email_Delivery-232F3E?style=for-the-badge&logo=amazonaws&logoColor=FF9900"/>

**Amazon Simple Email Service (SES)** is a cloud-based email sending service. It handles SMTP infrastructure, IP reputation, bounce management, and delivery at scale — so you don't have to.

### Why SES for this project?

- ✅ **High deliverability** — AWS maintains warm IP addresses with good sender reputation
- ✅ **Incredibly cheap** — $0.10 per 1,000 emails (free when sent from Lambda)
- ✅ **HTML email support** — send rich, styled HTML emails natively
- ✅ **Domain verification** — proves you own the sending domain to ISPs

### SES Modes

| Mode | Who Can Receive? | Sending Limit | How to Exit |
|:---|:---|:---|:---|
| **Sandbox** (default) | Only verified emails | 200/day, 1/sec | Request production access |
| **Production** | Anyone | Quota-based (request increase) | Approved by AWS |

### How SES sends the email

```python
ses_client.send_email(
    Source      = 'noreply@sainicaraccessories.shop',  # Verified sender
    Destination = {'ToAddresses': ['alice@example.com']},
    Message = {
        'Subject': {'Data': 'Your Weekly Tiny Tales Mail!', 'Charset': 'UTF-8'},
        'Body'   : {'Html': {'Data': '<html>...</html>',   'Charset': 'UTF-8'}}
    }
)
```

### SES Domain Verification

Before SES can send from your domain, AWS needs to confirm you own it. This is done by adding DNS records to GoDaddy (see DNS section below). Once verified, SES digitally signs all outgoing emails with DKIM — making them trusted by Gmail, Outlook, and other providers.

---

## 🩷 Service Deep-Dive: Amazon EventBridge

<img src="https://img.shields.io/badge/Amazon_EventBridge-Scheduler-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white"/>

**Amazon EventBridge** is a serverless event bus and scheduler. For this project, it acts as a **cron job in the cloud** — firing the Lambda function automatically on your chosen schedule.

### Why EventBridge?

- ✅ **No cron server needed** — replaces traditional Linux cron entirely
- ✅ **Highly reliable** — AWS manages all scheduling infrastructure
- ✅ **Free** — EventBridge scheduled rules are free for standard usage
- ✅ **Flexible scheduling** — supports both rate expressions and cron expressions

### Schedule Expressions

| Format | Example | Meaning |
|:---|:---|:---|
| Rate | `rate(7 days)` | Every 7 days from creation |
| Cron | `cron(0 9 ? * MON *)` | Every Monday at 9:00 AM UTC |
| Cron | `cron(0 8 * * ? *)` | Every day at 8:00 AM UTC |
| Cron | `cron(0 9 1 * ? *)` | 1st of every month at 9 AM UTC |

### How EventBridge connects to Lambda

```
EventBridge Rule
    └── Schedule: cron(0 9 ? * MON *)
    └── Target  : Lambda function ARN
    └── Input   : (optional JSON payload — unused in this project)
    └── Status  : ENABLED ✅
```

> When the rule fires, EventBridge passes an event payload to Lambda. Our Lambda ignores the payload entirely and reads everything fresh from S3.

---

## 🔴 Service Deep-Dive: AWS IAM

<img src="https://img.shields.io/badge/AWS_IAM-Access_Control-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white"/>

**AWS Identity and Access Management (IAM)** controls *who can do what* in your AWS account. Every Lambda function runs as an IAM **role** — and that role must explicitly be granted permission for each AWS action it performs.

### Why a custom IAM policy?

By default, a new Lambda role has **zero permissions**. Without the custom policy, Lambda would get `Access Denied` when trying to read from S3 or call SES. The policy we attached grants exactly — and only — what's needed.

### The Policy Explained

```json
{
    "Version": "2012-10-17",
    "Statement": [

        {
            "Sid"    : "AllowS3ReadForMailAssets",
            "Effect" : "Allow",
            "Action" : [ "s3:GetObject" ],
            "Resource": "arn:aws:s3:::mail-agent-heisamrit/*"
            // ↑ ONLY this specific bucket — Lambda can't touch any other S3 bucket
        },

        {
            "Sid"    : "AllowSESSendEmail",
            "Effect" : "Allow",
            "Action" : [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
            // ↑ Can send email from any verified SES identity in your account
        }

    ]
}
```

### Principle of Least Privilege

| Permission | Granted? | Why |
|:---|:---:|:---|
| `s3:GetObject` on target bucket | ✅ | Lambda needs to read CSV and HTML |
| `s3:PutObject` / `s3:DeleteObject` | ❌ | Lambda never writes to S3 — not needed |
| `ses:SendEmail` | ✅ | Core functionality |
| `ses:CreateReceiptRule` | ❌ | Lambda doesn't manage SES config |
| `iam:*` | ❌ | Lambda never touches IAM |

> 🔐 **Security best practice:** Only grant the minimum permissions required. This limits blast radius if credentials are ever compromised.

---

## 🌐 Service Deep-Dive: GoDaddy DNS

<img src="https://img.shields.io/badge/GoDaddy-Domain_%26_DNS-1BDBDB?style=for-the-badge&logo=godaddy&logoColor=white"/>

**GoDaddy** hosts the domain `sainicaraccessories.shop` and its DNS records. Email authentication records must be added here so that receiving mail servers (Gmail, Outlook, etc.) trust emails sent from SES.

### DNS Records Added

| Type | Host / Name | Value | Purpose |
|:---:|:---|:---|:---|
| `TXT` | `@` | `v=spf1 include:amazonses.com ~all` | **SPF** — tells the world only SES can send from your domain |
| `CNAME` | `[token]._domainkey` | `[token].dkim.amazonses.com` | **DKIM** — cryptographic signature on every email |
| `TXT` | `_dmarc` | `v=DMARC1; p=none; rua=mailto:admin@yourdomain.com` | **DMARC** — defines policy for failed authentication |

### What each record does

```
📬 An email arrives at Gmail from noreply@sainicaraccessories.shop
         │
         ├── Gmail checks SPF  → "Is SES allowed to send for this domain?" → ✅ YES
         ├── Gmail checks DKIM → "Is the signature valid?"                 → ✅ YES
         ├── Gmail checks DMARC→ "Do SPF + DKIM pass the domain policy?"  → ✅ YES
         │
         └── 📥 Email lands in INBOX (not spam!) ✅
```

> Without these records, your emails will likely land in spam or be rejected outright.

---

## 📁 Project File Structure

```
📦 GitHub Repository
 ┗ 📄 lambda_function.py         ← The entire serverless logic
 ┗ 📄 email_template.html        ← (sample) HTML email design
 ┗ 📄 contacts.csv               ← (sample) Subscriber list
 ┗ 📄 iam_policy.json            ← IAM policy document
 ┗ 📄 README.md                  ← This file

☁️ Amazon S3 Bucket: mail-agent-heisamrit
 ┣ 📄 contacts.csv               ← Live subscriber list (keep updated)
 ┗ 📄 email_template.html        ← Live email template (edit anytime)
```

---

## 📄 contacts.csv — Subscriber List

The CSV is the **subscriber database** for this system. Stored in S3 and read fresh on every Lambda execution.

### Format

```csv
FirstName,Email
Alice,alice@example.com
Bob,bob@example.com
Charlie,charlie@example.com
Diya,diya@example.com
```

### Rules

| Rule | Detail |
|:---|:---|
| **Required columns** | `FirstName` and `Email` — must match exactly (case-sensitive) |
| **Encoding** | UTF-8 — avoid special characters or Excel's default Windows encoding |
| **No BOM** | Save as UTF-8 without BOM to avoid parse errors |
| **No empty rows** | Trailing blank lines can cause errors |
| **Valid emails** | In SES sandbox, every email must be individually verified in SES |

> 💡 **To add/remove subscribers:** Just update the CSV and re-upload to S3. No code changes needed.

---

## ✉️ email_template.html — Email Design

The HTML file is the **visual design** of your email. Stored in S3 and fetched by Lambda on every run.

### The `{{FirstName}}` Placeholder

Lambda replaces `{{FirstName}}` with each contact's real name before sending:

```python
personalized_email = email_html.replace('{{FirstName}}', contact['FirstName'])
# "Hi {{FirstName}}," → "Hi Alice,"
```

### Sample Template Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Tiny Tales Weekly</title>
</head>
<body style="margin:0; padding:0; background-color:#f4f4f4; font-family: Georgia, serif;">

  <table width="100%" cellpadding="0" cellspacing="0">
    <tr>
      <td align="center" style="padding: 40px 0;">

        <!-- Email Card -->
        <table width="600" cellpadding="0" cellspacing="0"
               style="background:#ffffff; border-radius:12px;
                      box-shadow: 0 4px 20px rgba(0,0,0,0.1);">

          <!-- Header Banner -->
          <tr>
            <td style="background: linear-gradient(135deg, #1a1a2e, #16213e);
                       padding: 32px; border-radius:12px 12px 0 0; text-align:center;">
              <h1 style="color:#e94560; margin:0; font-size:28px;">📖 Tiny Tales</h1>
              <p style="color:#a8d8ea; margin:8px 0 0;">Your Weekly Story Digest</p>
            </td>
          </tr>

          <!-- Body -->
          <tr>
            <td style="padding: 32px;">
              <p style="font-size:16px; color:#333;">
                Hi <strong style="color:#e94560;">{{FirstName}}</strong>,
              </p>
              <p style="color:#555; line-height:1.7;">
                Welcome to this week's edition — packed with bite-sized stories
                and ideas handpicked just for you. Grab a coffee ☕ and enjoy!
              </p>

              <!-- Story content goes here -->

              <hr style="border:none; border-top:1px solid #eee; margin: 24px 0;"/>

              <p style="font-size:12px; color:#aaa; text-align:center;">
                You're receiving this because you subscribed at
                <a href="https://sainicaraccessories.shop" style="color:#e94560;">
                  sainicaraccessories.shop
                </a><br/>
                <a href="#" style="color:#aaa;">Unsubscribe</a>
              </p>
            </td>
          </tr>

        </table>
      </td>
    </tr>
  </table>

</body>
</html>
```

> ⚠️ **Important:** Use **inline CSS only** — Gmail, Outlook, and Apple Mail strip `<style>` tags and external stylesheets.

---

## ⚙️ Lambda Function — Full Code

```python
import boto3
import csv

# ── Initialize AWS SDK clients ───────────────────────────────────────────────
s3_client  = boto3.client('s3')   # S3 client for reading files
ses_client = boto3.client('ses')  # SES client for sending emails

def lambda_handler(event, context):
    """
    Main Lambda entry point.
    Triggered by EventBridge on a schedule.
    Reads contacts + template from S3, then sends personalized emails via SES.
    """

    bucket_name = 'mail-agent-heisamrit'  # ← Your S3 bucket name

    try:

        # ── STEP 1: Read contacts CSV from S3 ───────────────────────────────
        csv_file = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
        lines    = csv_file['Body'].read().decode('utf-8').splitlines()
        print(f"✅ Loaded contacts.csv from S3")

        # ── STEP 2: Read HTML email template from S3 ────────────────────────
        email_template = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
        email_html     = email_template['Body'].read().decode('utf-8')
        print(f"✅ Loaded email_template.html from S3")

        # ── STEP 3 & 4: Loop contacts → personalize → send ──────────────────
        contacts     = csv.DictReader(lines)
        sent_count   = 0
        failed_count = 0

        for contact in contacts:
            try:
                # Replace {{FirstName}} placeholder with actual name
                personalized_email = email_html.replace(
                    '{{FirstName}}', contact['FirstName']
                )

                # Send via SES
                response = ses_client.send_email(
                    Source      = 'noreply@sainicaraccessories.shop',
                    Destination = {'ToAddresses': [contact['Email']]},
                    Message = {
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

                sent_count += 1
                print(f"📨 Sent to {contact['Email']} | ID: {response['MessageId']}")

            except Exception as contact_error:
                failed_count += 1
                print(f"❌ Failed for {contact['Email']}: {contact_error}")
                continue  # Keep processing remaining contacts even if one fails

        print(f"\n📊 Done — Sent: {sent_count} | Failed: {failed_count}")

    except Exception as e:
        print(f"❌ Fatal error: {e}")
        raise  # Re-raise so Lambda marks the invocation as FAILED in CloudWatch
```

### Code Walkthrough

```
Line  1-2   → Import boto3 (AWS SDK) and csv (standard library)
Line  5-6   → Create reusable S3 and SES connections (initialized once outside handler)
Line  9     → lambda_handler(event, context) ← AWS calls this function on every trigger
Line 18     → Get contacts.csv from S3 and decode bytes to string
Line 22     → Get email_template.html from S3 and decode bytes to string
Line 27     → csv.DictReader parses the CSV into a list of {FirstName: ..., Email: ...} dicts
Line 31     → Loop over every subscriber row
Line 34     → Replace {{FirstName}} placeholder dynamically
Line 37     → ses_client.send_email() → fires the email
Line 53     → continue on failure → one bad email doesn't stop the rest
Line 57     → Print summary to CloudWatch
Line 61     → raise re-propagates fatal errors so CloudWatch alerts you
```

---

## 🔐 IAM Policy — Permissions

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid"     : "AllowS3ReadForMailAssets",
            "Effect"  : "Allow",
            "Action"  : [ "s3:GetObject" ],
            "Resource": "arn:aws:s3:::mail-agent-heisamrit/*"
        },
        {
            "Sid"     : "AllowSESSend",
            "Effect"  : "Allow",
            "Action"  : [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```

### How to attach this policy

```
AWS Console → IAM → Roles
  → Find your Lambda's execution role (shown in Lambda → Configuration → Permissions)
  → Click the role name
  → Add permissions → Create inline policy
  → Paste the JSON above
  → Name it: LambdaMailAgentPolicy
  → Save ✅
```

---

## 🧪 Test Event — Manual Trigger

Use this in the Lambda console to **manually fire** the function without waiting for the EventBridge schedule.

```json
{
  "comment": "Generic test event for scheduled Lambda execution. The function does not use this event data.",
  "test": true
}
```

> Our Lambda doesn't read the event payload at all — it fetches everything from S3. This JSON simply triggers the execution.

### How to run a test

```
Lambda Console
  → Your function → "Test" tab
  → "Create new test event"
  → Name it: ManualTestRun
  → Paste JSON above → Save
  → Click "Test" button
  → Watch "Execution result" panel
  → Click "View CloudWatch logs" to see all print() output
```

---

## 🕐 EventBridge — Scheduling

### Creating the Schedule

```
AWS Console → Amazon EventBridge → Rules → Create rule

  Step 1: Define rule detail
    ├── Name       : WeeklyTinyTalesTrigger
    ├── Description: Triggers the mail agent Lambda every Monday
    └── Rule type  : Schedule

  Step 2: Define schedule
    ├── Schedule type: Cron-based schedule
    └── Cron         : 0 9 ? * MON *
                       │ │ │   │
                       │ │ │   └── Day of week: MON (Monday)
                       │ │ └────── Month: every (*)
                       │ └──────── Hour: 9 (9 AM UTC)
                       └────────── Minute: 0

  Step 3: Select target
    └── Target: AWS Lambda function
    └── Function: [your Lambda name]

  Step 4: Review → Create rule ✅
```

### Useful Cron Expressions

| Description | Cron Expression |
|:---|:---|
| Every Monday at 9 AM UTC | `cron(0 9 ? * MON *)` |
| Every day at 8 AM UTC | `cron(0 8 * * ? *)` |
| Every Sunday at 10 AM UTC | `cron(0 10 ? * SUN *)` |
| 1st of every month at 9 AM | `cron(0 9 1 * ? *)` |
| Twice a week (Mon + Thu) | `cron(0 9 ? * MON,THU *)` |

> ⏰ All EventBridge times are in **UTC**. Convert to your local timezone (IST = UTC+5:30).

---

## 🚀 Step-by-Step Deployment Guide

### 🔵 Step 1 — Create the S3 Bucket

```
AWS Console → S3 → Create bucket
  ├── Bucket name : mail-agent-heisamrit
  ├── Region      : us-east-1 (same region as SES + Lambda)
  ├── Block all public access: ✅ ON (files are only accessed by Lambda)
  └── Create bucket

Then upload:
  → contacts.csv
  → email_template.html
```

---

### 📧 Step 2 — Set Up Amazon SES

```
AWS Console → Amazon SES → Verified Identities → Create Identity

  Option A — Verify your Domain:
    ├── Identity type: Domain
    ├── Domain: sainicaraccessories.shop
    └── AWS provides DNS records → add them to GoDaddy (see Step 3)

  Option B — Verify individual emails (sandbox only):
    ├── Identity type: Email address
    └── AWS sends a verification link → click it

  Request Production Access (to email unverified addresses):
    → SES Console → Account Dashboard → Request production access
    → Fill in: use case, expected volume, bounce handling plan
    → AWS approves in 24–48 hours ✅
```

---

### 🌐 Step 3 — Configure GoDaddy DNS

```
GoDaddy Dashboard → DNS → Add Records:

  1. SPF Record
     Type  : TXT
     Host  : @
     Value : v=spf1 include:amazonses.com ~all
     TTL   : 1 hour

  2. DKIM Records (AWS provides 3 CNAME records)
     Type  : CNAME
     Host  : [token]._domainkey
     Value : [token].dkim.amazonses.com
     (Add all 3 DKIM CNAMEs provided by SES)

  3. DMARC Record
     Type  : TXT
     Host  : _dmarc
     Value : v=DMARC1; p=none; rua=mailto:admin@sainicaraccessories.shop
```

---

### ⚡ Step 4 — Create the Lambda Function

```
AWS Console → Lambda → Create function
  ├── Function name : MailAgentHandler
  ├── Runtime       : Python 3.12
  ├── Architecture  : x86_64
  └── Create function

Configuration:
  ├── Timeout: 5 minutes (Configuration → General configuration)
  ├── Memory : 128 MB
  └── Paste lambda_function.py code → Deploy
```

---

### 🔐 Step 5 — Attach the IAM Policy

```
Lambda → Configuration → Permissions → Click the execution role name
  → IAM opens → Add permissions → Create inline policy
  → JSON tab → paste the policy JSON
  → Policy name: LambdaMailAgentPolicy
  → Create policy ✅
```

---

### 🧪 Step 6 — Run a Manual Test

```
Lambda → Test tab → Create new test event
  → Name: ManualTest
  → Paste: { "test": true }
  → Save → Click "Test"

  Expected output:
  ✅ Loaded contacts.csv from S3
  ✅ Loaded email_template.html from S3
  📨 Sent to alice@example.com | ID: 0102018...
  📨 Sent to bob@example.com   | ID: 0102019...
  📊 Done — Sent: 2 | Failed: 0
```

---

### ⏰ Step 7 — Create EventBridge Schedule

```
EventBridge → Rules → Create rule
  → Schedule: cron(0 9 ? * MON *)
  → Target: Lambda → MailAgentHandler
  → Create → Enable ✅

Your system is now fully live and automated! 🎉
```

---

## ⚠️ Troubleshooting & Common Pitfalls

| ❌ Error | 🔍 Cause | ✅ Fix |
|:---|:---|:---|
| `Email address is not verified` | SES sandbox mode — recipient not verified | Verify recipient in SES, or request production access |
| `AccessDenied` on S3 | IAM policy missing or bucket name wrong | Check policy ARN matches exact bucket name |
| `AccessDenied` on SES | `ses:SendEmail` not in IAM policy | Re-attach the IAM policy correctly |
| Lambda times out | Too many contacts, timeout too short | Increase Lambda timeout up to 15 minutes |
| Emails land in spam | Missing SPF / DKIM / DMARC | Add all 3 DNS record types in GoDaddy |
| `KeyError: 'FirstName'` | CSV column header is wrong | Ensure header is exactly `FirstName` (capital F, capital N) |
| `UnicodeDecodeError` | CSV saved in Windows encoding | Re-save CSV as UTF-8 without BOM |
| `NoRegionError` | Lambda region not matching SES region | Deploy Lambda in same region as your SES identities |

---

## 💰 Cost Breakdown

> All estimates based on AWS pricing as of 2024. Free tier applies to new accounts.

| Service | What We Use | Free Tier | Paid Cost |
|:---|:---|:---|:---|
| **AWS Lambda** | 1 invocation/week | 1M invocations/month free | $0.00 |
| **Amazon S3** | 2 tiny files, ~10 KB | 5 GB free | ~$0.00 |
| **Amazon SES** | ~500 emails/month | 62K emails/month free (from Lambda) | $0.00 |
| **EventBridge** | 1 scheduled rule | Free | $0.00 |
| **IAM** | 1 role, 1 policy | Always free | $0.00 |
| **CloudWatch Logs** | Lambda execution logs | 5 GB free | ~$0.00 |
| **GoDaddy Domain** | `sainicaraccessories.shop` | N/A (3rd party) | ~$1/month |
| | | **Total** | **~$1/month** |

> 💡 The entire AWS infrastructure runs **within the free tier** for most usage levels. The only real cost is your GoDaddy domain registration.

---

## 📈 Roadmap & Future Improvements

- [x] ✅ Basic personalized email sending
- [x] ✅ S3-based template and contact management
- [x] ✅ Automated scheduling via EventBridge
- [x] ✅ Secure least-privilege IAM configuration
- [ ] 📊 **Open & Click Tracking** — SES Configuration Sets + SNS for engagement metrics
- [ ] 🔕 **Unsubscribe Handling** — One-click unsubscribe link + auto-remove from CSV
- [ ] 📋 **DynamoDB Backend** — Replace CSV with a proper NoSQL subscriber database
- [ ] 🔔 **Failure Alerts** — SNS notification when Lambda fails or bounce rate spikes
- [ ] 📬 **Bounce & Complaint Handling** — SES feedback loop to auto-remove bad addresses
- [ ] 📎 **Attachment Support** — PDF/image attachments via `send_raw_email`
- [ ] 🖥️ **Admin Web UI** — Simple dashboard to manage subscribers and preview templates
- [ ] 🌍 **Multi-region Failover** — Deploy in us-east-1 + eu-west-1 for resilience

---

## 👤 Author

<div align="center">

<br/>

**Amrit Saini**

*Built as a real-world serverless project demonstrating AWS automation at zero server cost.*

<br/>

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/)
[![Portfolio](https://img.shields.io/badge/Portfolio-FF5722?style=for-the-badge&logo=google-chrome&logoColor=white)](#)

<br/>

</div>

---

<div align="center">

<br/>

*If this project helped you or inspired your own AWS journey — drop a ⭐ on the repo!*

<br/>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:2c5364,50:203a43,100:0f2027&height=120&section=footer&text=Built%20with%20☁️%20AWS%20%2B%20❤️%20Python&fontSize=20&fontColor=a8d8ea&animation=fadeIn&fontAlignY=65" width="100%"/>

</div>
