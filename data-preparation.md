# Data Preparation & Denodo Integration

In this section, we will set up a Relational Database (RDS) on Huawei Cloud and connect it to Denodo. 

> [!NOTE]
> If you already have data in Denodo or prefer to use an existing data source, you can skip the RDS setup and go straight to **Connecting with Denodo**.

---

## 1. Data Preparation (PostgreSQL on RDS)

### Step 1: Create an RDS Instance
Create a Relational Database Service (RDS) instance using PostgreSQL or MySQL. Denodo supports all major database engines, so choose the one you are most comfortable with.

### Step 2: Ingest Dummy Data
Log in to your RDS instance using your credentials and open the SQL window. Use a standard `CREATE TABLE` and `INSERT` script to add some sample data. 

*Tip: If your table doesn't appear immediately after creation, try refreshing the database view.*

<img width="1847" height="777" alt="RDS SQL Window" src="https://github.com/user-attachments/assets/159ad4dd-16b0-43f4-8607-54a761b28110" />

### Step 3: Bind a Public IP
To allow Denodo to "see" your database, ensure you bind an **Elastic IP (EIP)** to your RDS instance and configure the security group to allow inbound traffic on the database port.

---

## 2. Connecting with Denodo

> [!IMPORTANT]
> **Network Connectivity:** Ensure your Denodo instance is reachable. In this setup, Denodo is deployed on a VM with a Public IP.

### Step 1: Initialize Denodo
Ensure all necessary Denodo services are running. We will primarily use the **Web Design Studio**. 
* **Port 9999:** Ensure this port is open/active.
* **Credentials:** It is highly recommended to change the default Denodo password before connecting data sources.

<img width="903" height="618" alt="Denodo Control Center" src="https://github.com/user-attachments/assets/18b5c298-6a25-4103-92ce-55e5c89a6df2" />

### Step 2: Configure the Data Source
1. In the **Design Studio**, right-click on the `admin` database (or your preferred database).
2. Select **New** > **Data Source** and choose your database type (e.g., **PostgreSQL**).

<img width="1015" height="718" alt="New Data Source" src="https://github.com/user-attachments/assets/ac0e0ab9-d264-48af-8164-7f482f4f4140" />

3. Fill in the connection details (Host, Port, Database Name, and Credentials).
4. **Test Connection:** Before saving, click **More options** > **Test Connection** to verify the link.

<img width="698" height="465" alt="Connection Details" src="https://github.com/user-attachments/assets/9ff716ce-7017-4b15-9b37-ac6dc22950dc" />

### Step 3: Create a Base View
Once the connection is saved, you need to create a **Base View** to expose the underlying table to Denodo's virtualization layer.
1. Select the tables you wish to import (e.g., the `fruit` table).
2. Click **Create Selected**.

<img width="997" height="493" alt="Create Base View" src="https://github.com/user-attachments/assets/3f0c2558-5bae-4e51-a00c-601613022ba1" />

### Step 4: Verify the Data
Finalize the process by running a quick `SELECT` query in the Design Studio to ensure the data is being retrieved correctly.

<img width="991" height="582" alt="Query Verification" src="https://github.com/user-attachments/assets/1564e672-00be-4d57-aaf9-a3dc3f4f3e8c" />

**Congratulations! You have successfully connected your data source to Denodo and created your first Base View!** :smiley:
