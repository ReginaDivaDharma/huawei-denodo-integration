# 🌊 Connecting Denodo to Huawei Cloud DLI — A Beginner's Guide

> This guide walks you through connecting **Denodo** (a data virtualization tool) to **Huawei Cloud Data Lake Insight (DLI)** step by step. No prior experience needed — every term is explained along the way!

---

## 📋 Table of Contents
1. [What Are We Trying to Do?](#what-are-we-trying-to-do)
2. [Before You Start](#before-you-start)
3. [Step 1 — Networking Setup](#step-1--networking-setup)
4. [Step 2 — Setting Up DLI Resources](#step-2--setting-up-dli-resources)
5. [Step 3 — VPC Peering & Authorization](#step-3--vpc-peering--authorization)
6. [Step 4 — Final Routing (The Tricky Part)](#step-4--final-routing-the-tricky-part)
7. [Step 5 — Verify Connectivity](#step-5--verify-connectivity)
8. [Step 6 — Create an OBS Bucket](#step-6--create-an-obs-bucket)
9. [Step 7 — Upload the Denodo JDBC Driver](#step-7--upload-the-denodo-jdbc-driver)
10. [Step 8 — Write a PySpark Job to Query Denodo](#step-8--write-a-pyspark-job-to-query-denodo)
11. [Step 9 — Create an IAM Agency to Access OBS](#step-9--create-an-iam-agency-to-access-obs)
12. [Step 10 — Submit the Spark Job](#step-10--submit-the-spark-job)
13. [Step 11 — See the Result!](#step-11--see-the-result)
14. [Step 12 — Connecting a DLI Table Back into Denodo](#step-12--connecting-a-dli-table-back-into-denodo)
15. [🚀 Modifying DLI Tables via Denodo (CRUD Guide)](#-modifying-dli-tables-via-denodo-crud-guide)

---

## What Are We Trying to Do?

Before jumping in, here's a plain-English explanation of the goal:

```
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│   DENODO    │ ◀──────▶ │  Huawei DLI │ ◀──────▶ │  OBS        │
│ (Your query │          │ (Processing │          │ (Your data  │
│  interface) │          │   engine)   │          │  storage)   │
└─────────────┘          └─────────────┘          └─────────────┘
```

- **Denodo** is the tool you use to query data — think of it as a smart window into your data
- **DLI (Data Lake Insight)** is Huawei Cloud's data processing engine — it runs your queries
- **OBS (Object Storage Service)** is where your actual data files live — like a cloud hard drive

**The goal:** Make Denodo and DLI talk to each other so you can query your data from one place.

---

## Before You Start

> ⚠️ **Important — Read This First!**
> 
> Make sure you have permission to create an **Agency in IAM** (Identity and Access Management). If you don't have this, coordinate with your Master Account administrator **before** starting — skipping this will waste a lot of time later!

**What you'll need:**
- A Huawei Cloud account with DLI and OBS enabled
- A running Denodo instance with a public IP address
- Admin or sufficient IAM permissions
- Basic familiarity with cloud consoles (no coding required until Step 8)

---

## Step 1 — Networking Setup

### 🤔 Why Do We Need This?

By default, DLI lives in its own isolated network — it can't reach the outside internet (or your Denodo instance) on its own. We need to build a "bridge" using a **NAT Gateway**.

Think of it like this: DLI is in a room with no windows. The NAT Gateway is the window we're adding so it can see outside.

### What To Do

**A. Create an Enterprise Project**

An Enterprise Project is just a folder to keep all your related resources organized and easy to track. Create one first before anything else.

**B. Create a VPC (Virtual Private Cloud)**

A VPC is your own private network on Huawei Cloud. Create a standard VPC with any name you like.

**C. Set Up a NAT Gateway**

A NAT Gateway is what lets DLI access the internet. Here's how:

1. Purchase a **NAT Gateway** inside your VPC
2. Bind an **EIP (Elastic IP)** to the gateway — this is the public IP address DLI will use to reach the internet
3. Add an **SNAT Rule** to enable outbound internet traffic

> 💡 **Plain English:** EIP = a fixed public address. SNAT Rule = permission slip that says "this network is allowed to go to the internet".

---

## Step 2 — Setting Up DLI Resources

Now we set up the actual compute environment where your jobs will run.

### A. Purchase a Resource Pool

A Resource Pool is the computing power DLI uses to run your jobs.

- For testing: **16 CU** is enough and keeps costs low
- Make sure it's in the **same Enterprise Project** as your VPC from Step 1

### B. Purchase a DLI Queue

A Queue is a dedicated lane for running your queries. Start with the **smallest configuration** for initial testing.

### C. Associate the Queue to the Resource Pool

1. Go to the **Resource Pool** page
2. Click **More** → **Associate Queue**
3. Select the queue you just purchased

> 💡 **Why:** The Resource Pool provides the horsepower. The Queue is what you'll actually submit jobs to. They need to be linked together.

---

## Step 3 — VPC Peering & Authorization

This step connects DLI's network to your VPC so they can communicate.

### A. Set Up Agency Authorization

An **Agency** is basically a permission slip that lets one Huawei service act on behalf of another.

1. Go to **DLI Settings**
2. Find the **Agency** section
3. Make sure all three agency settings are authorized

![Agency Settings](https://github.com/user-attachments/assets/9a954bc3-1f86-45e8-98c8-2ed0c7b4aada)

### B. Set Up VPC Peering

VPC Peering creates a direct private connection between DLI's network and your VPC.

1. Go to **Datasource Connection** in DLI
2. Click to create a new connection
3. Fill in your **NAT Gateway VPC details**
4. Make sure you select your **Resource Pool** in the form

![VPC Peering Setup](https://github.com/user-attachments/assets/b7fb7e68-a528-4f63-88da-9d1c7eef3bd5)

---

## Step 4 — Final Routing (The Tricky Part)

> 📖 **Official Huawei Docs for this step:**  
> https://support.huaweicloud.com/eu/bestpractice-dli/dli_05_0061.html

Even with VPC Peering done, the DLI Queue doesn't automatically know how to reach the internet. We need to give it specific directions — like a GPS address.

### A. Add a Second SNAT Rule for the DLI Queue

Go back to your **NAT Gateway** and add a **new SNAT Rule** specifically for your DLI Queue subnet:

| Field | What to Put |
|---|---|
| **Scenario** | Direct Connect / Cloud Connect |
| **Subnet** | The subnet where your DLI queue lives |
| **EIP** | The EIP bound to your NAT Gateway |

![SNAT Configuration](https://github.com/user-attachments/assets/ea7339ba-6433-41fe-b536-a23e24e37420)

### B. Add a Custom Route

This tells the DLI Queue exactly where Denodo lives.

1. Go back to **DLI Dashboard**
2. Click **Manage Route**
3. Click **Add Custom Route**
4. Enter the **Public IP address of your Denodo instance**

> 💡 **Why:** Without this route, DLI knows it can access the internet, but doesn't know where Denodo is specifically. This step is like entering a destination into GPS.

---

## Step 5 — Verify Connectivity

Let's make sure everything is working before moving on!

1. In the DLI Queue list, click **More** → **Test Address Connectivity**
2. Enter the **Public IP address of your Denodo instance**
3. ✅ If it says connected — you're ready to go!

![Connectivity Test](https://github.com/user-attachments/assets/bea764fd-cec1-4273-9ebf-ea8647f2c1d2)

> 🎉 **Congrats! You've completed the networking stage! The hard part is done.**

---

## Step 6 — Create an OBS Bucket

OBS (Object Storage Service) is Huawei Cloud's file storage — think of it like Google Drive but for cloud applications.

We need an OBS bucket to store:
- Your Spark job script (the code that runs your query)
- The Denodo JDBC driver (a file that lets DLI talk to Denodo)
- Job logs

**How to create it:**
1. Go to the OBS console
2. Click **Create Bucket**
3. Choose **Standard** storage type (recommended if you'll access it frequently)
4. Note down the bucket path — you'll need it in later steps

![OBS Bucket](https://github.com/user-attachments/assets/349188f4-868b-47b0-9b27-54f141e3bd33)

---

## Step 7 — Upload the Denodo JDBC Driver

### 🤔 What Is a JDBC Driver?

A **JDBC Driver** is a small file (`.jar`) that acts as a translator between two systems. Without it, DLI wouldn't know how to speak Denodo's language.

### How to Upload It

**1. Download the driver**

Go to the Denodo community portal and download the JDBC driver:
👉 https://community.denodo.com/drivers/jdbc/9

**2. Upload to OBS**

Upload the `.jar` file to your OBS bucket (keep it in a clearly named folder like `/drivers/`).

![Driver in OBS](https://github.com/user-attachments/assets/933f219b-ac54-42ef-80d0-066842da53ee)

**3. Register it in DLI**

1. Go to **DLI Dashboard** → **Package Management**
2. Click **Create Package**
3. Set the path to your `.jar` file in OBS
4. Set **Type** to `JAR`
5. For **Group**, select **Do not use**

![Package Management](https://github.com/user-attachments/assets/024a4873-8ce9-4e03-baaa-9812b9c5393c)

---

## Step 8 — Write a PySpark Job to Query Denodo

### 🤔 What Is PySpark?

**PySpark** is Python code that runs on Apache Spark — a powerful data processing engine. DLI uses Spark under the hood, so we write a small Python script to tell it what to do.

### The Script

Create a new file called `query_denodo.py` with this content:

```python
from pyspark.sql import SparkSession

# Start a Spark session (the entry point for all Spark jobs)
spark = SparkSession.builder \
    .appName("denodo_query") \
    .getOrCreate()

# Connect to Denodo and read a table
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:denodo://YOUR_DENODO_IP:9999/admin") \  # ← Replace with your Denodo IP
    .option("dbtable", "your_table_name") \                       # ← Replace with your table name
    .option("user", "your_username") \                            # ← Replace with your username
    .option("password", "your_password") \                        # ← Replace with your password
    .option("driver", "com.denodo.vdp.jdbc.Driver") \
    .load()

# Print the result
df.show()

spark.stop()
```

> ⚠️ **Note:** You'll need one script per table you want to query. Just copy this template and change the `dbtable` value each time.

**Upload the script to OBS** once it's ready (a dedicated `/jobs/` folder is recommended).

---

## Step 9 — Create an IAM Agency to Access OBS

### 🤔 Why Do We Need This?

DLI needs explicit permission to read files from your OBS bucket. Without this, the Spark job will fail because it can't access the driver file or the script.

### Step A — Get Your AK/SK Credentials

**AK (Access Key)** and **SK (Secret Key)** are like a username and password for Huawei Cloud APIs. You'll need these to authenticate.

Find them in: **My Account** → **Access Keys**

### Step B — Store Your AK/SK Securely in DEW

**DEW (Data Encryption Workshop)** is Huawei's secure secret storage — like a vault for sensitive credentials.

1. Go to the **DEW Console** → **Cloud Secret Management Service** → **Secrets**
2. Click **Create Secret**
3. Add two key-value pairs:
   - Key 1: Your Access Key (AK)
   - Key 2: Your Secret Access Key (SK)

![DEW Secret](https://github.com/user-attachments/assets/2704005e-3fdd-4bc4-aa1d-9e3856e30bc3)

### Step C — Create the IAM Agency

1. Go to **IAM Console** → **Agencies** → **Create Agency**
2. Fill in:

| Field | Value |
|---|---|
| **Agency Name** | `dli_dew_agency_access` (or any name you like) |
| **Agency Type** | Cloud service |
| **Cloud Service** | Data Lake Insight (DLI) |
| **Validity Period** | Unlimited |

3. Click **Next** → go to the **Permissions** tab → click **Authorize** → **Create Policy**
4. Set **Format** to JSON and paste this:

```json
{
    "Version": "1.1",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "csms:secretVersion:get",
                "csms:secretVersion:list",
                "kms:dek:decrypt"
            ]
        }
    ]
}
```

5. Click **Next** → On the **Select Scope** page, search for **OBS** and check all the highlighted permissions

![OBS Permissions 1](https://github.com/user-attachments/assets/460501fc-2852-41b0-89bd-5919640c2823)
![OBS Permissions 2](https://github.com/user-attachments/assets/95891811-5dec-4240-b4cf-aea5a61f1576)

6. Click **OK**

> ⏳ **Note:** It may take **15–30 minutes** for the permissions to take effect. Grab a coffee! ☕

---

## Step 10 — Submit the Spark Job

Time to put it all together and actually run the job!

1. Go to **DLI Console** → **Job Management** → **Spark Jobs** → **Create Job**
2. Fill in the job configuration:

| Field | Value |
|---|---|
| **Queue** | The queue you created |
| **Spark Version** | `3.3.1` (required for DEW agent support) |
| **Application** | Your `.py` script path in OBS |
| **Agency** | The IAM agency you created in Step 9 |
| **Jar Dependencies** | Your Denodo JDBC `.jar` file path in OBS |

> ⚠️ **Important Note:** When adding the Jar dependency, **type the OBS path manually — do NOT use the dropdown**. There's a known issue where the dropdown causes an error even though it looks like it should work.

![Job Configuration](https://github.com/user-attachments/assets/afdcafc6-bd13-49e4-9762-955981150b7c)

3. Click **Execute** in the top-right corner
4. Accept the privacy agreement and click **OK**

---

## Step 11 — See the Result! 🎉

Once the job finishes:

1. Click on your Spark job
2. Click the **More** dropdown
3. Select **Driver Log**
4. Scroll down — you should see your table data printed out!

![Result](https://github.com/user-attachments/assets/a916e50a-1b3b-464c-af4b-fc60a89de79c)

> 🥳 **You did it! Your Denodo data is now flowing through DLI!**

---

## Step 12 — Connecting a DLI Table Back into Denodo

Now let's go the other direction — taking a DLI table and making it queryable from Denodo.

### A. Get Your AK/SK Credentials

You'll need your **Access Key (AK)** and **Secret Key (SK)** again (same as Step 9A).

### B. Construct Your DLI JDBC Connection String

Denodo connects to DLI via JDBC. Use this template to build your connection URL:

```
jdbc:dli://{dli-endpoint}/{project-id}?regionname={region};authenticationmode=aksk;databasename={database};queuename={queue}
```

**Real example:**
```
jdbc:dli://dli.ap-southeast-4.myhuaweicloud.com/abc123?regionname=ap-southeast-4;authenticationmode=aksk;databasename=dummy_data;queuename=queue_spark
```

> 💡 **Not sure which endpoint to use?** Check the full list here:  
> https://console-intl.huaweicloud.com/apiexplorer/#/endpoint

### C. Install the DLI JDBC Driver in Denodo

**1. Import the driver**

Inside Denodo Design Studio:
1. Go to **File** → **Extension Management**
2. Click **Library** → **Import**
3. Set **Resource Type** to `JDBC_other`
4. Give it a name and click **OK**

**2. Configure the driver class**

Go to **Configuration** → **Advanced** and set the **Driver Class** as shown:

![Denodo Driver Class Configuration](https://github.com/user-attachments/assets/5a544a6f-f59e-4c5f-ac20-dd1c352adefe)

**3. Create a Base View**

1. Go to your new DLI data source in Denodo
2. Click **Create Base View**
3. Select your database and table
4. Click to create the view — your DLI table is now queryable from Denodo! 🎉

---

## 🚀 Step 13 — Modifying DLI Tables via Denodo (CRUD Guide)

If you don't want to switch between consoles, you can manage your data directly through Denodo. This section explains how to perform **CRUD** (Create, Read, Update, Delete) operations from Denodo back to Huawei Cloud DLI.

---

### 🛠️ The Prerequisites

To modify data, your DLI table must support **ACID transactions**.

- **Required Format:** Use **Hudi** (highly recommended for DLI)
- **The `_rt` Rule:** Hudi creates two table versions. Always use the **Real-Time (`_rt`)** table (e.g., `food_rt`) in Denodo to see your updates instantly.

> 💡 **Pro Tip: "Lao Da" Bypass**  
> DLI requires a partition filter to show data. To see the **entire table** at once, use:
>
> ```sql
> WHERE your_partition_column != ''
> ```
>
> Example:
>
> ```sql
> WHERE city != ''
> ```

---

### 1. Insertion (Create)

Adding new data is as simple as a standard SQL query.

**VQL Command:**

```sql
INSERT INTO bv_food_rt (id, name, category, price, city) 
VALUES (3, 'Pisang Goreng', 'Snack', 20000.0, 'Jakarta');
```

**Verification:**  
Checking both DLI and Denodo confirms the data is successfully ingested.

![Insert Verification](https://github.com/user-attachments/assets/0b2a7dcc-9937-4b00-88fb-7689509b94c4)

---

### 2. Update

When updating, always include the **Partition Key** (e.g., `city`) so DLI knows exactly where to look.

**VQL Command:**

```sql
UPDATE bv_food_rt 
SET price = 45000.0 
WHERE id = 1 AND city = 'Jakarta';
```

**Verification:**  
The price update is immediately reflected in the DLI console.

![Update Verification](https://github.com/user-attachments/assets/29a1cfb1-a5aa-480b-a9f2-ae5af8d0024e)

---

### 3. Delete

Removing a specific row also requires the **Partition Key** to satisfy DLI's safety rules.

**VQL Command:**

```sql
DELETE FROM bv_food_rt 
WHERE id = 10 AND city = 'Bali';
```

**Verification:**  
After running the query, the row is permanently removed from the data lake.

![Delete Verification](https://github.com/user-attachments/assets/b3cd1e2c-66a2-4a3b-a572-dc6affdcc4d4)

---

## ✅ Final Result

The data in Denodo now perfectly aligns with your DLI table.

![Final Result](https://github.com/user-attachments/assets/0dce9f13-2c84-4090-ae5b-1b4dd2f7f82e)

---

*Last updated: April 2026*

*Questions? Reach out: regina.diva333@gmail.com* 😄
