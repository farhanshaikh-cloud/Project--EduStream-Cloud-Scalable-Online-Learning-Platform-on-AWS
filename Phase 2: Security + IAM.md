Perfect — now we’ll do **Phase 2: Security + IAM** for the EduStream project.

In this phase we’ll create:

1. **IAM role for EC2** → so the app server can access S3 and CloudWatch without hardcoded keys
2. **Security Group for ALB**
3. **Security Group for EC2 App Server**
4. **Security Group for RDS**
5. *(Optional but recommended)* a security group for Lambda if later you want Lambda to access RDS inside the VPC

---

# Phase 2 — IAM + Security Setup

---

# Part A — Create IAM Role for EC2

We want the EC2 application server to be able to:

* read/write to your S3 course-content bucket
* send logs/metrics to CloudWatch
* optionally use SSM later if you want Session Manager access

## Step A1: Open IAM Console

Go to **AWS Console → IAM → Roles → Create role**

## Step A2: Trusted entity

Choose:

* **Trusted entity type:** `AWS service`
* **Use case:** `EC2`

Click **Next**

---

## Step A3: Attach permissions

Attach these policies:

### Required for CloudWatch / basic monitoring

* `CloudWatchAgentServerPolicy`

### For S3 access

For now, to keep setup easy, attach:

* `AmazonS3FullAccess`

> Later, for a cleaner production design, we can replace this with a custom least-privilege S3 policy.

### Optional but useful

* `AmazonSSMManagedInstanceCore`

This helps if you later want to connect through Systems Manager.

So ideally select these 3:

* `CloudWatchAgentServerPolicy`
* `AmazonS3FullAccess`
* `AmazonSSMManagedInstanceCore`

Click **Next**

---

## Step A4: Role name

Set:

* **Role name:** `EduStream-EC2-Role`

Click **Create role**

---

# Part B — Create IAM Instance Profile

Usually AWS automatically handles this when you attach the role to EC2.
So for now you don’t need to create anything extra manually.

---

# Part C — Security Group Design

We’ll create **3 main security groups**.

---

# 1) Security Group for ALB

This security group will allow internet users to access the load balancer.

## Step C1: Create ALB SG

Go to **VPC → Security Groups → Create security group**

Enter:

* **Security group name:** `EduStream-ALB-SG`
* **Description:** `Allow web traffic to ALB`
* **VPC:** `EduStream-VPC`

### Inbound rules

Add:

#### Rule 1

* Type: `HTTP`
* Port: `80`
* Source: `Anywhere-IPv4` (`0.0.0.0/0`)

#### Rule 2

* Type: `HTTPS`
* Port: `443`
* Source: `Anywhere-IPv4` (`0.0.0.0/0`)

### Outbound rules

Keep default:

* **All traffic** → `0.0.0.0/0`

Click **Create security group**

---

# 2) Security Group for EC2 App Server

This SG should **not** allow the whole internet to directly hit the app if you’re using an ALB.
Instead, it should accept traffic **only from the ALB security group**.

## Step C2: Create EC2 App SG

Create another security group:

* **Security group name:** `EduStream-App-SG`
* **Description:** `Allow app traffic from ALB and SSH from admin IP if needed`
* **VPC:** `EduStream-VPC`

### Inbound rules

## Rule 1 — App traffic from ALB

If your app will run on **port 80**:

* Type: `HTTP`
* Port: `80`
* Source: **Security Group**
* Source SG: `EduStream-ALB-SG`

If you plan to run Flask/Node directly on port 5000 or 8000, use that instead.
For assignment/demo simplicity, I recommend your app ultimately runs on **port 80** or behind nginx on port 80.

---

## Rule 2 — SSH access for admin/testing

Add one SSH rule so you can log into the EC2 server.

* Type: `SSH`
* Port: `22`
* Source: **My IP** *(recommended)*

If **My IP** doesn’t work for you, enter your current public IP manually with `/32`.

Example:

* `49.xx.xx.xx/32`

> Do **not** use `0.0.0.0/0` for SSH unless you have no other option.

### Outbound rules

Keep default:

* **All traffic** → `0.0.0.0/0`

Click **Create security group**

---

# 3) Security Group for RDS

The database should only accept traffic from the application server SG.

## Step C3: Create RDS SG

Create another security group:

* **Security group name:** `EduStream-RDS-SG`
* **Description:** `Allow MySQL access only from application servers`
* **VPC:** `EduStream-VPC`

### Inbound rules

If you’re using **MySQL / MariaDB**:

* Type: `MYSQL/Aurora`
* Port: `3306`
* Source: **Security Group**
* Source SG: `EduStream-App-SG`

That means:

* only EC2 instances with `EduStream-App-SG` can reach the DB

### Outbound rules

Keep default or all traffic.

Click **Create security group**

---

# 4) Optional Security Group for Lambda

Only needed if later your Lambda function must access RDS inside private subnets.

If yes, create:

* **Security group name:** `EduStream-Lambda-SG`
* inbound rules: usually none
* outbound: all traffic

Then later add `EduStream-Lambda-SG` as an allowed source in RDS SG if Lambda needs DB access.

For now, **you can skip this**.

---

# Final Security Group Architecture

## **EduStream-ALB-SG**

### Inbound

* 80 from `0.0.0.0/0`
* 443 from `0.0.0.0/0`

### Outbound

* all traffic

---

## **EduStream-App-SG**

### Inbound

* 80 from `EduStream-ALB-SG`
* 22 from **your IP**

### Outbound

* all traffic

---

## **EduStream-RDS-SG**

### Inbound

* 3306 from `EduStream-App-SG`

### Outbound

* all traffic

---

# Important note before Phase 3

Since your **EC2 app server will likely be in a private subnet** in the final architecture, SSH directly to it becomes tricky unless you use:

* a bastion host, or
* Session Manager, or
* temporarily launch the app server in a public subnet during practice

For a **practice project / assignment implementation**, I recommend this flow:

## **Implementation shortcut for now**

For **Version 1 of the build**, launch the EC2 app server in a **public subnet** first so:

* you can SSH easily
* install app dependencies
* test RDS connection
* test S3 upload

Then later we can move to a more production-style private-subnet + ALB design.

That’s the easiest way to finish the project successfully.

---

# What to do now

Please complete these in the console:

## Create:

* IAM role → `EduStream-EC2-Role`
* SG → `EduStream-ALB-SG`
* SG → `EduStream-App-SG`
* SG → `EduStream-RDS-SG`

---

# Reply back with:

## **“Phase 2 done”**

Once you do that, I’ll start **Phase 3: Database Layer (RDS MySQL)** with the **exact AWS Console values** for:

* DB subnet group
* RDS MySQL creation
* username/password setup
* public/private access settings
* attaching the right SG
* how to verify connectivity from EC2

If you want, after that I’ll also give you the **Python Flask app code** for the education portal so the EC2 server can connect to RDS and S3.
