
# Phase 5 — EC2 App Server + Flask App Deployment

## What we’ll build in this phase

A working Flask web app with these pages:

* `/` → Home page
* `/students` → Add/list students from RDS
* `/courses` → Add/list courses from RDS
* `/upload` → Upload a file to S3

So your architecture will actually use:

* **EC2** → hosts the Flask app
* **RDS MySQL** → stores students/courses
* **S3** → stores uploaded files
* **IAM role** → lets EC2 access S3 securely

---

# Important recommendation before you start

For the **practice build**, launch this EC2 instance in a **public subnet** so you can SSH into it easily and test the app.

Later, in Phase 6, we’ll improve the architecture with **ALB + Auto Scaling** and move toward the final design.

---

# Before starting Phase 5, I need 2 details from you

Please send me:

1. **Your RDS endpoint**
2. **Your S3 bucket name**

You can mask part of the endpoint if you want, like:

* `edustream-mysql-db.xxxxx.us-east-1.rds.amazonaws.com`

I need those values because I’ll give you the **exact app.py code** with your bucket/database placeholders.

---

# While you send that, here is the full Phase 5 plan

---

# Part A — Launch EC2 Instance

## Step A1: Open EC2 Console

Go to **EC2 → Instances → Launch instance**

---

## Step A2: Name

Set:

* **Name:** `EduStream-App-Server`

---

## Step A3: AMI

Choose:

* **Amazon Linux 2023 AMI**
  or Amazon Linux 2 if that’s what you prefer.

I recommend **Amazon Linux 2023**.

---

## Step A4: Instance type

Choose:

* `t2.micro` or `t3.micro`

---

## Step A5: Key pair

Select your existing key pair or create one.

You’ll need this to SSH into the instance.

---

## Step A6: Network settings

Click **Edit** in Network settings.

Use these values:

* **VPC:** `EduStream-VPC`
* **Subnet:** `EduStream-Public-1` *(important for easy access right now)*
* **Auto-assign public IP:** **Enable**
* **Firewall / Security group:** select existing security group
* choose **`EduStream-App-SG`**

---

## Step A7: IAM role

In **Advanced details**, attach the IAM role:

* **IAM instance profile:** `EduStream-EC2-Role`

This is important for S3 access.

---

## Step A8: Storage

Default 8 GB is okay for practice.

Click **Launch instance**

---

# Part B — Connect to EC2

Once instance is running, SSH into it.

Example:

```bash
ssh -i your-key.pem ec2-user@<EC2-PUBLIC-IP>
```

---

# Part C — Install Python + Flask + MySQL dependencies

Once logged in to EC2, run these commands **one by one**.

## For Amazon Linux 2023

```bash
sudo dnf update -y
sudo dnf install -y python3 python3-pip git
```

Now install app dependencies:

```bash
pip3 install flask pymysql boto3
```

If `pip3` needs root:

```bash
sudo pip3 install flask pymysql boto3
```

---

# Part D — Create Project Directory

Run:

```bash
mkdir ~/edustream-app
cd ~/edustream-app
```

---

# Part E — Create the Flask App Code

Now create the app file:

```bash
nano app.py
```

Paste the code below.

---

# **EduStream Flask App (`app.py`)**

> Replace the placeholders before saving:

* `YOUR_RDS_ENDPOINT`
* `YOUR_DB_PASSWORD`
* `YOUR_S3_BUCKET`

```python
from flask import Flask, request, redirect, render_template_string
import pymysql
import boto3
import os
from werkzeug.utils import secure_filename

app = Flask(__name__)

# =========================
# CONFIGURATION
# =========================
DB_HOST = "YOUR_RDS_ENDPOINT"
DB_USER = "admin"
DB_PASSWORD = "YOUR_DB_PASSWORD"
DB_NAME = "edustream"
S3_BUCKET = "YOUR_S3_BUCKET"
AWS_REGION = "us-east-1"   # change if your region is different

# S3 client (uses EC2 IAM Role)
s3 = boto3.client("s3", region_name=AWS_REGION)

# =========================
# DATABASE CONNECTION
# =========================
def get_db_connection():
    return pymysql.connect(
        host=DB_HOST,
        user=DB_USER,
        password=DB_PASSWORD,
        database=DB_NAME,
        cursorclass=pymysql.cursors.DictCursor
    )

# =========================
# INIT DB TABLES
# =========================
def init_db():
    conn = get_db_connection()
    with conn.cursor() as cursor:
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS students (
                id INT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                email VARCHAR(100) NOT NULL
            )
        """)
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS courses (
                id INT AUTO_INCREMENT PRIMARY KEY,
                title VARCHAR(100) NOT NULL,
                instructor VARCHAR(100) NOT NULL
            )
        """)
    conn.commit()
    conn.close()

# =========================
# HOME PAGE
# =========================
@app.route("/")
def home():
    return render_template_string("""
    <h1>EduStream Cloud - Education Portal</h1>
    <ul>
        <li><a href="/students">Manage Students</a></li>
        <li><a href="/courses">Manage Courses</a></li>
        <li><a href="/upload">Upload Course File to S3</a></li>
    </ul>
    """)

# =========================
# STUDENTS PAGE
# =========================
@app.route("/students", methods=["GET", "POST"])
def students():
    conn = get_db_connection()
    with conn.cursor() as cursor:
        if request.method == "POST":
            name = request.form["name"]
            email = request.form["email"]
            cursor.execute("INSERT INTO students (name, email) VALUES (%s, %s)", (name, email))
            conn.commit()
            return redirect("/students")

        cursor.execute("SELECT * FROM students")
        students_data = cursor.fetchall()

    conn.close()

    html = """
    <h1>Students</h1>
    <form method="post">
        Name: <input type="text" name="name" required><br><br>
        Email: <input type="email" name="email" required><br><br>
        <button type="submit">Add Student</button>
    </form>
    <br>
    <h2>Student List</h2>
    <ul>
    """
    for s in students_data:
        html += f"<li>{s['name']} - {s['email']}</li>"
    html += """
    </ul>
    <br><a href="/">Back to Home</a>
    """
    return html

# =========================
# COURSES PAGE
# =========================
@app.route("/courses", methods=["GET", "POST"])
def courses():
    conn = get_db_connection()
    with conn.cursor() as cursor:
        if request.method == "POST":
            title = request.form["title"]
            instructor = request.form["instructor"]
            cursor.execute("INSERT INTO courses (title, instructor) VALUES (%s, %s)", (title, instructor))
            conn.commit()
            return redirect("/courses")

        cursor.execute("SELECT * FROM courses")
        courses_data = cursor.fetchall()

    conn.close()

    html = """
    <h1>Courses</h1>
    <form method="post">
        Course Title: <input type="text" name="title" required><br><br>
        Instructor: <input type="text" name="instructor" required><br><br>
        <button type="submit">Add Course</button>
    </form>
    <br>
    <h2>Course List</h2>
    <ul>
    """
    for c in courses_data:
        html += f"<li>{c['title']} - Instructor: {c['instructor']}</li>"
    html += """
    </ul>
    <br><a href="/">Back to Home</a>
    """
    return html

# =========================
# FILE UPLOAD TO S3
# =========================
@app.route("/upload", methods=["GET", "POST"])
def upload_file():
    message = ""
    if request.method == "POST":
        file = request.files["file"]
        if file:
            filename = secure_filename(file.filename)
            s3_key = f"courses/notes/{filename}"
            s3.upload_fileobj(file, S3_BUCKET, s3_key)
            message = f"File uploaded successfully to S3: {s3_key}"

    return render_template_string("""
    <h1>Upload Course File</h1>
    <form method="post" enctype="multipart/form-data">
        Select file: <input type="file" name="file" required><br><br>
        <button type="submit">Upload</button>
    </form>
    <p>{{ message }}</p>
    <br><a href="/">Back to Home</a>
    """, message=message)

# =========================
# MAIN
# =========================
if __name__ == "__main__":
    init_db()
    app.run(host="0.0.0.0", port=80)
```

---

# Part F — Replace the placeholders in `app.py`

You must edit these values:

```python
DB_HOST = "YOUR_RDS_ENDPOINT"
DB_PASSWORD = "YOUR_DB_PASSWORD"
S3_BUCKET = "YOUR_S3_BUCKET"
AWS_REGION = "us-east-1"
```

### Example:

```python
DB_HOST = "edustream-mysql-db.xxxxxx.us-east-1.rds.amazonaws.com"
DB_PASSWORD = "EduStream@123"
S3_BUCKET = "edustream-course-storage-farhan-2026"
AWS_REGION = "us-east-1"
```

Save the file:

* Press `Ctrl + O`
* Enter
* `Ctrl + X`

---

# Part G — Start the Flask App

Run:

```bash
sudo python3 app.py
```

If everything is correct, you should see Flask start.

---

# Part H — Open the App in Browser

In EC2 console, copy the **public IPv4 address** of your instance.

Open:

```bash
http://YOUR-EC2-PUBLIC-IP/
```

You should see:

* EduStream Cloud - Education Portal
* Manage Students
* Manage Courses
* Upload Course File to S3

---

# Part I — Test the App

## Test 1 — Students page

Go to:

```bash
http://YOUR-EC2-PUBLIC-IP/students
```

Add a student and verify it saves.

---

## Test 2 — Courses page

Go to:

```bash
http://YOUR-EC2-PUBLIC-IP/courses
```

Add a course and verify it saves.

---

## Test 3 — Upload page

Go to:

```bash
http://YOUR-EC2-PUBLIC-IP/upload
```

Upload a file and then check your S3 bucket under:

```bash
courses/notes/
```

---

# Troubleshooting checks

## 1) If the app doesn’t open in browser

Check:

* EC2 is in **public subnet**
* EC2 has **public IP**
* `EduStream-App-SG` allows:

  * **HTTP 80** from somewhere you can access
  * or from `0.0.0.0/0` temporarily for testing

### Important note:

Right now your `EduStream-App-SG` probably only allows **HTTP from ALB-SG**, which is correct for final architecture but won’t let you directly open the EC2 app in a browser.

So for **testing Phase 5**, temporarily add this inbound rule to **EduStream-App-SG**:

* Type: `HTTP`
* Port: `80`
* Source: `0.0.0.0/0`

Later in Phase 6, once ALB is in front, we’ll remove it.

---

## 2) If DB connection fails

Check:

* EC2 uses **EduStream-App-SG**
* RDS uses **EduStream-RDS-SG**
* RDS SG allows **3306 from EduStream-App-SG**
* DB username/password are correct
* RDS is in **Available** state

---

## 3) If S3 upload fails

Check:

* EC2 instance has **IAM role `EduStream-EC2-Role` attached**
* role has `AmazonS3FullAccess`
* bucket name is correct
* region is correct

---

# What I need from you now

Please send me these 2 values so I can tailor the app config exactly:

1. **RDS endpoint**
2. **S3 bucket name**

Then I’ll do one more thing for you:

## I’ll give you the **final exact `app.py` with your values filled in**, plus the **commands to run it as a background service** so it stays running after logout.

---

# Your action for now

## 1. Launch the EC2 instance

Use:

* public subnet
* public IP enabled
* SG = `EduStream-App-SG`
* IAM role = `EduStream-EC2-Role`

## 2. Temporarily add this rule to `EduStream-App-SG`

* HTTP 80 from `0.0.0.0/0`

## 3. SSH into the instance and deploy the Flask app

---

# Reply with:

## **“EC2 launched”**

and paste:

* **RDS endpoint**
* **S3 bucket name**
* **AWS region**

Then I’ll give you the **ready-to-run final code and background service setup**.
