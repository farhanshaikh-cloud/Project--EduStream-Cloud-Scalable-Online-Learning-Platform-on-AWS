# Phase 4 — S3 Storage Setup

## Goal

Create an S3 bucket to store:

* course videos
* PDF notes
* assignments
* student submissions
* course thumbnails / assets

We’ll also create a neat folder structure so it looks like a real project.

---

# What we’ll build in this phase

## S3 bucket

One main bucket, for example:

* `edustream-course-storage-<unique-suffix>`

Since S3 bucket names must be globally unique, you’ll likely need to add something like:

* your initials
* date
* random number

Example:

* `edustream-course-storage-farhan-2026`
* `edustream-course-storage-4512`

---

# Suggested S3 folder structure

Inside the bucket, create these prefixes/folders:

* `courses/videos/`
* `courses/notes/`
* `courses/thumbnails/`
* `assignments/`
* `submissions/`
* `backups/`

This makes the project look organized and production-like.

---

# Part A — Create the S3 Bucket

## Step A1: Open S3 Console

Go to **AWS Console → S3 → Create bucket**

---

## Step A2: Bucket details

Enter:

* **Bucket name:**
  Example: `edustream-course-storage-farhan-2026`

Choose a unique name.

* **AWS Region:**
  Use the **same region** where your EC2/RDS resources are created.

---

## Step A3: Object Ownership

Keep:

* **ACLs disabled (recommended)**

This is the standard modern setting.

---

## Step A4: Block Public Access settings

For this project, keep **Block all public access = ON**

That means:

* files are not publicly exposed by default
* your app/EC2 can still access them using IAM

This is the safer setup.

---

## Step A5: Bucket Versioning

For practice, I recommend:

* **Enable Versioning** → **Yes**

Why:

* useful if files are overwritten or deleted
* nice point to mention in assignment under durability/data protection

If you want to keep it simpler, you can leave it disabled, but I’d enable it.

---

## Step A6: Default encryption

Keep:

* **Server-side encryption with Amazon S3 managed keys (SSE-S3)**

This is a good security best practice.

---

## Step A7: Create bucket

Click **Create bucket**

---

# Part B — Create Folder / Prefix Structure

Open the bucket and create these folders one by one.

## Create these prefixes:

1. `courses/`

2. inside `courses/`, create:

   * `videos/`
   * `notes/`
   * `thumbnails/`

3. in bucket root create:

   * `assignments/`
   * `submissions/`
   * `backups/`

---

# Final structure should look like

```bash id="8d7d80"
courses/
  videos/
  notes/
  thumbnails/
assignments/
submissions/
backups/
```

---

# Part C — Verify EC2 IAM Role Access to S3

In Phase 2, you created the role:

* `EduStream-EC2-Role`

and attached:

* `AmazonS3FullAccess`
* `CloudWatchAgentServerPolicy`
* `AmazonSSMManagedInstanceCore` *(optional)*

So from a permissions perspective, your future EC2 app server should be able to:

* upload files to S3
* list bucket objects
* download files

For now, just verify the role exists in IAM.

---

# Part D — Optional Bucket Policy

For this project, **you do not need a bucket policy right now** if:

* bucket is private
* EC2 will access it through IAM role
* Lambda later can also use IAM role permissions

So for **Phase 4**, skip bucket policy unless you specifically need public access or cross-account access.

---

# Part E — Optional Lifecycle Rule (Good for assignment marks)

If you want your project to look more complete, add a lifecycle rule for old backups.

## Example lifecycle rule

* move objects in `backups/` to cheaper storage after 30 days
* or expire old temp files after 90 days

You can skip this for now if you want to move fast.

---

# Part F — Optional CORS

You **do not need CORS now** unless:

* your frontend directly uploads from browser to S3, or
* you host a frontend that calls S3 objects from another origin

For our current implementation, skip CORS.

---

# Phase 4 Deliverables

At the end of this phase you should have:

## S3 bucket created

Example:

* `edustream-course-storage-farhan-2026`

## Folder structure

* `courses/videos/`
* `courses/notes/`
* `courses/thumbnails/`
* `assignments/`
* `submissions/`
* `backups/`

## Bucket settings

* private bucket
* versioning enabled
* default encryption enabled

---

# Important for next phase

Please **save the exact S3 bucket name**, because in Phase 5 we’ll use it inside the application code.

Example:

```bash id="1f0e56"
S3_BUCKET = "edustream-course-storage-farhan-2026"
```

---

# What to do now

Complete the following:

## Create:

* S3 bucket
* folder structure inside it

---

# Reply with:

## **“Phase 4 done”**

and also send me:

1. **your S3 bucket name**
2. **your RDS endpoint**
3. confirm whether you want the app in **Python Flask** or **Node.js**

---

# My recommendation for Phase 5

Use **Python Flask** because it’s the easiest for:

* EC2 deployment
* MySQL connection
* S3 upload integration
* assignment demo

Once you send **Phase 4 done + bucket name + RDS endpoint**, I’ll start **Phase 5: EC2 App Server + full EduStream Flask app deployment**, including:

* EC2 launch settings
* security group attachment
* attaching IAM role
* installing Python + pip + virtualenv
* Flask app code
* MySQL connection code
* S3 upload code
* test pages for student/course data

So after Phase 5, you’ll actually have a **working AWS education app** running.
