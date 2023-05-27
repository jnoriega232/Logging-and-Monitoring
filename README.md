# Logging and Monitoring

## Objectives for the next 7 Labs, which will help us build our Logging and Monitoring System:

- Overview of Azure Logging at Different Layers (Tenant, Activity, Resource)
- Geo IP Data Ingestion & Log Analytics & Microsoft Sentinel Setup
- Enable MDC & Configure Log Collection for VMs
- KQL Deep Dive
- Tenant Level Logging
- Subscription Level Logging
- Resource Level Logging

### Environments and Technologies Used:

- Microsoft Azure
- Microsoft Sentinel
- Microsoft Cloud Defender

### Operating Systems Used:

- VM Windows 10 PRO (21H2)
- VM Linux Ubuntu 20.12
<details close>

<div>

</summary>

  ## Here is an overview of Azure logging at different layers: tenant, subscription, and resource:
  
1. Tenant Level Logging: Azure allows you to enable logging at the tenant level, capturing logs for services and activities that span across your entire Azure tenant. This includes Azure AD logs, Azure Resource Manager logs, and more. Tenant-level logging provides a holistic view of activities and events across your Azure environment.

2. Subscription Level Logging: Logging can also be configured at the subscription level, allowing you to monitor and track activities specific to a particular Azure subscription. This includes capturing logs for resource deployments, management operations, and changes within the subscription.

3. Resource Level Logging: Azure provides the ability to enable logging at the resource level, focusing on specific Azure resources such as virtual machines, storage accounts, or databases. Resource-level logging offers detailed insights into the operations, performance metrics, and diagnostic data of individual resources.

By leveraging logging at these different layers, you can effectively monitor and analyze activities, detect security threats, troubleshoot issues, and ensure compliance across your Azure environment.

<p align="center">
<img src="https://i.imgur.com/lLaiDex.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<details close>

---

</summary>

  ## In this lab, we will utilize two GeoIP files to enhance our analysis capabilities. These files will allow us to correlate IP addresses and determine the geographical location from which the attacks originated. This information will provide valuable insights into the source of the attacks and help us strengthen our security measures. <b>

- To begin, we will download two IP files that are essential for our analysis. 

![image](https://user-images.githubusercontent.com/112146207/231026643-50acaa3b-3f83-4467-b6ee-49cf28ee2225.png)

- Next, we will access our Azure account and navigate to the "Storage accounts" section. From there, we will initiate a search for "Storage accounts" and select the option to "Create storage account." This will enable us to set up a new storage account in Azure.

<p align="center">
<img src="https://i.imgur.com/gjdNQ3d.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- While creating the storage account, please ensure that you place it under the "RG-Cyber-Lab" resource group. Additionally, specify a unique storage account name, select the desired region, and set the redundancy to locally-redundant storage (LRS). These configurations will ensure that the storage account is properly organized and aligned with the required specifications for our lab.

- It's important to remember that redundancy in cybersecurity plays a crucial role in enhancing fault tolerance. Fault tolerance refers to the system's ability to continue operating smoothly even in the event of a failure in one of its components. By implementing redundancy measures, such as locally-redundant storage (LRS) in our storage account, we ensure that our data remains available and protected even if certain components or systems encounter issues or failures.

<p align="center">
<img src="https://i.imgur.com/weQglom.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- After successfully creating the storage account, proceed to the search bar and enter "storage account" to locate and access the storage account within Azure. This will allow us to manage and configure the storage account according to our specific requirements. 
> Within the storage account, create a container named “ipgeodata”

<p align="center">
<img src="https://i.imgur.com/RMcy7Va.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Click on ```ipgeodata``` and we will upload the 2 files we downloaded
> This might take a while since one of the files is very large, but patience is key! 

<p align="center">
<img src="https://i.imgur.com/6JGq0yO.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We now need to generate SAS URLs for both of these files.
> A SAS (Shared Access Signature) is utilized to grant access to files on an individual basis, rather than providing unrestricted access to the entire container. It allows for more granular control and security by specifying the specific permissions and timeframe for which the access is granted.

- We will first copy the file names and jot them down on a notepad 

<p align="center">
<img src="https://i.imgur.com/iLc6Obf.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- To generate a SAS for the first file, right-click on its name and select the "Generate SAS" option. Ensure that you generate the SAS for the city blocks IPv4 file specifically. 

<p align="center">
<img src="https://i.imgur.com/DaESGBQ.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Extend the expiration date, by setting it to be at least one year in the future, and then click on the "Generate SAS" button. After generating the SAS, make sure to copy the Blob SAS URL and save it in your notes for future reference. This SAS URL will be utilized later in the process, so it's crucial to keep it readily available.

- Allow me to explain the rationale behind this process. The purpose of generating the SAS URL is to provide it to the Log repository. The Log repository will utilize this SAS URL to access and read the data from the specified file in Azure Sentinel. This data will then be ingested into the Azure Sentinel database for further analysis and monitoring. We can anticipate exploring this integration in the upcoming steps, which will allow us to leverage the power of Azure Sentinel for efficient log management and analysis.

<p align="center">
<img src="https://i.imgur.com/JdqUeoP.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- The identical steps will be followed for the "City-Locations" file.

<p align="center">
<img src="https://i.imgur.com/iHJFjV6.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- It is crucial to have copied the Blob URLs to a notepad or another document, as we will be relying on them for further steps. Please ensure that you have securely saved the Blob URLs, as they will be necessary for our upcoming tasks and integrations.

<p align="center">
<img src="https://i.imgur.com/U6UO6vT.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Our SIEM (Security Information and Event Management) solution will be configured to monitor our log analytics workspace. It will collect, analyze, and identify logs in real-time.

<p align="center">
<img src="https://i.imgur.com/NCuqbKt.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- To create a Log Analytics workspace, please navigate to portal.azure.com and use the search bar to find "log analytics workspace". Once located, click on the "Create" button to initiate the workspace creation process. 

> Enter your resource group, name, and region
> Click "create"

<p align="center">
<img src="https://i.imgur.com/VOSRAh3.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We have created our Log Analytics workspace, which will be enriched with Geo data to correlate IP addresses and determine origins. Now, we will create our SIEM resource and establish a connection with the Log Analytics workspace. This integration will empower us to monitor and respond to security events effectively by utilizing the comprehensive log data collected within the workspace. It will enhance our cybersecurity capabilities and enable proactive threat management. 

- In the Azure portal, search for "Microsoft Sentinel" in the search bar and click "Create" to initiate the provisioning of the powerful security information and event management (SIEM) solution. 

- Afterward, select your Log Analytics workspace and click on the "Add" button to establish the connection between the Microsoft Sentinel SIEM solution and the Log Analytics workspace. 

![image](https://user-images.githubusercontent.com/112146207/231042846-4b76998a-cc03-4c4c-9568-174d36763b7b.png)

- We will now create 2 Watchlists within Azure Sentinel and ingest geo-data CSV Files from Azure Storage

![image](https://user-images.githubusercontent.com/112146207/231043926-67b9a1bc-b5cf-4b74-81bc-2370a8901560.png)

- Now add this information exactly and use YOUR Blob URLs that you copied and saved 

![image](https://user-images.githubusercontent.com/112146207/231044304-b37eee42-396e-4d38-a3b7-24f2d6062360.png)

![image](https://user-images.githubusercontent.com/112146207/231045193-e4b86e10-1d9f-43a7-98da-d447ddcb1955.png)

![image](https://user-images.githubusercontent.com/112146207/231052913-3c103c38-3ee6-4f4c-9efa-8d3832d7252a.png)

- For the second watchlist, I put this information in

![image](https://user-images.githubusercontent.com/112146207/231053907-bdec4f66-2204-4310-8cd4-0ead229d4c39.png)

- Now we have to allow these files to “upload”/load from our storage account into Sentinel/Log Analytics Workspace. The big one will likely take over 24 hours

- We will go to the work analytics workspace and query the watchlists we created just to make sure we can see the records from both watchlists 

- It should look something like this:

![image](https://user-images.githubusercontent.com/112146207/231058252-685c7062-26c7-427d-970d-34c9fe2d851a.png)
<details close>

---

</summary>

In this section, we will create a Linux VM, and we're going to configure the Windows security event logs from our Windows machine, and the syslog logs from our Linux machine to send to our log analytics workspace. In addition to the VM, we will also configure logging for the NSG (AKA the mini firewalls) and we're going to send flow logs into the log analytics workspace. 

To create another Virtual Machine in Azure, use the same Region, Resource Group, and VNet as the previous VM, and name it "linux-vm". 
Avoid choosing B1s for the VM size as it may stop creating logs during a DDoS attack. 
Use a username and password for authentication instead of SSH keys to restrict access. 
Lastly, open up the NSG to all traffic to allow for inbound and outbound traffic control.
  
![image](https://user-images.githubusercontent.com/112146207/231915269-71d8b02a-9a26-4606-80a6-15b9e2dbfb3e.png)
![image](https://user-images.githubusercontent.com/112146207/231915347-62f9b7dc-8417-407e-8c89-868c9030add0.png)


---
 
To enhance the security of your Azure environment, there are three steps that we will take. 
First, enable Microsoft Defender for Cloud for your Log Analytics Workspace. 
 
![image](https://user-images.githubusercontent.com/112146207/231920040-942a930e-8b64-4e57-a570-1c061285996c.png)
   
![image](https://user-images.githubusercontent.com/112146207/231926363-08254f1f-16cb-4063-b121-98b233bd2e3c.png)
 
Make sure everything is checked off and that you have your resource group, subscription, and your log analytics workspace info.
![image](https://user-images.githubusercontent.com/112146207/231927395-28eb3a69-932b-4460-8cad-6641401e2528.png)

We're going to go ahead and enable the security policy since we will use it later on

Make sure to click on "security policy" and then click "Add more standards"

![image](https://user-images.githubusercontent.com/112146207/232256225-219fdb5c-777a-452f-bdc3-7b4b71149ba1.png)

We're going to add NIST 800-53: Security and privacy controls and Azure CIS 1.4.0 (latest version)

We will now enable Defender Plans for both VMs and SQL Instances on VMs to detect and respond to potential security threats
  
Now go back to Microsoft Defender for Cloud and click on "environment settings" and we're going to do the same thing for our work analytics workspace
  
![image](https://user-images.githubusercontent.com/112146207/232257863-6e58db5c-69af-45f5-83cb-f5344f04ce41.png)

![image](https://user-images.githubusercontent.com/112146207/232257971-c3b108d8-5904-4084-ad61-7a53d68eda18.png)

This is from the Windows event log, so we are configuring this to send security events to our log analytics workspace

![image](https://user-images.githubusercontent.com/112146207/232257982-e4af1d6f-14eb-4a93-9ea6-f49c6a99f7aa.png)
 
Now we will configure logging and log forwarding for our NSG (mini firewall) 
  
We will start by going to our Azure home page, clicking "Windows VM", then going to networking,
click on "windows-vm-nsg" 
  
![image](https://user-images.githubusercontent.com/112146207/232265995-760bf78d-f492-4d34-baaf-4651c0dc47c7.png)

We will now create some NSG flow logs 
  
![image](https://user-images.githubusercontent.com/112146207/232343530-137bf329-4e04-4045-b763-f99d2fb0246a.png)

We will do the same thing for our Linux VM 
  
![image](https://user-images.githubusercontent.com/112146207/232343651-7cad24f6-e116-4672-ae02-af80285dd4ec.png)
<details close>

---

</summary>
 
In this section, we will enable diagnostic settings for both NSGs 
  
Search "VM", click "Windows-VM" go to networking and click on your network security group 
Click on "diagnostic settings" and "add diagnostic setting" 
Put in your information and click "save" 

![image](https://user-images.githubusercontent.com/112146207/232344379-94128f13-b60a-4ea2-9ddd-75138f189657.png)

Do the same process for the Linux-VM
  
We will now add data connectors to our VMs and create some data collection rules 
  
First, go to Sentinel and click on "data connectors" and search "windows" 

You should be able to see "Windows security events via AMA"

Then click "open connector page" 

> What is Data Connectors? You can stream all security events from the Windows machines connected to your Microsoft Sentinel workspace using the Windows agent. This connection enables you to view dashboards, create custom alerts, and improve investigation. This gives you more insight into your organization’s network and improves your security operation capabilities.

![image](https://user-images.githubusercontent.com/112146207/232345194-b5cccdab-a8b2-482b-9f6b-ab2cfa2c7762.png)

Click "create data collection rule" 

This allows events/logs to be brought into the log analytics workspace from our VMs

Fill in the information 

![image](https://user-images.githubusercontent.com/112146207/232345533-3e5fe29a-37f0-42c9-ac76-0c3d84546730.png)

Go to resources and click "add resources" 
  
![image](https://user-images.githubusercontent.com/112146207/232345592-00326467-8e19-4e76-a2ba-132f0819b762.png)

And at the end, it should look like this 

![image](https://user-images.githubusercontent.com/112146207/232345624-34d89b98-2405-4606-ad08-50b6a0ff32d1.png)

We will now do this for our Linux VM
  
Search "log analytics workspace" and click on "agents"
 
Click on "Linux servers" and click "Data collection rules" 
  
![image](https://user-images.githubusercontent.com/112146207/232345917-8ff52ffd-9f89-4769-8fcb-c25e2acb3b30.png)

Click "Create" 
  
![image](https://user-images.githubusercontent.com/112146207/232345993-dff68917-0376-4073-b1ef-eb6c80c3eac6.png)

Now go to resources and click on the Linux-VM

![image](https://user-images.githubusercontent.com/112146207/232346046-56e725ef-55e0-45e5-8dc1-fb7ff2e326be.png)

We will now add a data source for our Linux VM
  
The data source type is "Linux Syslog" and leave LOG_AUTH set to LOG_DEBUG

The rest of the logs should be "none"
  
![image](https://user-images.githubusercontent.com/112146207/232346488-8bd5f1a3-5b25-4bbd-b8cb-48dcc902debb.png)

The final result should look something like this 
  
![image](https://user-images.githubusercontent.com/112146207/232348779-32801522-00c0-4f07-a9b8-1d50ea834278.png)
 
We will go back to our log analytics workspace and create another window to make sure it's collecting application logs 
  
![image](https://user-images.githubusercontent.com/112146207/232349001-7d3135e3-2d2d-42ed-a3b9-2c7cc9b65f60.png)

The final result should look like this

![image](https://user-images.githubusercontent.com/112146207/232349296-8e3bd95d-0dc4-4d8a-b597-3bb9363dc859.png)

Now, we will keep checking/ refreshing the log analytics agents tab and ensure the VMs show up there 
  
Go to "log analytics workspace" and then "agents" and make sure that your Windows and Linux VMs are showing up 
  
![image](https://user-images.githubusercontent.com/112146207/232349589-a0925f87-7c29-4b37-aa07-d1289f503c3e.png)

![image](https://user-images.githubusercontent.com/112146207/232349604-fbb882c4-332c-44d1-a4f2-ed38879289e5.png)

Type in "syslog" and click "run" and you should be able to see your logs coming in 
  
![image](https://user-images.githubusercontent.com/112146207/232672859-5c5ad373-9d24-42e6-a114-ee59c9e99ac6.png)
  
> We are now exploring KQL which is very similar to SQL 

> KQL helps us filter through logs to show us exactly what we want to find 
 
![image](https://user-images.githubusercontent.com/112146207/232673231-f23821d7-0627-43ed-80b9-5d0c7980ddc4.png)

- We will log in to our attack VM and fail a couple of logins against the Linux and Windows computers and observe them in the log analytics 
  
> Get the public IP address of your Windows VM

> Go to SSMS 

> Fail to login 3 times 

![image](https://user-images.githubusercontent.com/112146207/232674032-a4f854b3-7946-4d24-9b34-e2b16b10a153.png)
  
- Now fail to login an RDP 
  
![image](https://user-images.githubusercontent.com/112146207/232674353-95d1b883-5f04-42f4-a907-16f79cdbf617.png)
 
  We will now fail login 3 times for our Linux machine and 1 successful connection 

![image](https://user-images.githubusercontent.com/112146207/232672245-53df1c87-5d0b-4a2c-9917-00bd40a845ee.png)

We can now go to "log Analytics" 

The KQL query will look at the SSMS Authentication logs on the Windows computer

We can see that the IP address is that of the attack VM
  
![image](https://user-images.githubusercontent.com/112146207/232676039-4441b94f-6f57-4e85-9d65-ae3c1c8bb345.png)
  
We will check our Linux failed authentication attempts 
  
We can see the times I tried to log in using an incorrect user and password 
  
![image](https://user-images.githubusercontent.com/112146207/232676715-cb74bdf0-52a7-46d6-af05-ed6267308bfd.png)

![image](https://user-images.githubusercontent.com/112146207/232676902-4c080307-0cbf-49ee-b609-7bd7309dc3e1.png)

We will now check the failed RDP failures 
  
![image](https://user-images.githubusercontent.com/112146207/232677531-d8b20f1d-f561-4105-9585-70e24baae515.png)
<details close>

---

</summary>

In this section, we will bring tenant-level logs from Azure Active Directory 
 
The most important part of this lab is to get the "Azure Active Directory Premium P2"

Go to Active Directory> Licenses > All products

Then click "add" and you should be able to see the free trial to Premium P2

After that is all set and done 

Search "Azure AD" go to "Security" then click "Identity Protection" and find "User Risk Policy"

Make sure that you have a user risk policy on 

Sign-in risk policy should also be on 
  
![image](https://user-images.githubusercontent.com/112146207/232914636-98383955-f4f8-412b-a5e4-f6a941508934.png)
  
Go to Azure AD and search for "diagnostic settings"

We will configure which logs we want to be collected 
 
![image](https://user-images.githubusercontent.com/112146207/232916264-d04e3c5f-4979-4282-855a-31a275df883e.png)
  
In Azure AD look for "users" and create a new user 

![image](https://user-images.githubusercontent.com/112146207/232917967-8c2786ca-1d8c-4afb-a201-a16d9cd01ad1.png)

Next, we will assign our dummy_user the role of Global Administrator 
  
![image](https://user-images.githubusercontent.com/112146207/232919275-093723f3-579e-4587-a580-0954f07d08a2.png)
  
Now we have to delete our dummy_user
  
![image](https://user-images.githubusercontent.com/112146207/232920249-a0fa6588-8650-48fd-b009-6a6f39b30a0f.png)
  
We will now simulate a brute force attack against AAD

Then we will observe those logs in the work analytics workspace 
  
First, get your V.S code 
  
![image](https://user-images.githubusercontent.com/112146207/233215613-0b2e01f1-1f40-4a80-8aff-4a872309ae60.png)

Run the "AAD-Brute-Force-Success-Simulator.ps1" from within your attack-VM
  
![image](https://user-images.githubusercontent.com/112146207/233221860-106b7e3f-b176-4f47-bf82-b2a0d54202df.png)
  
 We now go back to our log analytics workspace 

Click on "logs" 

We are using KQL to query logs we want to see
  
![image](https://user-images.githubusercontent.com/112146207/233224548-ad887a59-bd80-48a3-b4a3-2f0196f4b52e.png)

<details close>

In this lab, we will bring in subscription-level logging (activity log)
  
First, we will export Azure Activity Logs to the log analytics workspace 
  
Go to "Azure monitor" click "activity log" and find "export activity logs" 

From there click "Add diagnostic setting" 
  
![image](https://user-images.githubusercontent.com/112146207/233226463-261d0029-d59e-4f6e-b392-3c1e63b47e68.png)

We will now create a new resource group named "Scratch-Resource Group" and "critical infrastructure wastewater" 

![image](https://user-images.githubusercontent.com/112146207/233227240-5c494dbb-0dec-4649-9aaf-d55845edeb6f.png)

We will now delete the resource groups we just created 

The reason for this is that we want to generate logs and observe them 
  
We will produce test lab queries 

This is to better understand KQL and how to use it to filter through log activity
  
![image](https://user-images.githubusercontent.com/112146207/233230084-af667fea-5b7e-4fb4-81f2-a1d43227e4e4.png) 
<details close>

---

</summary>
  
In this lab, we will collect logs for our blob storage and our key vault

We will Configure logging for our storage account by enabling diagnostic settings for blob storage

![image](https://user-images.githubusercontent.com/112146207/233239472-a29723af-0560-45ea-86f6-b604f8475c7a.png)
  
Generate some Logs for Azure Storage (read some blobs/files)
  
![image](https://user-images.githubusercontent.com/112146207/233239985-f46c4019-6955-484d-81d3-109330c46641.png)
  
We are going to create a diagnostic setting to enable logging of the key vault
  
![image](https://user-images.githubusercontent.com/112146207/233242773-15eb31ee-af85-465c-8829-396201ab2934.png)

We will now configure logging for our key vault 
  
First, we will create a Key Vault Instance
  
![image](https://user-images.githubusercontent.com/112146207/233240688-6d9b72f0-2e52-47cd-8a62-022ed5cf76ae.png)

- Now we will add a secret to Key Vault called “Tenant-Global-Admin-Password” with a made-up password
  
![image](https://user-images.githubusercontent.com/112146207/233241999-ced3fa1c-47a4-4f10-b861-b336ea2cd6c2.png)

- Generate some Logs for Key Vault by reading this secret within the Azure Portal

> Observe the Logs (they may take a moment to appear) -  Use the KQL Query Cheat Sheet
