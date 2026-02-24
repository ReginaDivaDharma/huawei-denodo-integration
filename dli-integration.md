# Data Lake Insight Integration
By far this is the only way we can integrate denodo with huawei cloud, since this service is able to have a custom dirver. Please follow my steps below if you need to integrate denodo with Huawei Cloud.
The challenge of using DLI is that when you purchase DLI, it is actually deployed outside of your VPC, and you have to do some networking if we want out DLI to reach the public internet.

**WARNING: Before you start make sure you have access to create agency in IAM. This blocked my progress for a while, so cordinate with your master account** :joy:

## Setting Up Your Network
We start by making our own VPC, this VPC will consists of the networking service that we need to use to access our Denodo with DLI, which is **NAT Gateway**
1. Make an Enterprise project
Before we start to make things simpler it is better if you make an enterprise project so that you can note down your services easily. 
2. Create a simple VPC
As written above you can just create a simple VPC , name it anything
3. Create NAT Gateway
Within your VPC you should purchase a NAT Gateway and bind an EIP to this NAT Gateway. Dont forget to configure your SNAT rule so you can access the internet (i think this will be automatically updated once you click configure SNAT /i forgot) This is how we will connect to denodo!

## Setting Up Data Lake Insight Network
Now we are getting to more complicated part, setting up DLI and connecting it to the NAT Gateway :satisfied:
1. Buy a Resource Pool
A resource pool is necessary to purchase, this will be the place where your queue / job will run. Pick any specification that fits your needs, because im only testing the connectivity i used the basic with 16 CU range to save cost. Make sure that it's in the same enterprise project of your nat gateway / vpc.
2. Buy a DLI package
you also need to buy a DLI package if you want to run a queue or a job, i picked the smallest configuration for testing, please purchase as you needed. 
3. Associate Queue
After you bought two of these things , please go back to the resource pool page again. Now you will need to click on more on your resource pool, there should be a text saying associate queue. You should select the queue you just bought to associate with it.

## Setting Up VPC Peering from DLI to Nat Gateway's VPC
Now if you already have your queue set up then let's move to the more compicated part -networking.
1. Authorize Agency
Before we make a connection it would be better if you can check all the authorization that we need in this integration 
<img width="1352" height="802" alt="image" src="https://github.com/user-attachments/assets/9a954bc3-1f86-45e8-98c8-2ed0c7b4aada" />
In the image above i suggest that you tick all three agency settings to make things easier, after you already checked them all be sure to click update
2. Set Up VPC Peering
Click on the datasource connection, this is where you can do a vpc peering from DLI to our VPC with Nat Gateway
<img width="1892" height="731" alt="image" src="https://github.com/user-attachments/assets/b7fb7e68-a528-4f63-88da-9d1c7eef3bd5" />
Input the information according to your NAT Gateway VPC, and make sure to put your resource pool in the form
3. Configure the Queue Connection to Data Source on the internet > **Do not miss this!**
Reference Link : https://support.huaweicloud.com/eu/bestpractice-dli/dli_05_0061.html
Now here's the tricky part that i struggled with, the queue even though you have connected your resource pool to the NAT Gateway, you would still need to configure a connection from your queue to the internet.
- Configure NAT Gateway SNAT
  Now you've bought right? now we need to go back to the NAT Gateway again, and add another SNAT rule specifically for our queue so our queue can *connect* via the NAT gateway
  <img width="1068" height="606" alt="image" src="https://github.com/user-attachments/assets/ea7339ba-6433-41fe-b536-a23e24e37420" />
  ` * Make sure to choose direct connect / cloud connect`
  ` * Select the subnet where the queue you want to connect locates. `
  ` * Select the target EIP, you can just choose the EIP you just bought and bind to the NAT gateway here.`
  ` * Click OK`
- Adding a Custom route
After you already make a direct connection from the queue's subnet to the NAT , next we need to tell our queue to connect to the denodo IP. Which is why we need to make a custom route. Now let's go back to the data lake insight dashboard after configuring the SNAT rule
<img width="1905" height="471" alt="image" src="https://github.com/user-attachments/assets/4ae98185-c0c2-47d1-af07-a36df18fde0a" />
  ` * click on the manage route `
  ` * Add a custom route for the enhanced datasource connection you have created. Specify the route information of the IP address you want to access. That means let's put the denodo IP address here`
  - Now let's test the connectivity from our queue!
    Test the connectivity between the queue and the public network. Click More > Test Address Connectivity in the Operation column of the target queue and enter the public IP address you want to access.
    <img width="676" height="311" alt="image" src="https://github.com/user-attachments/assets/bea764fd-cec1-4273-9ebf-ea8647f2c1d2" />

  
  


