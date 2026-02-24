# Data Lake Insight (DLI) Integration Guide

Integrating **Denodo** with **Huawei Cloud** via **Data Lake Insight (DLI)** is currently the most effective method, as DLI supports custom drivers. 

> [!CAUTION]
> **Prerequisite:** Ensure you have permissions to create an **Agency in IAM**. This is a common bottleneck; coordinate with your Master Account administrator before proceeding. :joy:

---

## 1. Networking Setup
DLI is deployed outside of your VPC by default. To allow DLI to reach the public internet (and Denodo), we must configure a **NAT Gateway**.

### Create the VPC Environment
* **Enterprise Project:** Create an Enterprise Project first to keep your resources organized and easily trackable.
* **VPC:** Create a standard VPC with a name of your choice.
* **NAT Gateway:** 1. Purchase a NAT Gateway within your VPC.
    2. Bind an **EIP** (Elastic IP) to this Gateway.
    3. Configure the **SNAT rule** to enable outbound internet access.

---

## 2. Setting Up DLI Resources
Next, we prepare the DLI environment where your jobs will actually run.

1. **Purchase a Resource Pool:** - This is the compute environment for your jobs. 
   - *Tip:* For testing, a basic 16 CU range is sufficient to manage costs. Ensure it is in the same Enterprise Project as your VPC.
2. **Purchase a DLI Package:** Select the smallest configuration for initial connectivity testing.
3. **Associate the Queue:** - Go to the **Resource Pool** page.
   - Click **More** > **Associate Queue** and select the queue you purchased.

---

## 3. VPC Peering & Authorization
This step links the DLI infrastructure to your specific VPC.

### Step A: Authorize Agency
Before connecting, ensure the necessary permissions are active. Navigate to DLI settings and verify/update the three Agency settings to ensure seamless integration.

<img width="1352" height="802" alt="Agency Settings" src="https://github.com/user-attachments/assets/9a954bc3-1f86-45e8-98c8-2ed0c7b4aada" />

### Step B: Set Up VPC Peering
1. Click on **Datasource Connection**. This is where you perform VPC peering from DLI to your VPC.
2. Input the information according to your **NAT Gateway VPC**.
3. Ensure you select your **Resource Pool** in the form.

<img width="1892" height="731" alt="VPC Peering Setup" src="https://github.com/user-attachments/assets/b7fb7e68-a528-4f63-88da-9d1c7eef3bd5" />

---

## 4. Final Routing (The "Tricky" Part)
Even with peering, the queue needs specific instructions to reach the internet via the NAT Gateway.

### Configure NAT Gateway SNAT
Return to your **NAT Gateway** settings and add a new **SNAT Rule** specifically for the DLI Queue:
* **Scenario:** Choose *Direct Connect / Cloud Connect*.
* **Subnet:** Select the specific subnet where your DLI queue is located.
* **EIP:** Use the EIP bound to your NAT Gateway.

<img width="1068" height="606" alt="SNAT Configuration" src="https://github.com/user-attachments/assets/ea7339ba-6433-41fe-b536-a23e24e37420" />

### Add a Custom Route
Go back to the **Data Lake Insight Dashboard** after configuring the SNAT rule:
1. Click on **Manage Route**.
2. **Add a Custom Route:** Enter the **Public IP address of your Denodo instance**. This tells the queue exactly where to send traffic.

<img width="1905" height="471" alt="Custom Route Setup" src="https://github.com/user-attachments/assets/4ae98185-a528-4f63-88da-9d1c7eef3bd5" />

---

## 5. Verify Connectivity
Finally, test if the bridge is working:
1. In the DLI Queue list, click **More** > **Test Address Connectivity**.
2. Enter the **Public IP address** you want to access (your Denodo IP).

** Congrats you've passed the networking stage! now lets move to the last part which is making a spark job! 😃** 
4. If the connection is successful, you are ready to go!

<img width="676" height="311" alt="Connectivity Test" src="https://github.com/user-attachments/assets/bea764fd-cec1-4273-9ebf-ea8647f2c1d2" />
