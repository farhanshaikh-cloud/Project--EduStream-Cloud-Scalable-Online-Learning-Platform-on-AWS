# Phase 1 — VPC Setup (Step-by-step on AWS Console)

## **Target network design**

We’ll build this:

* **VPC CIDR:** `10.0.0.0/16`

### Public Subnets

* **Public-Subnet-1** → `10.0.1.0/24`
* **Public-Subnet-2** → `10.0.2.0/24`

### Private App Subnets

* **Private-App-Subnet-1** → `10.0.11.0/24`
* **Private-App-Subnet-2** → `10.0.12.0/24`

### Private DB Subnets

* **Private-DB-Subnet-1** → `10.0.21.0/24`
* **Private-DB-Subnet-2** → `10.0.22.0/24`

---

# Step 1: Create VPC

Go to **VPC Console** → **Your VPCs** → **Create VPC**

Choose:

* **Resources to create** → **VPC only**
* **Name tag** → `EduStream-VPC`
* **IPv4 CIDR** → `10.0.0.0/16`
* Tenancy → Default

Click **Create VPC**

---

# Step 2: Create 6 Subnets

Go to **VPC Console** → **Subnets** → **Create subnet**

Select **EduStream-VPC**

Create these one by one:

## Public subnets

### 1. Public-Subnet-1

* Name: `EduStream-Public-1`
* AZ: `us-east-1a` *(or your first AZ)*
* CIDR: `10.0.1.0/24`

### 2. Public-Subnet-2

* Name: `EduStream-Public-2`
* AZ: `us-east-1b`
* CIDR: `10.0.2.0/24`

## Private app subnets

### 3. Private-App-Subnet-1

* Name: `EduStream-Private-App-1`
* AZ: `us-east-1a`
* CIDR: `10.0.11.0/24`

### 4. Private-App-Subnet-2

* Name: `EduStream-Private-App-2`
* AZ: `us-east-1b`
* CIDR: `10.0.12.0/24`

## Private DB subnets

### 5. Private-DB-Subnet-1

* Name: `EduStream-Private-DB-1`
* AZ: `us-east-1a`
* CIDR: `10.0.21.0/24`

### 6. Private-DB-Subnet-2

* Name: `EduStream-Private-DB-2`
* AZ: `us-east-1b`
* CIDR: `10.0.22.0/24`

---

# Step 3: Enable auto-assign public IP only for public subnets

Go to **Subnets** → select `EduStream-Public-1` → **Actions** → **Edit subnet settings**

* Enable **Auto-assign public IPv4 address**

Do the same for:

* `EduStream-Public-2`

Do **not** enable this for private subnets.

---

# Step 4: Create Internet Gateway

Go to **Internet Gateways** → **Create internet gateway**

* Name: `EduStream-IGW`

Create it, then:

* select it
* click **Actions → Attach to VPC**
* attach to `EduStream-VPC`

---

# Step 5: Create Elastic IP for NAT Gateway

Go to **EC2 Console** or **VPC → Elastic IPs**

* Click **Allocate Elastic IP address**
* Keep default
* Allocate

---

# Step 6: Create NAT Gateway

Go to **VPC → NAT Gateways** → **Create NAT Gateway**

Enter:

* **Name**: `EduStream-NAT`
* **Subnet**: `EduStream-Public-1`
* **Elastic IP**: select the EIP you just created

Create NAT Gateway.

> NAT is needed so instances in private subnets can access the internet for package installs/updates without being publicly exposed.

---

# Step 7: Create Route Tables

## A. Public Route Table

Go to **Route Tables** → **Create route table**

* Name: `EduStream-Public-RT`
* VPC: `EduStream-VPC`

After creating:

### Add route

Edit routes:

* Destination: `0.0.0.0/0`
* Target: **Internet Gateway**
* Select `EduStream-IGW`

### Associate public subnets

Go to **Subnet Associations** and associate:

* `EduStream-Public-1`
* `EduStream-Public-2`

---

## B. Private App Route Table

Create another route table:

* Name: `EduStream-Private-App-RT`

Add route:

* Destination: `0.0.0.0/0`
* Target: **NAT Gateway**
* Select `EduStream-NAT`

Associate:

* `EduStream-Private-App-1`
* `EduStream-Private-App-2`

---

## C. Private DB Route Table

Create another route table:

* Name: `EduStream-Private-DB-RT`

For DB subnet route table, keep it **private**.
Usually no internet route is needed.

Associate:

* `EduStream-Private-DB-1`
* `EduStream-Private-DB-2`

---

# At the end of Phase 1, your network should look like this

## **VPC**

`10.0.0.0/16`

## **Public**

* `10.0.1.0/24`
* `10.0.2.0/24`
* Internet Gateway route

## **Private App**

* `10.0.11.0/24`
* `10.0.12.0/24`
* NAT Gateway route

## **Private DB**

* `10.0.21.0/24`
* `10.0.22.0/24`
* no direct internet route

---

# What I need from you

If you want, we can build this **together live**.

## Reply with:

### **“Phase 1 done”**
