# Data Preparation 
Within this tutorial i chose to deploy Relational Database (PostgreSQL) on huawei clouds and ingesting some data into the database first. But to be honest you can just put any data , and you dont have to deploy an RDS instance if you want to try connecting to denodo. You can skip this part if you already did upload some data in your own denodo :D

1. Make a Relational Data Base Instance (PostgreSQL / MySQL)
During this step the database engine you choose does really matter since denodo supports all types of database.
2. Putting Dummy Data in RDS
The data itself does not have to be alot. So after you created your RDS instance you need to log in using the credentials you inserted when your made your instance. After logging in you should see a SQL window, there you can just use a simple SQL syntax to make a table and insert the data (If the table did not appear at first you probably need to refresh). You can see the image below of what table / data i made
<img width="1847" height="777" alt="image" src="https://github.com/user-attachments/assets/159ad4dd-16b0-43f4-8607-54a761b28110" />
3. Bind Public IP to your RDS Instance
After making your data / table, you should bind your database with a public IP so the denodo server can access this data.

## Connecting with Denodo
**Before we start with Denodo, make sure that there' a way we can connect to denodo. For example, in this set up i put denodo on top of a VM so it can have a public IP that we can connect to.**
1. Starting with Denodo
Before we start connecting our database to denodo, please make sure you've already set up your denodo. After installing denodo proceed normally (FYI im using the free trial so i might have less features than you do), at least i didn't install anything weird or out of box, just follow the installation as intructed. At the end you should see the image below.
<img width="903" height="618" alt="image" src="https://github.com/user-attachments/assets/18b5c298-6a25-4103-92ce-55e5c89a6df2" />
Be sure to turn on the port 9999 , and we will mostly work in the web design tool, so please turn that on too and click on the link for the design studio.\
*I forgot to mention, it will be better that you change your denodo password from the original one , because you need to do this if you want to connect with a datasource if im not wrong*
3. Connecting your database to Denodo
Now depending on the data source that you created, this step will be different. Since i made a PostgreSQL database then you can follow what i did.
- Click on the three dots on the admin database, you should see new > then click data source. After clicking it you should see this page below. And since im using PostgreSQL, i will be clicking on that.
<img width="1015" height="718" alt="image" src="https://github.com/user-attachments/assets/ac0e0ab9-d264-48af-8164-7f482f4f4140" />
Now after you click the postgreSQL data source you should see a form asking for your database information, please insert your info here. 
<img width="698" height="465" alt="image" src="https://github.com/user-attachments/assets/9ff716ce-7017-4b15-9b37-ac6dc22950dc" />
Before you click save, i suggest you test your data source connection first , you can do this by clicking more options > Test Connection
4. Making a Create Base View
After you have your data connection , you should now create a base view. This is the thing we will try to extract from Denodo. Since the table i careated was **fruits** i will be creating a base view for that table.
<img width="997" height="493" alt="image" src="https://github.com/user-attachments/assets/3f0c2558-5bae-4e51-a00c-601613022ba1" />
Before we move on to the next step, i also suggest that you should try to query your data from the base view you just created, just to check if the data is actually there.
<img width="991" height="582" alt="image" src="https://github.com/user-attachments/assets/1564e672-00be-4d57-aaf9-a3dc3f4f3e8c" />

**Congratulations you've successfully connected your data source to denodo and created a base view!** :smiley:






