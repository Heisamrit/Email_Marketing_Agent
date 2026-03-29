<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,14,24,30&height=230&section=header&text=📬%20AWS%20Mail%20Agent&fontSize=70&fontColor=ffffff&animation=fadeIn&fontAlignY=42&desc=Serverless%20·%20Automated%20·%20Personalized%20Email%20Delivery&descAlignY=65&descSize=16&descColor=c9f0ff" width="100%"/>

<br/>

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS%20Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![S3](https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![SES](https://img.shields.io/badge/Amazon%20SES-FF4F00?style=for-the-badge&logo=amazonaws&logoColor=white)
![EventBridge](https://img.shields.io/badge/EventBridge-E7157B?style=for-the-badge&logo=amazonaws&logoColor=white)
![IAM](https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)
![GoDaddy](https://img.shields.io/badge/GoDaddy-1BDBDB?style=for-the-badge&logo=godaddy&logoColor=white)

<br/>

![Status](https://img.shields.io/badge/🚀%20Status-Live%20%26%20Operational-00e676?style=flat-square&labelColor=1b5e20)
![Cost](https://img.shields.io/badge/💸%20Cost-~%240.11%2Fmonth-ffab00?style=flat-square&labelColor=e65100)
![Architecture](https://img.shields.io/badge/☁️%20Architecture-100%25%20Serverless-ce93d8?style=flat-square&labelColor=4a148c)
![License](https://img.shields.io/badge/License-MIT-29b6f6?style=flat-square&labelColor=01579b)

<br/><br/>

> 🔥 **Fully automated, serverless bulk email system — built entirely on AWS.**
> Upload contacts. Write a template. AWS handles the rest — every week, forever.

<br/>

</div>

---

## 🏗️ Architecture at a Glance

```
  ⏰ EventBridge                    📦 Amazon S3
  (Weekly Cron)  ──── triggers ──▶  ⚡ Lambda  ◀── contacts.csv
                                        │           email_template.html
                                        │
                                    personalizes ──▶ 📧 Amazon SES ──▶ 📥 Inboxes
                                        │
                                    🔐 IAM Role         🌐 GoDaddy DNS
                                    (permissions)       (SPF · DKIM · DMARC)
```

---

## 🧰 AWS Services Used

<br/>

### 🟠 AWS Lambda — *The Brain*

> Serverless compute that runs your Python code on demand. No servers, no idle cost — billed only for the milliseconds it runs.

- Fetches contacts & template from S3
- Personalizes each email with `{{FirstName}}`
- Sends emails through SES in a loop
- **Runtime:** Python 3.x | **Timeout:** 5 min | **Memory:** 128 MB

---

### 🟢 Amazon S3 — *The Storage*

> Object storage for your email assets. Think of it as a cloud drive that Lambda reads files from.

| File | What It Contains |
|:---|:---|
| `contacts.csv` | Subscriber list with `FirstName` and `Email` columns |
| `email_template.html` | Styled HTML email body with `{{FirstName}}` placeholder |

- Update your list or redesign your email anytime — just re-upload to S3, no code changes needed.

---

### 📧 Amazon SES — *The Postman*

> Managed email delivery service. Handles SMTP infrastructure, IP reputation, and delivery at scale.

| Mode | Who Can Receive | Limit |
|:---|:---|:---|
| **Sandbox** (default) | Verified emails only | 200/day |
| **Production** | Anyone | Request increase from AWS |

- Domain `sainicaraccessories.shop` is **verified** in SES
- Sends as `noreply@sainicaraccessories.shop`

---

### 🩷 Amazon EventBridge — *The Alarm Clock*

> A serverless scheduler that triggers Lambda on a cron schedule. No cron server needed.

```
cron(0 9 ? * MON *)  →  Every Monday at 9:00 AM UTC
cron(0 8 * * ? *)    →  Every day at 8:00 AM UTC
cron(0 9 1 * ? *)    →  1st of every month at 9 AM
```

---

### 🔴 AWS IAM — *The Security Guard*

> Controls exactly what Lambda is allowed to do — nothing more, nothing less.

```json
{
  "Statement": [
    {
      "Action"  : "s3:GetObject",
      "Resource": "arn:aws:s3:::mail-agent-heisamrit/*"  ✅ Read this bucket only
    },
    {
      "Action"  : ["ses:SendEmail", "ses:SendRawEmail"],
      "Resource": "*"                                     ✅ Send emails via SES
    }
  ]
}
```

---

### 🌐 GoDaddy DNS — *The Trust Layer*

> DNS records that prove to Gmail/Outlook that your emails are legitimate — not spam.

| Record | Purpose |
|:---:|:---|
| **SPF** `TXT` | Tells the world only SES can send from your domain |
| **DKIM** `CNAME` | Cryptographic signature on every outgoing email |
| **DMARC** `TXT` | Policy for how ISPs handle failed authentication |

---

## 📄 contacts.csv Format

```csv
FirstName,Email
Alice,alice@example.com
Bob,bob@example.com
```

> ⚠️ Column names must be exactly `FirstName` and `Email` (case-sensitive). Save as **UTF-8**.

---

## ⚙️ Lambda Function

```python
import boto3, csv

s3_client  = boto3.client('s3')
ses_client = boto3.client('ses')

def lambda_handler(event, context):
    bucket_name = 'mail-agent-heisamrit'
    try:
        # 1️⃣ Read contacts from S3
        csv_file  = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
        lines     = csv_file['Body'].read().decode('utf-8').splitlines()

        # 2️⃣ Read HTML template from S3
        template  = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
        email_html = template['Body'].read().decode('utf-8')

        # 3️⃣ Personalize & send for each contact
        for contact in csv.DictReader(lines):
            personalized = email_html.replace('{{FirstName}}', contact['FirstName'])
            response = ses_client.send_email(
                Source      = 'noreply@sainicaraccessories.shop',
                Destination = {'ToAddresses': [contact['Email']]},
                Message     = {
                    'Subject': {'Data': 'Your Weekly Tiny Tales Mail! 📖', 'Charset': 'UTF-8'},
                    'Body'   : {'Html': {'Data': personalized, 'Charset': 'UTF-8'}}
                }
            )
            print(f"✅ Sent to {contact['Email']} | ID: {response['MessageId']}")

    except Exception as e:
        print(f"❌ Error: {e}")
        raise
```

---

## 🧪 Test Event

```json
{
  "comment": "Generic test event — Lambda does not use this data.",
  "test": true
}
```

---

## 🚀 Deployment Steps

<br/>

### ![Step1](https://img.shields.io/badge/STEP%201-Create%20S3%20Bucket-569A31?style=for-the-badge&logo=amazons3&logoColor=white)

```
S3 → Create bucket → Name: mail-agent-heisamrit
Upload:  contacts.csv  and  email_template.html
```

---

### ![Step2](https://img.shields.io/badge/STEP%202-Set%20Up%20Amazon%20SES-FF4F00?style=for-the-badge&logo=amazonaws&logoColor=white)

```
SES → Verified Identities → Create Identity
  → Verify domain: sainicaraccessories.shop
  → AWS gives you DNS records → add them to GoDaddy
  → (Optional) Request production access to email anyone
```

---

### ![Step3](https://img.shields.io/badge/STEP%203-Configure%20GoDaddy%20DNS-1BDBDB?style=for-the-badge&logo=godaddy&logoColor=white)

```
GoDaddy → DNS → Add Records:
  TXT   @          v=spf1 include:amazonses.com ~all
  CNAME _domainkey [SES-provided DKIM values — add all 3]
  TXT   _dmarc     v=DMARC1; p=none; rua=mailto:you@yourdomain.com
```

---

### ![Step4](https://img.shields.io/badge/STEP%204-Create%20Lambda%20Function-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)

```
Lambda → Create function
  → Runtime: Python 3.12
  → Paste the code above → Deploy
  → Configuration → Timeout: 5 min | Memory: 128 MB
```

---

### ![Step5](https://img.shields.io/badge/STEP%205-Attach%20IAM%20Policy-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)

```
Lambda → Configuration → Permissions → Click execution role
  → Add permissions → Create inline policy
  → Paste the IAM JSON → Save as: LambdaMailAgentPolicy
```

---

### ![Step6](https://img.shields.io/badge/STEP%206-Test%20Manually-7B61FF?style=for-the-badge&logo=amazonaws&logoColor=white)

```
Lambda → Test tab → Create new event
  → Paste test JSON → Save → Click "Test"
  → Check Execution result + CloudWatch logs
```

---

### ![Step7](https://img.shields.io/badge/STEP%207-Add%20EventBridge%20Schedule-E7157B?style=for-the-badge&logo=amazonaws&logoColor=white)

```
EventBridge → Rules → Create rule
  → Type: Schedule → Cron: cron(0 9 ? * MON *)
  → Target: Lambda → your function name
  → Enable → Done! 🎉 You're fully live and automated.
```

---

## ⚠️ Common Pitfalls

| ❌ Problem | ✅ Fix |
|:---|:---|
| `Email not verified` error | Verify recipient in SES or get production access |
| `AccessDenied` on S3 | Check IAM policy — bucket name must match exactly |
| Emails land in spam | Add SPF, DKIM, DMARC records in GoDaddy |
| Lambda times out | Increase timeout to 5–15 min in Configuration |
| `KeyError: 'FirstName'` | CSV header must be exactly `FirstName` |
| `UnicodeDecodeError` | Re-save CSV as UTF-8 without BOM |

---

## 💰 Cost Estimate

| Service | Monthly Cost |
|:---|:---:|
| AWS Lambda | **Free** |
| Amazon S3 | **~$0.00** |
| Amazon SES | **~$0.10** |
| EventBridge | **Free** |
| GoDaddy Domain | **~$1.00** |
| **Total** | **~$1.11/month** |

---

## 📈 Future Improvements

- [ ] 📊 Open & click tracking via SES Configuration Sets
- [ ] 🔕 One-click unsubscribe + auto-remove from CSV
- [ ] 📋 Replace CSV with DynamoDB for dynamic subscriber management
- [ ] 🔔 SNS failure alerts when Lambda errors out
- [ ] 🖥️ Admin dashboard to manage contacts & preview templates

---

<div align="center">

## 👤 Author

**Amrit Saini**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/)

<br/>

*⭐ Star this repo if it helped you!*

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,14,24,30&height=120&section=footer&text=Built%20with%20☁️%20AWS%20%2B%20❤️%20Python&fontSize=18&fontColor=c9f0ff&animation=fadeIn&fontAlignY=70" width="100%"/>

</div>
