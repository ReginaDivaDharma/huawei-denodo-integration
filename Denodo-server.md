# Denodo Server Set Up
I suggest that doing an integration with denodo , you should have a public IP that our DLI can access. Which is why i decided to put denodo on top of a VM so they can have a public IP.

## Setting up a VM
Since im using Huawei Cloud you can see what i did below 
1. Purchase an ECS with EIP
Any ECS would do , and since huawei cloud does not support windows OS (it needs to be whitelisted) i used a simple linux os. Dont forget to bind this ECS with a public IP so you can access the internet, and vice versa
2. Denodo Setup
After purchaing the ECS i usually like to install a visualization tool for VNC so im not just looking at.. lines. You can treat the ECS like your own personal laptop after this step, just go to a pre-existing browser and download denodo. Denodo itself supprts 3 OS if im not mistaken, 
windows, linux, and MacOS. Follow the instruction to install it.
3. Security Group
In your security group setting, make sure that your ECS allows inbound for port 9999 , since it is the port that denodo is using. And make sure that you can reach your IP using the IP:Port telnet command, if this can run, then you're doing great.

## Conclusion
In the end, just make sure that your denodo is accessible whether you're using a public IP or other sources. The Data Lake Insight would need to have access to the port 9999 especially.
