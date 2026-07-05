# Phase 8 — Lambda + S3 Event Automation

## Goal

We’ll build an event-driven workflow:

**File uploaded to S3 → S3 triggers Lambda → Lambda logs file details to CloudWatch**

This is a clean, practical implementation of **serverless processing** in your EduStream platform.

---

# What this phase adds to your architecture

When an instructor uploads a file (for example a PDF or course note) into the S3 bucket:

* S3 generates an **ObjectCreated** event
* **Lambda** is triggered automatically
* Lambda captures details like:

  * bucket name
  * file key / path
  * file size (if available from event context or via head_object)
  * upload timestamp / event time
* Lambda writes the event into **CloudWatch Logs**

---

# Final Phase 8 architecture flow

## Upload flow

1. Instructor uploads file through EduStream app
2. App stores file in **S3**
3. **S3 event notification** triggers **Lambda**
4. **Lambda** logs upload metadata to **CloudWatch Logs**
5. You can demonstrate event-driven automation in your project

---

# What we’ll create in this phase

1. **IAM role for Lambda**
2. **Lambda function** → `EduStream-S3-Processor`
3. **S3 event notification** on your bucket
4. **Test upload** through the app or directly in S3
5. Verify logs in **CloudWatch**

---

# Before you start

We’ll use your existing bucket:

* `edustream-course-storage-farhan-2026`

---

# Step 1 — Create IAM Role for Lambda

Lambda needs permission to:

* write logs to CloudWatch
* read object metadata from S3

---

## Go to:

**IAM → Roles → Create role**

### Trusted entity

Choose:

* **AWS service**
* **Lambda**

Click **Next**

---

## Attach permissions

Attach these policies:

### Required

* `AWSLambdaBasicExecutionRole`

This allows Lambda to write to CloudWatch Logs.

### For reading S3 object metadata

To keep it simple, attach:

* `AmazonS3ReadOnlyAccess`

> Later, you could replace this with a custom least-privilege policy limited to your bucket only.

Click **Next**

---

## Role name

Set:

* **Role name:** `EduStream-Lambda-Role`

Create the role.

---

# Step 2 — Create Lambda Function

## Go to:

**AWS Console → Lambda → Create function**

Choose:

* **Author from scratch**

Use:

* **Function name:** `EduStream-S3-Processor`
* **Runtime:** `Python 3.12` *(or latest available Python 3.x runtime)*
* **Architecture:** `x86_64`

### Permissions

Choose:

* **Use an existing role**
* select **`EduStream-Lambda-Role`**

Click **Create function**

---

# Step 3 — Add Lambda Code

Open the function and replace the default code with this.

## Lambda code (`lambda_function.py`)

```python id="yvy8ea"
import json
import urllib.parse
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    print("Received event:")
    print(json.dumps(event, indent=2))

    for record in event.get("Records", []):
        bucket = record["s3"]["bucket"]["name"]
        key = urllib.parse.unquote_plus(record["s3"]["object"]["key"])
        event_time = record.get("eventTime", "N/A")

        try:
            response = s3.head_object(Bucket=bucket, Key=key)
            size = response.get("ContentLength", "Unknown")
            content_type = response.get("ContentType", "Unknown")
        except Exception as e:
            print(f"Could not fetch object metadata for {key}: {str(e)}")
            size = "Unknown"
            content_type = "Unknown"

        print("=== EduStream Upload Event ===")
        print(f"Bucket: {bucket}")
        print(f"Object Key: {key}")
        print(f"Event Time: {event_time}")
        print(f"Size (bytes): {size}")
        print(f"Content Type: {content_type}")
        print("==============================")

    return {
        "statusCode": 200,
        "body": json.dumps("S3 event processed successfully")
    }
```

Click **Deploy**

---

# Step 4 — Configure S3 Trigger for Lambda

Now we’ll tell S3 to trigger this Lambda when a file is uploaded.

## Go to:

**S3 → your bucket → `edustream-course-storage-farhan-2026`**

Open:

* **Properties**
* scroll to **Event notifications**
* click **Create event notification**

---

## Event notification settings

### Name

* `edustream-upload-event`

### Prefix filter (recommended)

Since your app uploads files into:

* `courses/notes/`

Set prefix as:

```bash id="fdh5ww"
courses/notes/
```

This ensures Lambda triggers only for course note uploads.

You can leave suffix blank, or optionally use:

* `.pdf`
  if you want only PDFs.

---

## Event types

Select:

* **All object create events**

---

## Destination

Choose:

* **Lambda function**
* select **`EduStream-S3-Processor`**

Save / create the event notification.

---

# Step 5 — Test the Lambda trigger

Now upload a file into the S3 bucket under the matching prefix.

You can do it in either of these ways:

## Option A — Use your Flask app

Open:

```bash id="cpl9s0"
http://YOUR-ALB-DNS-NAME/upload
```

Upload a file through the EduStream app.
Because your Flask app uploads files to:

```python id="f2hpkh"
courses/notes/<filename>
```

it should trigger the Lambda automatically.

---

## Option B — Upload directly in S3 console

Upload a file manually into:

```bash id="jlwmzc"
courses/notes/
```

inside your bucket.

---

# Step 6 — Verify Lambda execution in CloudWatch Logs

Go to:
**CloudWatch → Log groups**

Find the Lambda log group:

```bash id="z0czf4"
/aws/lambda/EduStream-S3-Processor
```

Open the latest log stream.

You should see log output similar to:

```text id="sj0vnn"
=== EduStream Upload Event ===
Bucket: edustream-course-storage-farhan-2026
Object Key: courses/notes/sample.pdf
Event Time: 2026-07-05T...
Size (bytes): 12345
Content Type: application/pdf
==============================
```

That confirms:

* S3 upload happened
* Lambda was triggered
* Lambda successfully read object metadata
* CloudWatch captured the event logs

---

# What this proves in your project

With this phase complete, your architecture demonstrates:

## Serverless processing

* no server required to process upload events

## Event-driven automation

* S3 automatically triggers Lambda

## Monitoring / observability

* Lambda execution logs are available in CloudWatch

## Real cloud integration

* app → S3 → Lambda → CloudWatch

---

# Optional enhancement: write upload metadata into DynamoDB or RDS

If you want to make the project look even more advanced, the Lambda can also write metadata such as:

* file name
* uploaded by
* upload timestamp
* content type
* size

into:

* **DynamoDB** (recommended for event metadata)
  or
* **RDS**

For now, CloudWatch logging is enough to complete the assignment.

---

# Final Phase 8 deliverables

## Lambda role

* `EduStream-Lambda-Role`

## Lambda function

* `EduStream-S3-Processor`

## S3 event notification

* `edustream-upload-event`

## Log group

* `/aws/lambda/EduStream-S3-Processor`

---

# Final architecture after Phase 8

Your project now includes:

* **VPC**
* **Public / private subnets**
* **IAM**
* **EC2**
* **Application Load Balancer**
* **Auto Scaling Group**
* **RDS MySQL**
* **S3**
* **CloudWatch**
* **Lambda**
* **S3 Event Notification**

This is a solid **end-to-end AWS Education domain architecture project**.

---

# What to do now

Please complete these:

1. Create **`EduStream-Lambda-Role`**
2. Create **`EduStream-S3-Processor`** Lambda
3. Add S3 event notification for prefix `courses/notes/`
4. Upload a test file
5. Verify logs in CloudWatch

---

# Reply with:

## **“Phase 8 done”**

and tell me:

* whether the Lambda trigger worked
* whether you saw logs in CloudWatch

---

# After Phase 8

Once you confirm it’s done, I can give you the **final project completion pack** with:

1. **Complete architecture explanation for submission**
2. **Final Draw.io diagram layout**
3. **AWS services used + purpose table**
4. **Step-by-step implementation summary**
5. **Viva/interview questions & answers**
6. **How to present the demo to faculty / interviewer**

If you want that, just say:

## **“prepare final project report”**
