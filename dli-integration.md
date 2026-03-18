# Data Lake Insight (DLI) Integration Guide

Integrating **Denodo** with **Huawei Cloud** via **Data Lake Insight (DLI)** is currently the most effective method, as DLI supports custom drivers.

> [!CAUTION]
> **Prerequisite:** Make sure you have permissions to create an **Agency in IAM**. Coordinate with your Master Account administrator before proceeding, or else you will waste alot of time 

---

## Table of Contents
1. [Networking Setup](#1-networking-setup)
2. [Setting Up DLI Resources](#2-setting-up-dli-resources)
3. [VPC Peering & Authorization](#3-vpc-peering--authorization)
4. [Final Routing (The "Tricky" Part)](#4-final-routing-the-tricky-part)
5. [Verify Connectivity](#5-verify-connectivity)
6. [Create an OBS Bucket](#6-create-an-obs-bucket)
7. [Upload the Denodo JDBC Driver](#7-upload-the-denodo-jdbc-driver)
8. [Write a PySpark Job to Query Denodo](#8-write-a-pyspark-job-to-query-denodo)
9. [Create an IAM Agency to Access OBS](#9-create-an-iam-agency-to-access-obs)
10. [Submit the Spark Job](#10-submit-the-spark-job)
11. [See the Result!](#11-see-the-result)
12. [Pulling DLI Data into Denodo] 

---

## 1. Networking Setup

DLI is deployed outside of your VPC by default. To allow DLI to reach the public internet (and Denodo), you must configure a **NAT Gateway**.

### Create the VPC Environment

- **Enterprise Project:** Create an Enterprise Project first to keep your resources organized and easily trackable.
- **VPC:** Create a standard VPC with a name of your choice.
- **NAT Gateway:**
  1. Purchase a NAT Gateway within your VPC.
  2. Bind an **EIP** (Elastic IP) to this Gateway.
  3. Configure the **SNAT rule** to enable outbound internet access.

---

## 2. Setting Up DLI Resources

Next, prepare the DLI environment where your jobs will run.

1. **Purchase a Resource Pool**
   - This is the compute environment for your jobs.
   - *Tip:* For testing, a basic 16 CU range is sufficient to manage costs. Ensure it is in the same Enterprise Project as your VPC.

2. **Purchase a DLI Package**
   - Select the smallest configuration for initial connectivity testing.

3. **Associate the Queue**
   - Go to the **Resource Pool** page.
   - Click **More** > **Associate Queue** and select the queue you purchased.

---

## 3. VPC Peering & Authorization

This step links the DLI infrastructure to your specific VPC.

### Step A: Authorize Agency

Before connecting, ensure the necessary permissions are active. Navigate to DLI settings and verify/update the three Agency settings to ensure seamless integration.

![Agency Settings](https://github.com/user-attachments/assets/9a954bc3-1f86-45e8-98c8-2ed0c7b4aada)

### Step B: Set Up VPC Peering

1. Click on **Datasource Connection** this is where you perform VPC peering from DLI to your VPC.
2. Input the information according to your **NAT Gateway VPC**.
3. Ensure you select your **Resource Pool** in the form.

![VPC Peering Setup](https://github.com/user-attachments/assets/b7fb7e68-a528-4f63-88da-9d1c7eef3bd5)

---

## 4. Final Routing (The "Tricky" Part)
For official documentation from Huawei Cloud You can see here
🍀https://support.huaweicloud.com/eu/bestpractice-dli/dli_05_0061.html

Even with peering active, the queue needs specific instructions to reach the internet via the NAT Gateway.

### Configure NAT Gateway SNAT

Return to your **NAT Gateway** settings and add a new **SNAT Rule** specifically for the DLI Queue:

| Field | Value |
|---|---|
| **Scenario** | Direct Connect / Cloud Connect |
| **Subnet** | The specific subnet where your DLI queue is located |
| **EIP** | The EIP bound to your NAT Gateway |

![SNAT Configuration](https://github.com/user-attachments/assets/ea7339ba-6433-41fe-b536-a23e24e37420)

### Add a Custom Route

Go back to the **Data Lake Insight Dashboard** after configuring the SNAT rule:

1. Click on **Manage Route**.
2. Click **Add a Custom Route** and enter the **Public IP address of your Denodo instance**. This tells the queue exactly where to send traffic.

---

## 5. Verify Connectivity

Test if the bridge is working:

1. In the DLI Queue list, click **More** > **Test Address Connectivity**.
2. Enter the **Public IP address** of your Denodo instance.
3. If the connection is successful, you are ready to go! 🎉

![Connectivity Test](https://github.com/user-attachments/assets/bea764fd-cec1-4273-9ebf-ea8647f2c1d2)

**Congrats! you've passed the networking stage! Now let's move on to the Spark job setup. 😃**

---

## 6. Create an OBS Bucket

Create an OBS bucket to store your Spark job script, Denodo JDBC driver, and logs. The **Standard** storage type is recommended if you plan to access this bucket frequently.

![OBS Bucket](https://github.com/user-attachments/assets/349188f4-868b-47b0-9b27-54f141e3bd33)

> **Note:** Make sure you remember the path where you store your driver and Spark job files.

---

## 7. Upload the Denodo JDBC Driver

1. Download the Denodo JDBC driver from the community portal:
   👉 https://community.denodo.com/drivers/jdbc/9
   The file will be a `.jar` file.

2. Upload the `.jar` file to your OBS bucket.

   ![Driver in OBS](https://github.com/user-attachments/assets/933f219b-ac54-42ef-80d0-066842da53ee)

3. Head back to the **DLI Dashboard** and go to **Package Management**.
   - Specify the path to the `.jar` file in OBS.
   - Set the type to **JAR**.
   - For the group, select **Do not use** — it's not required.

   ![Package Management](https://github.com/user-attachments/assets/024a4873-8ce9-4e03-baaa-9812b9c5393c)

Your Denodo JDBC driver is now uploaded and registered in DLI!

---

## 8. Write a PySpark Job to Query Denodo

Create a PySpark script to fetch data from Denodo. Here's an example script:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("denodo_test") \
    .getOrCreate()

df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:denodo://xxx.xxx.xxx.xxx:9999/admin") \
    .option("dbtable", "fruit") \  # Replace with your table name
    .option("user", "****") \
    .option("password", "****") \
    .option("driver", "com.denodo.vdp.jdbc.Driver") \
    .load()

df.show()

spark.stop()
```

> **Note:** You will need a separate script for each table you want to ingest.

Once your script is ready:
- Save it as a `.py` file.
- Upload it to your OBS bucket (a dedicated folder for jobs is recommended for easier management).

---

## 9. Create an IAM Agency to Access OBS

Your DLI queue needs explicit permission to access OBS. This is done by creating an **IAM Agency**.

For official reference, see:
👉 https://support.huaweicloud.com/intl/en-us/qs-dli/dli_13_0003.html

### Step 1: Get Your AK/SK

To access OBS from DLI, you will need your account's **Access Key (AK)** and **Secret Key (SK)**.

### Step 2: Purchase DEW and Store Your AK/SK

1. Log in to the **DEW management console**.
2. In the navigation pane, choose **Cloud Secret Management Service** > **Secrets**.
3. Click **Create Secret** and configure the following key-value pairs:
   - **Line 1 key:** Your Access Key ID (AK)
   - **Line 2 key:** Your Secret Access Key (SK)

   ![DEW Secret](https://github.com/user-attachments/assets/2704005e-3fdd-4bc4-aa1d-9e3856e30bc3)

### Step 3: Create the IAM Agency

1. In the IAM console, go to **Agencies** and click **Create Agency**.
2. Fill in the following parameters:

   | Field | Value |
   |---|---|
   | **Agency Name** | e.g., `dli_dew_agency_access` |
   | **Agency Type** | Cloud service |
   | **Cloud Service** | Data Lake Insight (DLI) |
   | **Validity Period** | Unlimited |
   | **Description** | Agency with OBS OperateAccess permissions *(optional)* |

3. Click **Next**, then open the agency and go to the **Permissions** tab.
4. Click **Authorize** > **Create Policy** and configure the following:

   - **Policy Name:** e.g., `dli-dew-agency`
   - **Format:** JSON
   - **Policy Content:**

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

5. Click **Next**, select the custom policy you just created, then click **Next** again.
6. On the **Select Scope** page, search for **OBS** and check all the highlighted permissions shown below:

   ![OBS Permissions 1](https://github.com/user-attachments/assets/460501fc-2852-41b0-89bd-5919640c2823)
   ![OBS Permissions 2](https://github.com/user-attachments/assets/95891811-5dec-4240-b4cf-aea5a61f1576)

7. Click **OK**.

> **Note:** It may take **15–30 minutes** for the authorization to take effect.

Your DLI queue now has access to OBS! 🥳

---

## 10. Submit the Spark Job

You're almost there! Now let's submit the Spark job to pull data from Denodo.

1. On the DLI console, go to **Job Management** > **Spark Jobs** and click **Create Job**.
2. Configure the job with the following parameters:

   | Field | Value |
   |---|---|
   | **Queue** | The queue you created |
   | **Spark Version** | `3.3.1` *(required for DEW agent support)* |
   | **Application** | Your PySpark `.py` script from OBS |
   | **Agency** | The IAM agency you created |
   | **Jar Dependencies** | The Denodo JDBC `.jar` file |

Note: For some reason when i re-tested with the fruits table, the driver only works if you put it here. **Do NOT use the drop down**. Im not sure why there's an error if you use that method.

   <img width="606" height="472" alt="image" src="https://github.com/user-attachments/assets/afdcafc6-bd13-49e4-9762-955981150b7c" />


4. Click **Execute** in the upper right corner, accept the privacy agreement, and click **OK** to submit the job.

---

## 11. See the Result!

Once the job completes successfully, your Denodo data will be available in DLI. Check the job logs to verify the output. 🎉
You can check this by clicking on your spark job > More drop down > select driver log then scroll down if you can see your result. 

<img width="463" height="176" alt="image" src="https://github.com/user-attachments/assets/a916e50a-1b3b-464c-af4b-fc60a89de79c" />

Yay! we got the fruits table!

## 12. Connecting a DLI Table in Denodo

Once you have your table in Data Lake Insight (DLI), you may want to bring it back into Denodo for further use. This tutorial walks you through exactly how to do that!

---

### A. Get Your AK/SK Credentials

Before anything else, you'll need to retrieve your **Access Key (AK)** and **Secret Key (SK)** from your Huawei Cloud account. These credentials allow Denodo to authenticate and access your DLI table.

---

### B. Write Down Your DLI JDBC Endpoint

Denodo connects to DLI via a **JDBC connection**. Use the template below to construct your endpoint:
```
jdbc:dli://dli.dli.{huawei-region-endpoint}/{project-id}?regionname={region-code};authenticationmode=aksk;databasename={databasename};queuename={queuename}
```

> 💡 **Tip:** Not sure which DLI endpoint to use? Check the full list of Huawei Cloud endpoints here:  
> [https://console-intl.huaweicloud.com/apiexplorer/#/endpoint](https://console-intl.huaweicloud.com/apiexplorer/#/endpoint)

---

### C. Install the Custom DLI JDBC Driver in Denodo

Once your endpoint is ready, you need to install the DLI JDBC driver into Denodo.

**1. Download the driver**  
Download the DLI JDBC driver file from the link provided *(still trying to make the download link lol)*.

**2. Import the driver into Denodo**  
Follow these steps inside Denodo:

1. Go to **File** → **Extension Management**
2. Click on **Library**
3. Click the **Import** button
4. Set the **Resource Type** to `JDBC_other`
5. Give it a name of your choice
6. Click **OK** and return to the **Connection** tab

**3. Configure the driver class**  
Navigate to **Configuration** → **Advanced** → set the **Driver Class** as shown in the screenshot below.

![Denodo Driver Class Configuration](https://github.com/user-attachments/assets/5a544a6f-f59e-4c5f-ac20-dd1c352adefe)

---

Once everything is configured, test your connection to verify that Denodo can successfully reach your DLI table. Good luck! 


**I hope that this guide can save you some time 😄, if you have any questions you can always email me at : regina.diva333@gmail.com**
