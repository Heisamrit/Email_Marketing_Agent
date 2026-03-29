<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,14,24,30&height=220&section=header&text=📬%20AWS%20Mail%20Agent&fontSize=70&fontColor=ffffff&animation=fadeIn&fontAlignY=42&desc=Serverless%20·%20Automated%20·%20Personalized%20Email%20Delivery&descAlignY=65&descSize=16&descColor=c9f0ff" width="100%"/>

<br/>

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS%20Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![S3](https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![SES](https://img.shields.io/badge/Amazon%20SES-FF4F00?style=for-the-badge&logo=amazonaws&logoColor=white)
![EventBridge](https://img.shields.io/badge/EventBridge-E7157B?style=for-the-badge&logo=amazonaws&logoColor=white)
![IAM](https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)
![GoDaddy](https://img.shields.io/badge/GoDaddy-1BDBDB?style=for-the-badge&logo=godaddy&logoColor=white)

<br/>

![Status](https://img.shields.io/badge/🚀%20Status-Live-00e676?style=flat-square&labelColor=1b5e20)
![Cost](https://img.shields.io/badge/💸%20Cost-~%241%2Fmonth-ffab00?style=flat-square&labelColor=e65100)
![Architecture](https://img.shields.io/badge/☁️-100%25%20Serverless-ce93d8?style=flat-square&labelColor=4a148c)

<br/><br/>

> 🔥 Reads contacts from S3, personalizes an HTML email template, and sends via Amazon SES — fully automated every week using EventBridge.

</div>

---

## 📋 Steps & Usage

<br/>

### ![s1](https://img.shields.io/badge/01-Amazon%20S3%20–%20Storage-569A31?style=for-the-badge&logo=amazons3&logoColor=white)

**What is S3?**
Amazon S3 (Simple Storage Service) is AWS's cloud object storage — like a drive in the cloud. It stores your files and makes them instantly accessible to other AWS services like Lambda. It's durable, cheap, and requires zero maintenance.

**How it's used here:**
Create a bucket named `mail-agent-heisamrit` and upload these two files:

```
📦 mail-agent-heisamrit/
 ┣ 📄 contacts.csv          →  Subscriber list (FirstName + Email)
 ┗ 📄 email_template.html   →  HTML email body with {{FirstName}} placeholder
```

**contacts.csv format:**
```csv
FirstName,Email
Alice,alice@example.com
Bob,bob@example.com
```

> 💡 To update your subscriber list or redesign the email — just re-upload the file to S3. No code changes needed.

---

### ![s2](https://img.shields.io/badge/02-Amazon%20SES%20+%20GoDaddy%20–%20Email%20%26%20Domain-FF4F00?style=for-the-badge&logo=amazonaws&logoColor=white)

**What is SES?**
Amazon SES (Simple Email Service) is AWS's managed email delivery platform. It handles SMTP servers, IP reputation, bounce management, and delivers your emails reliably at scale — so you don't run your own mail server.

**What is GoDaddy used for here?**
GoDaddy hosts the domain `sainicaraccessories.shop`. DNS records must be added there so Gmail, Outlook, and other providers trust emails coming from SES. Without these, emails go to spam.

**Setup:**
Go to **SES → Verified Identities → Create Identity → Domain** → verify `sainicaraccessories.shop`. AWS provides DNS records — add them in GoDaddy:

| DNS Record | Type | Purpose |
|:---:|:---:|:---|
| SPF | `TXT` | Tells the world SES is allowed to send from your domain |
| DKIM | `CNAME` | Cryptographically signs every outgoing email |
| DMARC | `TXT` | Policy for ISPs when authentication fails |

> 📌 SES starts in **Sandbox mode** — only verified emails can receive. Go to **SES → Account Dashboard → Request production access** to send to anyone.

---

### ![s3](https://img.shields.io/badge/03-AWS%20Lambda%20–%20Serverless%20Logic-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)

**What is Lambda?**
AWS Lambda is serverless compute — you write code and AWS runs it on demand. There are no servers to manage, no idle costs. Lambda wakes up when triggered, runs your code, and shuts down. You only pay for execution time (milliseconds).

**How it's used here:**
Lambda is the brain of the system. It reads both files from S3, personalizes the HTML template for each contact by replacing `{{FirstName}}`, then fires the email via SES — all in one run.

**Setup:** Lambda → Create function → **Python 3.12** → Paste code → Deploy
Set **Timeout: 5 min** | **Memory: 128 MB** under Configuration → General.

```python
import boto3, csv

s3_client  = boto3.client('s3')
ses_client = boto3.client('ses')

def lambda_handler(event, context):
    bucket_name = 'mail-agent-heisamrit'
    try:
        # 1️⃣ Read contacts from S3
        csv_file   = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
        lines      = csv_file['Body'].read().decode('utf-8').splitlines()

        # 2️⃣ Read HTML template from S3
        template   = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
        email_html = template['Body'].read().decode('utf-8')

        # 3️⃣ Personalize & send for each contact
        for contact in csv.DictReader(lines):
            personalized = email_html.replace('{{FirstName}}', contact['FirstName'])
            ses_client.send_email(
                Source      = 'noreply@sainicaraccessories.shop',
                Destination = {'ToAddresses': [contact['Email']]},
                Message     = {
                    'Subject': {'Data': 'Your Weekly Tiny Tales Mail! 📖', 'Charset': 'UTF-8'},
                    'Body'   : {'Html': {'Data': personalized, 'Charset': 'UTF-8'}}
                }
            )
            print(f"✅ Sent to {contact['Email']}")

    except Exception as e:
        print(f"❌ Error: {e}")
        raise
```

---

### ![s4](https://img.shields.io/badge/04-AWS%20IAM%20–%20Permissions-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)

**What is IAM?**
AWS IAM (Identity and Access Management) controls who can do what inside your AWS account. Every Lambda function runs as an IAM **role** — and by default it has zero permissions. You must explicitly grant it access to each service it needs to use.

**How it's used here:**
A custom inline policy is attached to the Lambda execution role, giving it the minimum permissions required — read from the S3 bucket and send emails via SES. Nothing more.

**Setup:** Lambda → Configuration → Permissions → Click execution role → Add inline policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect"  : "Allow",
      "Action"  : ["s3:GetObject"],
      "Resource": "arn:aws:s3:::mail-agent-heisamrit/*"
    },
    {
      "Effect"  : "Allow",
      "Action"  : ["ses:SendEmail", "ses:SendRawEmail"],
      "Resource": "*"
    }
  ]
}
```

> 🔐 The S3 permission is scoped to only this bucket — Lambda cannot access any other S3 bucket in your account.

---

### ![s5](https://img.shields.io/badge/05-Test%20Manually-7B61FF?style=for-the-badge&logo=amazonaws&logoColor=white)

**Why test manually?**
Before letting EventBridge fire the function automatically every week, run it once manually to confirm emails land correctly and CloudWatch logs show no errors.

**Setup:** Lambda → Test tab → Create new test event → Paste below → Click **Test**

```json
{
  "comment": "Generic test event. Lambda does not use this data.",
  "test": true
}
```

> Check **Execution result** and **View CloudWatch logs** to verify every contact received their email.

---

### ![s6](https://img.shields.io/badge/06-Amazon%20EventBridge%20–%20Scheduler-E7157B?style=for-the-badge&logo=amazonaws&logoColor=white)

**What is EventBridge?**
Amazon EventBridge is a serverless event bus and scheduler. It acts as a cloud-native cron job — triggering your Lambda on any schedule you define, with no server to maintain and no extra cost.

**How it's used here:**
A rule is created that fires the Lambda function automatically every week (or on whatever cadence you set). Lambda then runs the full email pipeline without any human input.

**Setup:** EventBridge → Rules → Create rule → **Schedule** → Cron expression → Target: your Lambda

```
cron(0 9 ? * MON *)   →  Every Monday at 9 AM UTC
cron(0 8 * * ? *)     →  Every day at 8 AM UTC
cron(0 9 1 * ? *)     →  1st of every month at 9 AM
```

Enable the rule → **Create** ✅ — your system is now fully live and automated. 🎉

---

## ⚠️ Common Pitfalls

| ❌ Problem | ✅ Fix |
|:---|:---|
| `Email not verified` | Verify recipient in SES or request production access |
| `AccessDenied` on S3 | Bucket name in IAM policy must match exactly |
| Emails go to spam | Add SPF, DKIM, DMARC records in GoDaddy |
| Lambda times out | Increase timeout to 5–15 min in Configuration |
| `KeyError: 'FirstName'` | CSV header must be exactly `FirstName` (case-sensitive) |

---

## 🏗️ Architecture

```
  ⏰ EventBridge (cron schedule)
         │
         ▼
  ⚡ AWS Lambda (Python)
         │
         ├──▶ 📦 S3: reads contacts.csv
         ├──▶ 📦 S3: reads email_template.html
         │
         │    for each contact:
         │      replaces {{FirstName}} → sends email
         │
         └──▶ 📧 Amazon SES ──▶ 📥 Recipient Inboxes

  🔐 IAM Role  →  grants Lambda permission to read S3 + send via SES
  🌐 GoDaddy   →  SPF · DKIM · DMARC for email authentication
```

---

<div align="center">

**Amrit Saini**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/)

*⭐ Star this repo if it helped you!*

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,14,24,30&height=120&section=footer&text=Built%20with%20☁️%20AWS%20%2B%20❤️%20Python&fontSize=18&fontColor=c9f0ff&fontAlignY=70" width="100%"/>

</div>
