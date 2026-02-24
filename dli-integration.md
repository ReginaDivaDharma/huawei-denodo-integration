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
Within your VPC you should purchase a NAT Gateway and bind an EIP to this NAT Gateway. This is how we will connect to denodo!

## Setting UP Data Lake Insight 
Now we are getting to more complicated part, setting up DLI and connecting it to the NAT Gateway :satisfied:
1. Buy a Resource Pool
A resource pool is necessary to purchase, this will be the place where your queue / job will run. Pick any specification that fits your needs, because im only testing the connectivity i used the basic with 16 CU range to save cost. Make sure that it's in the same enterprise project of your nat gateway / vpc.
2. Buy a DLI package
you also need to buy a DLI package if you want to run a queue or a job, i picked the smallest configuration for testing, please purchase as you needed. 
3. Associate Queue
After you bought two of these things , please go back to the resource pool page again. Now you will need to click on more on your resource pool, there should be a text saying associate queue. You should select the queue you just bought to associate with it.

## Setting Up VPC Peering from DLI to Nat Gateway's VPC
Now if you already have your queue set up then let's move to the more compicated part -networking.
1.   Authorize VPC peering
Before we make a connection it would be better if you can check all the authorization that we need in this integration 
<img width="1352" height="802" alt="image" src="https://github.com/user-attachments/assets/9a954bc3-1f86-45e8-98c8-2ed0c7b4aada" />
In the image above i suggest that you tick all three agency settings to make things easier, after you already checked them all be sure to click update 
