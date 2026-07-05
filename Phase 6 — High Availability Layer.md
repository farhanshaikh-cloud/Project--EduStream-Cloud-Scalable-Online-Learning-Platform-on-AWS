# Phase 6 — High Availability Layer

## Goal of this phase

We will upgrade the current working app to:

* **Application Load Balancer**
* **Target Group**
* **Launch Template**
* **Auto Scaling Group**
* **2 EC2 app instances across 2 AZs**
* ALB URL as the main access point

So the flow becomes:

**User → ALB → EC2 instances → RDS / S3**

---

# Important note before starting

Right now, your current EC2 server is probably:

* in a **public subnet**
* reachable directly with HTTP 80
* running the app manually / via systemd

For the HA phase, we’ll use that server as the **golden source** to create an **AMI**, then build an Auto Scaling Group from it.

This is the easiest path because your app is already installed and working.

---

# Phase 6 plan

We’ll do it in this order:

## Step 1 — Make sure the current EC2 app server is production-ready

## Step 2 — Create an AMI from the working EC2 instance

## Step 3 — Create a Target Group

## Step 4 — Create an Application Load Balancer

## Step 5 — Create a Launch Template from the AMI

## Step 6 — Create an Auto Scaling Group across 2 subnets

## Step 7 — Test access through the ALB DNS

## Step 8 — Lock down the EC2 security group so only ALB can reach it

---

# Step 1 — Before creating the AMI, verify the app starts automatically

This matters because new EC2 instances launched by the Auto Scaling Group must automatically start the app.

If you already created the `edustream` systemd service in Phase 5, that’s good.

## On the current EC2 instance, run:

```bash
sudo systemctl status edustream
```

You want to see it as **active (running)**.

If you didn’t create the service yet, do it now.

---

# Create systemd service if not already done

```bash
sudo nano /etc/systemd/system/edustream.service
```

Paste:

```ini
[Unit]
Description=EduStream Flask App
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/edustream-app
ExecStart=/usr/bin/python3 /home/ec2-user/edustream-app/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Then run:

```bash
sudo systemctl daemon-reload
sudo systemctl enable edustream
sudo systemctl start edustream
sudo systemctl status edustream
```

---

# Step 2 — Create an AMI from the working EC2 instance

This AMI will contain:

* your Flask app code
* installed Python packages
* systemd service
* all current configuration

## In AWS Console:

Go to **EC2 → Instances**
Select your working instance: `EduStream-App-Server`

Then:

* **Actions → Image and templates → Create image**

Use:

* **Image name:** `EduStream-App-AMI`
* **Image description:** `AMI for EduStream Flask app server`

Leave defaults and click **Create image**

---

## Wait for the AMI to become available

Go to:

* **EC2 → AMIs**

Wait until `EduStream-App-AMI` status becomes **Available**

---

# Step 3 — Create a Target Group

The ALB needs a target group to know which instances to send traffic to.

## Go to:

**EC2 → Target Groups → Create target group**

Choose:

* **Target type:** `Instances`
* **Target group name:** `edustream-tg`
* **Protocol:** `HTTP`
* **Port:** `80`
* **VPC:** `EduStream-VPC`

### Health check settings

Keep:

* **Health check protocol:** HTTP
* **Health check path:** `/`

That’s good because your Flask app home page responds on `/`.

Click **Next**

You can skip manual instance registration for now because the Auto Scaling Group will register instances automatically.

Click **Create target group**

---

# Step 4 — Create Application Load Balancer

## Go to:

**EC2 → Load Balancers → Create Load Balancer**
Choose:

* **Application Load Balancer**

---

## ALB basic configuration

Use:

* **Name:** `EduStream-ALB`
* **Scheme:** `Internet-facing`
* **IP address type:** `IPv4`

---

## Network mapping

Choose:

* **VPC:** `EduStream-VPC`

Select **both public subnets**:

* `EduStream-Public-1`
* `EduStream-Public-2`

This is important because the ALB must be in public subnets.

---

## Security groups

Attach:

* **`EduStream-ALB-SG`**

This SG should allow:

* HTTP 80 from `0.0.0.0/0`
* HTTPS 443 from `0.0.0.0/0`

---

## Listeners and routing

For listener:

* **Protocol:** HTTP
* **Port:** 80

Default action:

* forward to **`edustream-tg`**

Click **Create load balancer**

---

# Step 5 — Create Launch Template

Now we’ll create a Launch Template that the ASG will use to launch app instances.

## Go to:

**EC2 → Launch Templates → Create launch template**

Use:

* **Launch template name:** `EduStream-App-LT`

---

## AMI

Choose **My AMIs**
Select:

* **`EduStream-App-AMI`**

---

## Instance type

Choose:

* `t2.micro` or `t3.micro`

---

## Key pair

Select the same key pair you used earlier
(optional, but useful if you want to SSH into ASG instances)

---

## Network settings

For launch template, don’t hardcode a subnet here — the ASG will choose subnets.

### Security group

Attach:

* **`EduStream-App-SG`**

### IAM instance profile

In **Advanced details**, choose:

* **`EduStream-EC2-Role`**

This is important for S3 access.

---

## User data

You can leave it empty because the AMI already contains the app and systemd service.

Click **Create launch template**

---

# Step 6 — Create Auto Scaling Group

## Go to:

**EC2 → Auto Scaling Groups → Create Auto Scaling group**

Use:

* **Auto Scaling group name:** `EduStream-ASG`
* **Launch template:** `EduStream-App-LT`

Click **Next**

---

## Network

Choose:

* **VPC:** `EduStream-VPC`

### Select subnets for EC2 instances

Use the **two public subnets for now** to keep it simple and testable:

* `EduStream-Public-1`
* `EduStream-Public-2`

> In a stricter production design, app instances would sit in private app subnets behind the ALB.
> But for your assignment + working implementation, using the public subnets here is okay and much easier to finish.

Click **Next**

---

## Load balancing

Choose:

* **Attach to an existing load balancer**
* **Choose from your load balancer target groups**
* select **`edustream-tg`**

---

## Health checks

Enable:

* **Elastic Load Balancing health checks**

Click **Next**

---

## Group size and scaling

Set:

* **Desired capacity:** `2`
* **Minimum capacity:** `2`
* **Maximum capacity:** `4`

This gives you:

* 2 running app instances by default
* room to scale if needed

Click **Next** through the rest of the wizard.

You can skip advanced scaling policies for now or add one simple policy later.

Create the ASG.

---

# Step 7 — Wait for instances to launch

Once ASG is created:

* it should launch **2 EC2 instances**
* those instances should register into **`edustream-tg`**

Go to:

* **EC2 → Instances** and verify 2 new instances launch
* **EC2 → Target Groups → edustream-tg → Targets**

You want both targets to become:

* **healthy**

---

# Step 8 — Test the app through the ALB

Go to:

* **EC2 → Load Balancers**
* click **EduStream-ALB**
* copy the **DNS name**

It will look something like:

```bash
EduStream-ALB-123456789.us-east-1.elb.amazonaws.com
```

Open in browser:

```bash
http://YOUR-ALB-DNS-NAME/
```

You should see your **EduStream Cloud** home page.

Test:

* `/`
* `/students`
* `/courses`
* `/upload`

All should work through the ALB URL.

---

# Step 9 — Important security cleanup after ALB works

Once the ALB is working, **remove direct public access to the EC2 app instances**.

Go to **EduStream-App-SG** and remove the temporary rule:

* **HTTP 80 from `0.0.0.0/0`**

Keep only:

* **HTTP 80 from `EduStream-ALB-SG`**
* **SSH 22 from your IP** *(if you still want admin access)*

This makes the architecture cleaner:

* users hit the **ALB**
* ALB talks to **EC2**
* EC2 is not directly exposed for web traffic

---

# Recommended final security group state after Phase 6

## EduStream-ALB-SG

### Inbound

* 80 from `0.0.0.0/0`
* 443 from `0.0.0.0/0`

---

## EduStream-App-SG

### Inbound

* 80 from `EduStream-ALB-SG`
* 22 from your IP only

---

## EduStream-RDS-SG

### Inbound

* 3306 from `EduStream-App-SG`

---

# Common issues and fixes

---

# Issue 1 — Targets show unhealthy

Check:

## A. App is actually running on new ASG instances

SSH into one ASG instance and run:

```bash
sudo systemctl status edustream
```

If not running, the AMI may have been created before the app service was properly enabled.

### Fix:

* go back to original EC2
* ensure service is enabled
* create a new AMI
* update launch template / ASG

---

## B. Health check path

Target group health check path should be:

```bash
/
```

---

## C. Security groups

Make sure:

* `EduStream-App-SG` allows HTTP 80 from `EduStream-ALB-SG`

---

# Issue 2 — ALB opens but app gives DB or S3 errors

That means the new ASG instances are missing something.

Check:

* launch template includes **IAM role**
* app code in AMI still contains correct RDS endpoint / bucket name
* RDS SG allows `EduStream-App-SG`
* ASG instances actually use `EduStream-App-SG`

---

# What I want you to do now

Please complete these in the console:

## Create:

1. **AMI** from working EC2 → `EduStream-App-AMI`
2. **Target Group** → `edustream-tg`
3. **Application Load Balancer** → `EduStream-ALB`
4. **Launch Template** → `EduStream-App-LT`
5. **Auto Scaling Group** → `EduStream-ASG`

---

# Reply with:

## **“Phase 6 done”**

and send me:

1. **ALB DNS name**
2. whether target group shows **healthy**
3. whether app is working through the ALB URL

---

# After that: Phase 7

Once Phase 6 is done, I’ll guide you through **Phase 7: CloudWatch Monitoring + Alarms**, where we’ll add:

* EC2 CPU alarms
* ALB health monitoring
* RDS monitoring
* log visibility
* architecture monitoring section for your assignment

That will make the project look complete from an AWS operations perspective.
