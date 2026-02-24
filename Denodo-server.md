# Denodo Server Set Up

I suggest that when doing an integration with Denodo, you should have a public IP that our DLI can access. This is why I decided to put Denodo on top of a VM so it can have a public IP.

---

## Setting up a VM
Since I'm using **Huawei Cloud**, you can see the steps I took below:

1. **Purchase an ECS with EIP**
   - Any ECS would do. Since Huawei Cloud does not support Windows OS by default (it needs to be whitelisted), I used a simple **Linux OS**. 
   - Don't forget to bind this ECS with a **Public IP (EIP)** so you can access the internet, and vice versa.

2. **Denodo Setup**
   - After purchasing the ECS, I usually like to install a visualization tool (VNC) so I'm not just looking at command lines. 
   - You can treat the ECS like your own personal laptop after this step; just go to a browser and download Denodo. 
   - Denodo supports three operating systems: **Windows, Linux, and MacOS**. Follow the instructions provided to install it.

3. **Security Group**
   - In your Security Group settings, make sure your ECS allows **inbound traffic for port 9999**, as this is the port Denodo uses. 
   - Verify connectivity by running a telnet command: `telnet [Your-IP] 9999`. If it connects, you're doing great!

---

## Conclusion
In the end, just make sure that your Denodo instance is accessible, whether you're using a public IP or other sources. The **Data Lake Insight (DLI)** will specifically need access to **port 9999**! 😆

Once you've gotten yourself the [data (base view) in denodo](data-preparation.md) , and made sure that denodo is accessible to the public, we can then move on to our integration with DLI 🤓

Please refer to this page [DLI Integration](dli-integration.md)
