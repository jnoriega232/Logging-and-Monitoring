# Logging and Monitoring

## Objectives for the next 6 Labs, which will help us build our Logging and Monitoring System:

- Overview of Azure Logging at Different Layers (Tenant, Activity, Resource)
- Geo IP Data Ingestion & Log Analytics & Microsoft Sentinel Setup
- Enable Microsoft Defender for Cloud & Configure Log Collection for VMs
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

- Continuously monitor and refresh the log analytics agents tab within the log analytics workspace to verify the presence of the Windows and Linux VMs. Navigate to the "log analytics workspace" and access the "agents" section to ensure that both the Windows and Linux VMs are listed and properly connected to the log analytics workspace.
  
<p align="center">
<img src="https://i.imgur.com/ixZZzri.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

<p align="center">
<img src="https://i.imgur.com/8WIRErR.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Navigate to the "log analytics workspace" and access the "logs" section. Enter the search term "syslog" and execute the search by selecting the "run" option. This action will enable you to observe the incoming logs.

<p align="center">
<img src="https://i.imgur.com/Nah5rXU.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

> Currently, we are delving into KQL (Kusto Query Language), which bears a strong resemblance to SQL. KQL assists us in sifting through logs effectively, allowing us to pinpoint precisely the information we are seeking to discover. 
 
<p align="center">
<img src="https://i.imgur.com/JiAQ1jt.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Next, we will access our designated attack virtual machine (VM) and intentionally perform a few unsuccessful login attempts against both the Linux and Windows computers. By doing so, we can closely monitor and examine the corresponding logs in the log analytics system.
  
> Get the public IP address of your Windows VM

> Go to SSMS 

> Fail to login 3 times 

<p align="center">
<img src="https://i.imgur.com/S8Pin77.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Deliberately induce a login failure using Remote Desktop Protocol (RDP).
  
<p align="center">
<img src="https://i.imgur.com/n2spfPq.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will now employ PowerShell to trigger three sequential unsuccessful login attempts for our Linux machine, subsequently followed by a single successful connection.

<p align="center">
<img src="https://i.imgur.com/JN9mxT4.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We can now proceed to access "log Analytics."

- The KQL query will examine the SSMS Authentication logs on the Windows computer.

- Upon inspection, we can observe that the IP address corresponds to the attacking virtual machine.
  
<p align="center">
<img src="https://i.imgur.com/SALCiY2.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will now review the failed authentication attempts made on our Linux system. 
  
- We can observe the instances where I attempted to log in using an incorrect username and password combination.

<p align="center">
<img src="https://i.imgur.com/zf4AySq.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

<p align="center">
<img src="https://i.imgur.com/iaWC1Tg.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will now analyze the unsuccessful login attempts made through Remote Desktop Protocol (RDP), with the option to filter the results based on the IP address of the attacker.
  
<p align="center">
<img src="https://i.imgur.com/LH8Ql4L.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<details close>

---

</summary>

- In this section, we will import tenant-level logs from Azure Active Directory. The crucial aspect of this lab is obtaining "Azure Active Directory Premium P2."

- To do so, navigate to Active Directory > Licenses > All products.

- Click on "Add" and locate the free trial for Premium P2.

- Once that is completed, search for "Azure AD," access "Security," and select "Identity Protection."

- Locate the "User Risk Policy" and ensure it is enabled.

- Additionally, verify that the "Sign-in Risk Policy" is also activated.
  
<p align="center">
<img src="https://i.imgur.com/AlxgXP2.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Navigate to Azure AD and perform a search for "diagnostic settings."

- We will customize the logging settings to specify which logs we want to collect.
 
<p align="center">
<img src="https://i.imgur.com/JD7YMdP.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Within Azure AD, locate the "users" section and proceed to create a new user.

<p align="center">
<img src="https://i.imgur.com/nFrYwBS.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Subsequently, we will assign the role of Global Administrator to our dummy_user.
  
<p align="center">
<img src="https://i.imgur.com/rf9Jmkz.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Now, we need to delete our dummy_user to generate logs for the removal of a global administrator.
  
<p align="center">
<img src="https://i.imgur.com/NDvrXIZ.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will now initiate a simulated brute force attack against Azure Active Directory (AAD), followed by the analysis of the resulting logs in the work analytics workspace.
  
- To begin, obtain your Visual Studio Code (VS Code) environment.
  
![image](https://user-images.githubusercontent.com/112146207/233215613-0b2e01f1-1f40-4a80-8aff-4a872309ae60.png)

- We will execute the "AAD-Brute-Force-Success-Simulator.ps1" script from our attack-VM.
  
<p align="center">
<img src="https://i.imgur.com/asUbJwi.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Let's return to our log analytics workspace and navigate to the "logs" section. From there, we will utilize KQL (Kusto Query Language) to query and display the logs of our interest.
  
<p align="center">
<img src="https://i.imgur.com/Ucl7MtZ.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

<details close>

- In this lab, our objective is to incorporate subscription-level logging, specifically the activity log.

- To achieve this, we will export Azure Activity Logs to the log analytics workspace.

- To initiate the process, navigate to "Azure Monitor," select "Activity Log," and locate the "Export activity logs" option.

- Proceed by clicking "Add diagnostic setting" to continue.
  
<p align="center">
<img src="https://i.imgur.com/8aLWepA.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Let's create two new resource groups: one named "Scratch-Resource Group" and another named "Critical Infrastructure Wastewater."

<p align="center">
<img src="https://i.imgur.com/uEaOzgl.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- To generate logs in the management plane and observe them, we will delete the resource groups we previously created.

- The purpose of this action is to generate log activity that we can analyze.

- We will perform test lab queries to gain a better understanding of KQL (Kusto Query Language) and its application in filtering through log activity.

<p align="center">
<img src="https://i.imgur.com/66dEbgo.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>
<details close>

---

</summary>
  
- In this lab, our focus will be on collecting logs for both our blob storage and key vault.

- To accomplish this, we will enable diagnostic settings for our storage account to configure logging specifically for the blob storage component.

<p align="center">
<img src="https://i.imgur.com/la4MQ8T.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Next, we will generate logs for Azure Storage by performing actions such as reading blobs or files within the storage.
  
<p align="center">
<img src="https://i.imgur.com/qYpw0LV.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will create a diagnostic setting to activate logging for the key vault.
  
<p align="center">
<img src="https://i.imgur.com/WE3mFwe.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Now, we will proceed to configure logging for our key vault.

- To begin, we will create a Key Vault instance.
  
<p align="center">
<img src="https://i.imgur.com/lMZYIuf.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- We will now add a secret to the Key Vault, naming it "Tenant-Global-Admin-Password" and assigning a fictitious password to it.
  
<p align="center">
<img src="https://i.imgur.com/haJe2C1.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

- Let's generate some logs for the Key Vault by accessing and reading the secret named "Tenant-Global-Admin-Password" within the Azure Portal.

<p align="center">
<img src="https://i.imgur.com/548Mi3m.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

<p align="center">
<img src="https://i.imgur.com/Z2xZ3zU.png" height="70%" width="70%" alt="Azure Free Account"/> 
</p>

> Take a moment to observe the logs, noting that they may require some time to appear. Feel free to refer to the [KQL Query Cheat Sheet](https://github.com/joshmadakor1/Cyber-Course/blob/main/KQL-Query-Cheat-Sheet.md) for assistance in analyzing the logs.
