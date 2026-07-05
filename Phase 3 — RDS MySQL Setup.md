
# Phase 3 — RDS MySQL Setup

## Goal

Create a MySQL database for storing:

* students
* instructors
* courses
* enrollments
* quiz data
* assignments metadata

We’ll place the DB in the **private DB subnets** you created in Phase 1.

---

# Part A — Create DB Subnet Group

RDS needs a **DB subnet group** so it knows which subnets to use for the database.

## Step A1: Open RDS Console

Go to **AWS Console → RDS**

In the left menu:

* click **Subnet groups**
* click **Create DB subnet group**

---

## Step A2: Enter subnet group details

Use these values:

* **Name:** `edustream-db-subnet-group`
* **Description:** `DB subnet group for EduStream MySQL database`
* **VPC:** `EduStream-VPC`

### Add Availability Zones / Subnets

Select the **two private DB subnets**:

* `EduStream-Private-DB-1`
* `EduStream-Private-DB-2`

Click **Create**

---

# Part B — Create RDS MySQL Database

Now we’ll create the actual database.

## Step B1: Go to Databases

In **RDS Console → Databases → Create database**

---

## Step B2: Choose creation method

Select:

* **Standard create**

---

## Step B3: Engine options

Choose:

* **Engine type:** `MySQL`

### Version

Pick the latest stable MySQL version available in your console.
If you want a safe common option, use **MySQL 8.0.x**.

---

## Step B4: Templates

Choose:

* **Free tier** if available and you want the cheapest setup
  **OR**
* **Dev/Test**

### My recommendation:

Choose **Dev/Test** if free tier isn’t available with your network choices.
If free tier is available and works, use it.

---

# Part C — DB Settings

## Step C1: DB instance identifier

Set:

* **DB instance identifier:** `edustream-mysql-db`

## Step C2: Master username

Set something simple like:

* **Master username:** `admin`

## Step C3: Master password

Create a password you’ll remember, for example:

* **Master password:** `EduStream@123`

Use your own secure password, but save it carefully.

> You’ll need this later in the EC2 application code.

---

# Part D — Instance Configuration

## For a practice project, choose a small instance

Set:

* **DB instance class:** `db.t3.micro`
  or the cheapest small class available in your account/region

---

# Part E — Storage

Use:

* **General Purpose SSD (gp3)** if available
* **Allocated storage:** 20 GB is enough for practice

You can leave storage autoscaling disabled for now if you want to keep it simple.

---

# Part F — Connectivity (Important)

This section matters a lot.

## Step F1: Compute resource

If AWS asks **Connect to an EC2 compute resource?**

* choose **Don’t connect to an EC2 compute resource**

We’ll connect manually from our app server later.

---

## Step F2: Network settings

Choose:

* **VPC:** `EduStream-VPC`

### DB subnet group

Select:

* `edustream-db-subnet-group`

### Public access

Set:

* **No**

This is important — database should stay private.

### VPC security group

Choose:

* **Existing**
* select **`EduStream-RDS-SG`**

Make sure it is **not** using the default SG if you don’t want that.

---

## Step F3: Availability Zone

You can leave **No preference** if it’s single-AZ, or choose one of the DB subnets’ AZs.

---

# Part G — Database Authentication

Keep default:

* **Password authentication**

---

# Part H — Additional Configuration

Expand **Additional configuration**.

## Database name

Set initial database name:

* **Initial database name:** `edustream`

This will create a DB named `edustream` inside MySQL.

---

## Backup

For a practice project, keep backup retention small/default.

---

## Encryption

Default is fine. If encryption is enabled by default, keep it.

---

## Monitoring

You can keep defaults for now.

---

## Log exports

Optional — you can skip for now.

---

# Part I — Final Review Before Create

Before clicking **Create database**, verify these values:

## Final RDS values

* Engine: **MySQL**
* DB identifier: `edustream-mysql-db`
* Username: `admin`
* Password: *(your chosen password)*
* DB class: `db.t3.micro`
* VPC: `EduStream-VPC`
* DB subnet group: `edustream-db-subnet-group`
* Public access: **No**
* Security group: `EduStream-RDS-SG`
* Initial DB name: `edustream`

Then click **Create database**

---

# Part J — Wait for Database to Become Available

RDS provisioning takes a few minutes.

In **RDS → Databases**, wait until status becomes:

* **Available**

---

# Part K — Save the DB Endpoint

Once the DB is available:

1. Click the database `edustream-mysql-db`
2. Go to **Connectivity & security**
3. Copy the **Endpoint**

It will look like something like:

```bash
edustream-mysql-db.xxxxxx.us-east-1.rds.amazonaws.com
```

Also note:

* **Port:** `3306`

Save these 4 values somewhere:

## DB connection details

* **Host / Endpoint** = RDS endpoint
* **Port** = `3306`
* **Database name** = `edustream`
* **Username** = `admin`
* **Password** = the one you set

---

# Very Important Check

Your RDS SG already allows **3306 from `EduStream-App-SG`**.
That means later your EC2 app server must use **`EduStream-App-SG`** so it can connect to the DB.

---

# Recommended Database Design for this project

Later, once EC2 is ready, we’ll create tables like:

* `students`
* `instructors`
* `courses`
* `enrollments`
* `assignments`

For now, just create the RDS instance.

---

# What to do now

Please complete these in AWS Console:

## Create:

* DB subnet group → `edustream-db-subnet-group`
* RDS MySQL → `edustream-mysql-db`

---

# Reply with:

## **“Phase 3 done”**

And if possible, also send me:

1. **the RDS endpoint** *(you can mask part of it if you want)*
2. whether the DB status is **Available**

Then I’ll start **Phase 4: S3 + IAM access setup**, where we’ll create:

* S3 bucket for course content
* folder structure for videos/assignments/submissions
* EC2 access policy/role verification
* bucket permissions
* optional versioning

After that we’ll move to **Phase 5: EC2 application server + Flask app deployment**.
