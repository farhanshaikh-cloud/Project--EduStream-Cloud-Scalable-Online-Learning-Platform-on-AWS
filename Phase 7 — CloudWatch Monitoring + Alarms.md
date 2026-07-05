# Phase 7 — CloudWatch Monitoring + Alarms

## Goal

Set up monitoring for:

* **EC2 / Auto Scaling app instances**
* **Application Load Balancer**
* **RDS MySQL**
* optional **custom logs / application logs**

By the end, you’ll have alarms for common issues like:

* high EC2 CPU
* unhealthy ALB targets
* RDS CPU/storage concerns

---

# What we’ll create in this phase

## CloudWatch alarms

1. **EC2 High CPU Alarm**
2. **ALB Unhealthy Host Alarm**
3. **RDS High CPU Alarm**
4. **RDS Low Free Storage Alarm** *(optional but good)*

## Optional log setup

* app logs from EC2 to CloudWatch Logs
* system logs to CloudWatch

For the assignment/demo, the alarms are the key part.
We can add log shipping after that.

---

# Phase 7 plan

We’ll do it in this order:

## Step 1 — Verify basic metrics exist

## Step 2 — Create EC2 CPU alarm

## Step 3 — Create ALB unhealthy target alarm

## Step 4 — Create RDS CPU alarm

## Step 5 — Create RDS free storage alarm

## Step 6 — (Optional) install CloudWatch Agent on EC2 for log collection

## Step 7 — Review alarms dashboard

---

# Step 1 — Verify metrics are available

Go to **CloudWatch → Metrics** and check that you can see metrics for:

* **EC2**
* **ApplicationELB**
* **RDS**

You should, because all these services publish basic metrics automatically.

---

# Step 2 — Create EC2 High CPU Alarm

We want an alarm if an app instance is under heavy load.

## Go to:

**CloudWatch → Alarms → Create alarm**

---

## Select metric

Choose:

* **EC2**
* **Per-Instance Metrics**
* pick the app instances launched by your ASG
* select **CPUUtilization**

If you have multiple ASG instances, you can either:

* create alarms for each instance, or
* use an Auto Scaling group metric later

For now, create at least **one EC2 CPU alarm** for the app tier.

---

## Configure threshold

Use:

* **Statistic:** Average
* **Period:** 5 minutes
* **Threshold type:** Static
* **Whenever CPUUtilization is…** `Greater than 80`

### Datapoints

For example:

* **3 out of 3 datapoints**

So if CPU stays above 80% for 15 minutes, alarm triggers.

---

## Notification

If you want email alerts, create or choose an SNS topic.
If you don’t want to configure email right now, you can still create the alarm without notification.

### Alarm name

Use:

* `EduStream-EC2-HighCPU`

Create the alarm.

---

# Step 3 — Create ALB Unhealthy Host Alarm

This is a very useful HA monitoring alarm.

## Go to:

**CloudWatch → Alarms → Create alarm**

Select metric:

* **ApplicationELB**
* **Per Target Group Metrics**
* find the metric for your target group:

  * **UnHealthyHostCount**

Choose the metric for:

* your target group `edustream-tg`
* and ideally the load balancer associated with it

---

## Threshold

Set:

* **Statistic:** Average
* **Period:** 1 minute or 5 minutes
* **Threshold:** `Greater than or equal to 1`

Meaning:

* if even **one app instance becomes unhealthy**, you get alerted

### Alarm name

* `EduStream-ALB-UnhealthyHosts`

Create it.

---

# Step 4 — Create RDS High CPU Alarm

This tells you if the database is under stress.

## Go to:

**CloudWatch → Alarms → Create alarm**

Select metric:

* **RDS**
* **Per-Database Metrics**
* choose your DB instance:

  * `edustream-mysql-db`
* select **CPUUtilization**

---

## Threshold

Set:

* **Statistic:** Average
* **Period:** 5 minutes
* **Threshold:** `Greater than 80`

### Alarm name

* `EduStream-RDS-HighCPU`

Create it.

---

# Step 5 — Create RDS Free Storage Alarm

This is a good production-style alarm to show you understand DB operations.

## Create another alarm**

Select metric:

* **RDS**
* **Per-Database Metrics**
* choose `edustream-mysql-db`
* select **FreeStorageSpace**

---

## Threshold suggestion

Because free storage is measured in bytes, use a simple threshold such as:

* **Lower than 2000000000**
  which is about **2 GB**

### Alarm name

* `EduStream-RDS-LowStorage`

Create it.

---

# Step 6 — Optional: Add CloudWatch Agent for app logs

This is optional for your assignment, but it’s a nice enhancement because then you can show that the app logs are centrally visible.

If you want, we can ship logs like:

* `/var/log/messages`
* your Flask app log file
* maybe `app.log`

For now I’ll give you the practical way to do it.

---

# Step 6A — SSH into one app EC2 instance

Use one of your app instances and run:

```bash id="tx95y4"
sudo dnf install -y amazon-cloudwatch-agent
```

---

# Step 6B — Create a simple log file for the app

If you’re using the systemd service, logs can already be seen via journal.
But for easier CloudWatch shipping, you can also redirect app logs to a file or ship system logs.

For now, let’s ship:

* `/var/log/messages`

and optionally:

* `/home/ec2-user/edustream-app/app.log` if you have one

---

# Step 6C — Create CloudWatch Agent config

Run:

```bash id="jlwm9o"
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Paste this:

```json id="g1mgql"
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "EduStream-System-Logs",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

Save it.

---

# Step 6D — Start CloudWatch Agent

Run:

```bash id="7duqjv"
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
-s
```

---

# Step 6E — Verify in CloudWatch Logs

Go to:

* **CloudWatch → Log groups**

You should see:

* `EduStream-System-Logs`

---

# Step 7 — Review your final monitoring setup

At the end of Phase 7, you should ideally have these alarms:

## Required alarms

* `EduStream-EC2-HighCPU`
* `EduStream-ALB-UnhealthyHosts`
* `EduStream-RDS-HighCPU`
* `EduStream-RDS-LowStorage`

## Optional log group

* `EduStream-System-Logs`

---

# Suggested assignment explanation for monitoring

You can explain it like this:

> Amazon CloudWatch is used to monitor the EduStream platform across compute, load balancing, and database layers. EC2 CPU alarms help detect overloaded application servers, the ALB unhealthy host alarm identifies application instance failures, and RDS CPU/storage alarms help monitor database performance and capacity. This monitoring layer improves operational visibility and supports faster incident detection and recovery.

---

# Recommended threshold summary

## EC2

* **CPUUtilization > 80%** for 3 evaluation periods

## ALB

* **UnHealthyHostCount >= 1**

## RDS CPU

* **CPUUtilization > 80%**

## RDS storage

* **FreeStorageSpace < 2 GB**

---

# What to do now

Please create the CloudWatch alarms:

1. **EduStream-EC2-HighCPU**
2. **EduStream-ALB-UnhealthyHosts**
3. **EduStream-RDS-HighCPU**
4. **EduStream-RDS-LowStorage**

Optional:
5. Install **CloudWatch Agent** and create `EduStream-System-Logs`

---

# Reply with:

## **“Phase 7 done”**

and tell me:

* which alarms you created
* whether you also configured CloudWatch logs

---

# After Phase 7

Then I’ll take you to **Phase 8: Lambda + S3 event automation**, which will complete the **serverless** part of your assignment architecture.

In that phase, we’ll make this flow work:

**Instructor uploads file to S3 → S3 event triggers Lambda → Lambda logs file metadata / event details**

That will complete:

* **EC2**
* **RDS**
* **S3**
* **ALB**
* **ASG**
* **CloudWatch**
* **Lambda**

which is basically the full AWS education project.
