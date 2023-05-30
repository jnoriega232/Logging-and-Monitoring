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

<p align="center">
<img src="https://i.imgur.com/D72hGmc.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Now, we will proceed to create two Watchlists within Azure Sentinel. Additionally, we will ingest CSV files containing geo-data from Azure Storage. These steps will enable us to leverage the Watchlists feature for advanced threat detection and response, while also enriching our security analytics with geo-location information.

<p align="center">
<img src="https://i.imgur.com/rRV673f.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Let's add the following information exactly, using the Blob URLs that you previously copied and saved
- After you carefully fill it out, Review and create

<p align="center">
<img src="https://i.imgur.com/pT7Yffk.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<p align="center">
<img src="https://i.imgur.com/qj0WkUT.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Let's replicate the same steps to create the second watchlist. Pay careful attention to filling in the details exactly as provided
  
<p align="center">
<img src="https://i.imgur.com/fQWcpY2.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<p align="center">
<img src="https://i.imgur.com/1pXDLiZ.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>  

- Now, we need to enable the upload or loading of these files from our storage account into Sentinel/Log Analytics Workspace. It's important to note that the larger file may take more than 24 hours to complete the process due to its size. Patience is required as the system processes and transfers the data from the storage account to the Log Analytics Workspace within Sentinel.

- In the Log Analytics workspace, we will execute queries to verify the presence of records from both watchlists, ensuring their visibility and accessibility. This step will confirm the successful ingestion of data from the watchlists into the Log Analytics workspace, enabling us to utilize the information for analysis and monitoring purposes.

- It should look something like this:

<p align="center">
<img src="https://i.imgur.com/jLEXBWl.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<details close>

---

</summary>

- In this segment, we will establish a Linux VM and configure the Windows security event logs from our Windows machine, along with the syslog logs from our Linux machine, to be sent to the log analytics workspace. Furthermore, we will configure logging for the NSG (Network Security Group), also referred to as mini firewalls, and transmit flow logs into the log analytics workspace. This comprehensive setup will allow us to centralize and analyze logs from various sources, enhancing our monitoring and security capabilities. 

- To create an additional Virtual Machine in Azure, ensure that you select the same Region, Resource Group, and VNet as the previous VM. Name this new VM "linux-vm" to maintain consistency across your deployment. 
- It is recommended to avoid choosing the 1 vcpu VM size due to its potential inability to generate logs during a DDoS attack. 
- For authentication purposes, opt to use a username and password instead of SSH keys to restrict access to the Virtual Machine.
- Finally, make sure that the Network Security Group (NSG) is set up to allow unrestricted traffic flow by permitting all types of traffic. This configuration will enable seamless communication and remove any restrictions imposed by the NSG on the flow of network traffic.

<p align="center">
<img src="https://i.imgur.com/ieyBzhM.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<p align="center">
<img src="https://i.imgur.com/zXLjrZf.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

---
 
- To bolster the security of your Azure environment, our initial step will be to activate Microsoft Defender for Cloud for our subscription. 
  
<p align="center">
<img src="https://i.imgur.com/C33vLa3.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>   
<p align="center">
<img src="https://i.imgur.com/RHG1w2E.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Make sure everything is checked off and that you have your resource group, subscription, and your log analytics workspace info.

<p align="center">
<img src="https://i.imgur.com/IYU9DdO.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- We're going to go ahead and enable the security policy since we will use it later on.

- Make sure to click on "security policy" and then click "Add more standards".

- We're going to add NIST 800-53: Security and privacy controls and Azure CIS 1.4.0 (latest version).
  
<p align="center">
<img src="https://i.imgur.com/NXbqskS.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will now enable Defender Plans for both VMs and SQL Instances on VMs to detect and respond to potential security threats.
  
- Go back to Microsoft Defender for Cloud and click on "environment settings" and we're going to do the same thing for our log analytics workspace.
  
<p align="center">
<img src="https://i.imgur.com/KS6IJJK.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<p align="center">
<img src="https://i.imgur.com/ixD4Uc5.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Go to Data Collection to enable all events. Since the information originates from the Windows event log, we will configure the system to send security events to our designated log analytics workspace. This setup ensures that security-related events from Windows are seamlessly forwarded to the log analytics workspace for centralized monitoring and analysis.

<p align="center">
<img src="https://i.imgur.com/xncoHmZ.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Next, we will proceed with the configuration of logging and log forwarding for our Network Security Group (NSG), often referred to as a mini firewall. 
  
- To begin, navigate to the Azure home page and select "Windows VM." From there, access the networking settings and locate the "windows-vm-nsg" option. This will allow us to proceed with the configuration and management of the Network Security Group associated with the Windows VM. 

<p align="center">
<img src="https://i.imgur.com/ycmjwxR.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Next, we will generate Network Security Group (NSG) flow logs to capture relevant network traffic information.
  
<p align="center">
<img src="https://i.imgur.com/2PAAsVh.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will now do the same thing for our Linux VM.
  
<p align="center">
<img src="https://i.imgur.com/JSQM97Y.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<p align="center">
<img src="https://i.imgur.com/prUXTL1.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

<details close>

---

</summary>
- In this segment, we will activate diagnostic settings for both Network Security Groups (NSGs). Enabling diagnostic settings allows us to capture and store valuable logs and metrics related to the NSGs, aiding in monitoring, analysis, and troubleshooting efforts for enhanced network security.
  
- To proceed, search for "VM" and select "Windows-VM" from the search results. Navigate to the networking section and click on your associated Network Security Group. From there, access the "diagnostic settings" and click on "add diagnostic setting." Fill in the required information and click "save" to finalize the configuration. 

<p align="center">
<img src="https://i.imgur.com/QOrUjcb.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Follow the same process mentioned earlier for the Linux-VM.
  
<p align="center">
<img src="https://i.imgur.com/u5xdKYn.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Next, we will integrate data connectors into our Virtual Machines (VMs) and establish data collection rules to gather relevant information. By adding data connectors and configuring data collection rules, we can effectively capture and analyze data from the VMs for monitoring and security purposes.
  
- To begin, navigate to Microsoft Sentinel and access the "data connectors" section. Search for "windows" and locate the "Windows security events via AMA" connector. Click on "open connector page" to proceed with the configuration. 

> Data Connectors in Microsoft Sentinel provide the ability to stream all security events from Windows machines connected to your workspace using the Windows agent. This integration allows you to leverage dashboards, create custom alerts, and enhance your investigation capabilities. By gaining deeper insights into your organization's network and bolstering your security operations, you can improve overall security posture and incident response effectiveness.

<p align="center">
<img src="https://i.imgur.com/94GlalH.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Proceed by clicking on "create data collection rule" to initiate the process. This step enables the transfer of events and logs from our Virtual Machines (VMs) into the log analytics workspace. Fill in the necessary information, ensuring the proper configuration for seamless data collection and analysis.

<p align="center">
<img src="https://i.imgur.com/Iu6VqXH.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Navigate to the "resources" section and select "add resources" to proceed with the next step.

<p align="center">
<img src="https://i.imgur.com/03wn7eo.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Finally, after completing the necessary steps, the configuration should resemble the following representation. 

<p align="center">
<img src="https://i.imgur.com/03wn7eo.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Now, let's repeat the same process for our Linux VM. Start by searching for "log analytics workspace" and access the "agents" section. From there, click on "Linux servers" and proceed to click on "Data collection rules." 
  
<p align="center">
<img src="https://i.imgur.com/iJHN4Wx.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Click on the "Create" button to initiate the setup process for data collection rules on the Linux VM. 
  
<p align="center">
<img src="https://i.imgur.com/omUx1ii.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Proceed to the "resources" section and select the Linux-VM from the available resources. 

<p align="center">
<img src="https://i.imgur.com/uLBIGKF.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
  
- Next, let's add a data source for our Linux VM by selecting the "Linux Syslog" as the data source type. Ensure that the LOG_AUTH is set to LOG_DEBUG while leaving the other logs as "none." 
  
<p align="center">
<img src="https://i.imgur.com/5OHzKO6.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- After completing the necessary configurations, the final result should resemble the provided example. This indicates that the data source for the Linux VM has been successfully added, allowing for the collection and analysis of syslog data within the log analytics workspace.
  
<p align="center">
<img src="https://i.imgur.com/8fMpu0J.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Return to the log analytics workspace and open a new window to verify that the collection of application logs is properly configured and operational.
  
<p align="center">
<img src="https://i.imgur.com/4A5wLJ1.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- After performing the necessary checks and configurations, the final result should resemble the provided example. 

<p align="center">
<img src="https://i.imgur.com/M6AUWi7.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

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
