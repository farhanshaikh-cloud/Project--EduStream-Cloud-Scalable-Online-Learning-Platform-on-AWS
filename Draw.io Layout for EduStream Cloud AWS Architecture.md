Below is a **Draw.io layout blueprint** you can follow exactly for your EduStream AWS architecture diagram. I’ve structured it so you can recreate it cleanly using **AWS Architecture Icons** in Draw.io / diagrams.net.

---

# Draw.io Layout for **EduStream Cloud AWS Architecture**

## 1) Canvas orientation

Use **landscape layout**.

Recommended flow:
**Left → Right** or **Top → Bottom**, but for this project the cleanest is:

## **Top to Bottom structure**

1. **Users / Internet**
2. **AWS Cloud**
3. **EduStream VPC**
4. **Public Subnets (ALB + App Tier)**
5. **Private DB Subnets (RDS)**
6. **AWS managed services outside VPC**
   (S3, Lambda, CloudWatch, IAM)

---

# 2) Main blocks to create on canvas

Create these **outer sections** in order:

## A. Top section

A small box or icon group for:

* **Student / Instructor / Admin Users**
* **Internet**

## B. Middle large section

A large outer boundary labeled:

# **AWS Cloud**

Inside AWS Cloud, create another large boundary labeled:

# **EduStream-VPC**

Inside the VPC, create:

* **2 Public Subnets**
* **2 Private DB Subnets**

## C. Right-side / lower-side service area outside VPC

Place:

* **Amazon S3**
* **AWS Lambda**
* **Amazon CloudWatch**
* **IAM Roles**

---

# 3) Exact layout structure to draw

---

# Top Row: Users and Internet

At the very top center place:

### Box / Icons:

* **Users**

  * Student
  * Instructor
  * Admin

Below that, place:

* **Internet**

Below Internet, place:

* **Application Load Balancer**

So top flow becomes:

```text
Users
  ↓
Internet
  ↓
Application Load Balancer
```

---

# 4) Create the VPC boundary

Below the ALB, draw one large rectangle labeled:

# **EduStream-VPC**

Inside this VPC, divide the space into **two horizontal layers**:

## Upper VPC layer

For **Public Subnets**

## Lower VPC layer

For **Private DB Subnets**

---

# 5) Public subnet section layout

Inside the VPC, in the **upper half**, create **two side-by-side subnet boxes**.

## Left public subnet

Label it:

# **Public Subnet 1 (AZ-1)**

Inside it place:

* **EC2 Instance – EduStream App 1**

Optional small label:

* Flask App
* Auto Scaling member

---

## Right public subnet

Label it:

# **Public Subnet 2 (AZ-2)**

Inside it place:

* **EC2 Instance – EduStream App 2**

Optional small label:

* Flask App
* Auto Scaling member

---

# 6) Show the Auto Scaling Group boundary

Around both EC2 instances, draw a dotted / grouped boundary labeled:

# **Auto Scaling Group – EduStream-ASG**

This makes it clear both EC2 instances belong to the same scalable application tier.

---

# 7) Show ALB connection to both EC2 instances

Place the **Application Load Balancer** above the VPC or at the top edge of the VPC.

Draw arrows:

* **ALB → EC2 App 1**
* **ALB → EC2 App 2**

You can label this connection:

* **HTTP/HTTPS traffic**
* **Load-balanced application requests**

---

# 8) Private DB subnet section layout

Inside the **lower half** of the VPC, create **two side-by-side subnet boxes**.

## Left DB subnet

Label:

# **Private DB Subnet 1 (AZ-1)**

## Right DB subnet

Label:

# **Private DB Subnet 2 (AZ-2)**

Now place **one Amazon RDS MySQL icon centered across both DB subnets**, or place it inside one DB subnet with a note saying it belongs to the **DB Subnet Group**.

Best visual option:

### Center between both DB subnets:

* **Amazon RDS MySQL**
* label it:

  * `edustream-mysql-db`
  * `Database: edustream`

Above or below it write:

* **DB Subnet Group**
* **Private / No Public Access**

---

# 9) Draw EC2 → RDS connections

From both app EC2 instances, draw arrows downward to RDS.

Label them:

* **MySQL 3306**
* **Student / Course data**

So it visually shows:

* app instances query the database

---

# 10) Place S3 outside the VPC

Now to the **right side of the VPC** place an **Amazon S3** icon.

Label it:

# **Amazon S3**

### Bucket:

`edustream-course-storage-farhan-2026`

Below that, add small folder notes:

* `courses/videos/`
* `courses/notes/`
* `assignments/`
* `submissions/`

---

# 11) Draw EC2 → S3 connection

From the **Auto Scaling Group / EC2 tier**, draw an arrow to **S3**.

Label it:

* **Upload course files**
* **Store notes / content**
* **Boto3 via IAM role**

This represents the Flask app uploading files to S3.

---

# 12) Place Lambda below or beside S3

Now place **AWS Lambda** close to S3, preferably **below S3**.

Label it:

# **AWS Lambda**

### `EduStream-S3-Processor`

Draw an arrow:

* **S3 → Lambda**

Label that arrow:

* **S3 Event Notification**
* **ObjectCreated**
* **Prefix: courses/notes/**

This is one of the most important parts of the diagram because it shows the **serverless flow**.

---

# 13) Place CloudWatch near Lambda and to the right of AWS services

Place **Amazon CloudWatch** icon on the far right side.

Label it:

# **Amazon CloudWatch**

* Metrics
* Alarms
* Logs

---

# 14) Draw CloudWatch monitoring connections

Now connect **CloudWatch** with dotted arrows or monitoring arrows from:

### From EC2 / ASG

Label:

* EC2 CPU Alarm
* App monitoring

### From ALB

Label:

* Unhealthy Host Alarm

### From RDS

Label:

* RDS CPU
* Free Storage Alarm

### From Lambda

Label:

* Lambda execution logs

This visually shows CloudWatch as the monitoring layer for the full architecture.

---

# 15) Place IAM roles near EC2 and Lambda

Place **IAM** icons either:

* on the far left of the AWS service side, or
* below the VPC

Use **two IAM role labels**:

## IAM Role 1

**EduStream-EC2-Role**

Draw arrow to:

* EC2 App Tier

Label permissions:

* S3 access
* CloudWatch access

## IAM Role 2

**EduStream-Lambda-Role**

Draw arrow to:

* Lambda

Label permissions:

* CloudWatch Logs
* S3 read metadata

---

# 16) Add Internet Gateway in the VPC diagram

To make the network layer more complete, add an **Internet Gateway** icon at the top edge of the VPC or between Internet and ALB.

Recommended placement:

* **Internet → Internet Gateway → ALB**

If you want a cleaner diagram, you can keep:

* **Users → Internet → ALB**
  and mention the Internet Gateway in the report instead of drawing it.
  But if your faculty expects more network detail, add it.

---

# 17) Final diagram structure (visual map)

Use this as your exact placement guide:

```text
+------------------------------------------------------+
|                     USERS / INTERNET                 |
|                                                      |
|   [Student / Instructor / Admin]                     |
|                 |                                    |
|              [Internet]                              |
|                 |                                    |
|     [Application Load Balancer - EduStream-ALB]      |
+------------------------------------------------------+

                         |
                         v

+====================================================================================+
|                                   AWS CLOUD                                        |
|                                                                                    |
|  +-------------------------------------------------------------------------------+ |
|  |                               EduStream-VPC                                   | |
|  |                                                                               | |
|  |   ----------------------- PUBLIC SUBNET LAYER ------------------------------  | |
|  |                                                                               | |
|  |   +----------------------+         +----------------------+                   | |
|  |   | Public Subnet 1      |         | Public Subnet 2      |                   | |
|  |   | (AZ-1)               |         | (AZ-2)               |                   | |
|  |   |                      |         |                      |                   | |
|  |   |  EC2 App Instance 1  |         |  EC2 App Instance 2  |                   | |
|  |   |  Flask App           |         |  Flask App           |                   | |
|  |   +----------------------+         +----------------------+                   | |
|  |                                                                               | |
|  |        <------ Auto Scaling Group : EduStream-ASG ------>                    | |
|  |                                                                               | |
|  |   ----------------------- PRIVATE DB SUBNET LAYER --------------------------  | |
|  |                                                                               | |
|  |   +----------------------+         +----------------------+                   | |
|  |   | Private DB Subnet 1  |         | Private DB Subnet 2  |                   | |
|  |   | (AZ-1)               |         | (AZ-2)               |                   | |
|  |   +----------------------+         +----------------------+                   | |
|  |                                                                               | |
|  |                 [Amazon RDS MySQL - edustream-mysql-db]                       | |
|  |                     DB Name: edustream                                        | |
|  |                     DB Subnet Group / Private                                 | |
|  +-------------------------------------------------------------------------------+ |
|                                                                                    |
|      EC2 -----> RDS                                                                |
|      EC2 -----> S3                                                                 |
+====================================================================================+

                          +-----------------------------+
                          |        Amazon S3            |
                          | edustream-course-storage-   |
                          | farhan-2026                 |
                          | courses/notes/              |
                          | assignments/                |
                          +-----------------------------+
                                      |
                                      | S3 Event Notification
                                      v
                          +-----------------------------+
                          |       AWS Lambda            |
                          | EduStream-S3-Processor      |
                          +-----------------------------+
                                      |
                                      v
                          +-----------------------------+
                          |      Amazon CloudWatch      |
                          | Metrics / Alarms / Logs     |
                          +-----------------------------+

Additional IAM blocks:
- EduStream-EC2-Role  -> EC2
- EduStream-Lambda-Role -> Lambda
```

---

# 18) Arrow labels to use in Draw.io

Use these labels on connectors:

## User to ALB

* **HTTP / HTTPS Requests**

## ALB to EC2

* **Load Balanced Traffic**

## EC2 to RDS

* **MySQL (3306)**

## EC2 to S3

* **Upload Course Files / Notes**

## S3 to Lambda

* **ObjectCreated Event**
* **Prefix: courses/notes/**

## Lambda to CloudWatch

* **Execution Logs**

## EC2 / ALB / RDS to CloudWatch

* **Metrics & Alarms**

---

# 19) Color / formatting suggestion

If you want the diagram to look neat:

## Use:

* **Blue** border for VPC
* **Light blue** fill for public subnets
* **Light gray / light purple** fill for private DB subnets
* **Dashed group box** around the Auto Scaling Group
* Keep AWS icons official if available from Draw.io AWS shape library

---

# 20) Minimum icons to include for assignment

If you want a complete but not overcrowded diagram, include these icons only:

* Users
* Internet / optional IGW
* ALB
* 2 EC2 instances
* Auto Scaling Group boundary
* 1 RDS MySQL icon
* 1 S3 bucket icon
* 1 Lambda icon
* 1 CloudWatch icon
* 2 IAM role icons
* VPC boundary
* 4 subnet boxes

That is enough to look professional and aligned with your implementation.

---

# 21) What to write as the diagram title

Use this title above the architecture:

# **EduStream Cloud – AWS Architecture for a Scalable Education Platform**

---

# 22) Recommended subtitle

Below the title, optionally write:

**Services Used: VPC, EC2, ALB, Auto Scaling, RDS MySQL, S3, Lambda, CloudWatch, IAM**

---

If you want, I can do **one more very useful thing**: I can turn this into a **box-by-box “place this icon here” checklist for Draw.io**, so while you build the diagram you can follow it line by line without thinking about layout.

If you want that version, just say **“give me drawio step-by-step placement checklist”**.
