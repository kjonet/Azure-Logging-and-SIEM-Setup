![img](https://i.imgur.com/FpRKtSe.png)

# Azure-Logging-and-SIEM-Setup
Microsoft Sentinel is a cloud-based SEIM solution used to analyze large amounts of data across multiple platforms. Microsoft Sentinel is designed to detect, investigate, and respond to any potential cyber threats that may compromise your organization. This will be a demonstration on how to create a central log repository using the Analytics Workspace within Azure and then we will connect that workspace to Microsoft Sentinel so that logs can be ingested. Once ingested, we'll be able to observe logs from multiple resources. We'll also be ingesting Geo data that will translate on a map giving us insight as to where attacks might originate from around the world. 


![img](https://i.imgur.com/jDPbH8A.png)

## Getting started
Before we can start sending logs, ensure you already have an established Azure subscription and resource group. The first step is to create a Log Analytics workspace, an important component in your security and analytics framework. Type in "Log Analytics workspace" in the search field. and select, "Create". Make sure to align the workspace with the designated resource group for monitoring, ensuring both share the same region. For instance, if your resource group resides in East US, your Log Analytics workspace should synchronize with this region. Complete the required fields, affirm your selections, and proceed to the final step by choosing "Review and Create." 

![img](https://i.imgur.com/8Qijm4G.png)

Next, we'll go into Microsoft Sentinel, select Create, and add our new workspace, creating a data connection between Sentinel and the Log Analytics Workspace. Because of this connection, we will be able to monitor logs and capture alerts from a variety of resources. 

![img](https://i.imgur.com/WggVm5P.png)

## Creating A Watchlist
Next, we're going to go into Microsoft Sentinel to create a watchlist. The watchlist has Geo data comprised of network information, country name, city name, and coordinates. This ingestion will enable us to make queries within Sentinel that will enable us to pinpoint the location of an attack. Once in Microsoft Sentinel, go to 'Watchlist' on the left and create a new watchlist. This will take you to the 'Watchlist wizard'. 
Under 'General', we want the 'Name' and 'Alias' to be "geoip". 

Under 'Source' we will upload a CSV file (provided by cybersecurity professional Josh Madakor) that contains the geo data we'll need to query. [geoip_CSV](https://github.com/joshmadakor1/Cyber-Course-v2/blob/main/Sentinel-Maps(JSON)/geoip-summarized.csv) 
For the 'SearchKey' we'll use "network". We'll use this search key to match the attacker's IP addresses. Finally, select 'Create'. It may take a while for the watchlist to be finalized because the CSV file contains so much data. 

![img](https://i.imgur.com/QbGXnaj.png)

## Testing Our Watchlist Inside Log Analytics

Now that our watchlist has been created, let's try out a query within the Logs Analytics workspace. Type in 'Logs Analytics workspace' within the search bar and select 'Logs' on the left. Once in the Logs workspace, type in the following query: 
_GetWatchlist("geoip")

If the ingestion was complete, you should get results that look similar to this: 

![img](https://i.imgur.com/ijUoKVB.png)

## Enabling Microsoft Defender For Cloud
Now that our Log Analytics workspace is set up and our geo data has been ingested, it's time to enable Microsoft Defender for Cloud. Microsoft Defender for Cloud will help give a secure scope of our Azure environment and ingest logs from our virtual machines. Once we enable Microsoft Defender For Cloud, an agent will be automatically installed on the virtual machines we have created within our resource group. 
Under Microsoft Defender for Cloud, head over to 'Environment Settings'. Go over to the Subscription where you wish to have Microsoft Defender for Cloud-enabled and select the Log Analytics workspace nested below. This will take you to the settings page. 

From the settings page, we want to enable Defender plans on any servers and SQL serves we have within our resource group. 

![img](https://i.imgur.com/qfJdDjY.png)

Next, go into 'Data collection' on the left and check 'All Event'. This will allow logs to be collected from the Windows security log attached to our Windows VM. 

![img](https://i.imgur.com/KrlQX6N.png)

Next, go back into 'Environment Settings' and edit subscription settings by selecting the three dots to the right.  

Here, we want to turn on the following: Servers, Storage, Databases, and Key Vaults.

![img](https://i.imgur.com/fxdaCey.png)


Next, under 'Server' settings, select 'Edit configuration' under the 'Log Analytics agent'. Select 'Custom workspace' and select the workspace we created earlier. Select 'Apply'. and save your settings under 'Defender Plans'. 

<h3>Continueous Export</h3>

Under the 'Continuous export' settings, we're going to enable this particular setting that exports alerts into the log analytics workspace. 

![img](https://i.imgur.com/EICfFiI.png)

## Enable Log Collection for VMs and Network Security Groups

In this section, we're going to configure our virtual machines and their Network Security Groups to forward their logs to the Log Analytics workspace. To begin, we're going to create a storage account. The storage account will be used to capture flow logs from the Network Security Groups that are attached to the Azure virtual machines. A Network Security Group is an Azure-based firewall that monitors traffic within the virtual machines. Flow logs are the traffic going in and out of the virtual machine. Within the Azure portal, search for 'Storage account' and name it. Just like when creating the Log Analytics workspace, the region must be the same as the region you assigned to your resource group. 

Next, we'll enable flow logs for our Network Security Groups. Go to the Azure portal and type in 'nsg'. Select the Virtual machine you want to create flow logs for. In this instance, I'm going to select my 'windows-vm-nsg'
Within the 'Create a flow log' page, under the 'Basics' tab, assign the appropriate storage account and keep the retention at 0.

Select the 'Analytics' tab, select 'Version 2' under 'Traffic Analytics', and check 'Enable Traffic Analytics'. This allows Defender for Cloud to analyze traffic and sort it. Set the 'Traffic Analytics processing interval' to every 10 minutes. Review and Create. 

<h3>Configuring Data Collection Rules</h3>

Next, we'll proceed to configure data collection rules. These rules will help specify what logs from the virtual machines will need to be forwarded to the log analytics workspace. We're going to have to be strategic on what logs we'd like to be forwarded because forwarding too many logs can increase costs for a company. 
To get started, head over to your Log Analytics workspace. Under 'Settings' select 'Agents'. An agent is a type of software installed on the virtual machines that will be used to collect logs. Once selected, click 'Data Collection Rules'. 

![img](https://i.imgur.com/5cB2txQ.png)

At the 'Data collection rules' window, give it a 'Rule Name', and select the resource group, subscription, and appropriate region. Next, select the platform you wish to target. In this instance, we are going to select 'Windows'. 

![img](https://i.imgur.com/9OD8SqX.png)

Under the 'Resources' tab, add your resource group and select the Windows virtual machine. Next, head over to the 'Collect and deliver' tab. 

To specify the logs you wish to collect, select 'Add data source'. Your data source should be 'Windows Event Log'. We're going to collect logs from Application and Security logs. Next, select the destination which should be your Log Analytics workspace that we set up earlier. 

![img](https://i.imgur.com/DrWQONr.png)

Finally, go ahead and create your Data Collection Rule. 

<h3>Windows Firewall Data Collection Rule</h3>

We're going to create one more rule that will enable us to capture logs from Windows Firewall. Head over back to the Log Analytics workspace, select your workspace, then 'Agents', and click on 'Data Collection Rules' again. 
We're going to select the rule we just created, then select 'Data Source'. Next, select 'Windows Event Logs', and at the 'Add data source' window, select 'Custom'. Within the field that says "Use XPath queries to filter event logs and limit data collection." We are going to use a special xpath query [here](https://github.com/joshmadakor1/Cyber-Course-v2/blob/main/Special-Windows-Event-Data-Collection-Rules/Rules.txt) (Provided by cybersecurity professional, Josh Madakor) Add both queries to the XPath field. 

The following query will be used for malware detection: 
- // Windows Defender Malware Detection XPath Query
Microsoft-Windows-Windows Defender/Operational!*[System[(EventID=1116 or EventID=1117)]]

While this query will be used for firewall tampering: 
- // Windows Firewall Tampering Detection XPath Query
Microsoft-Windows-Windows Firewall With Advanced Security/Firewall!*[System[(EventID=2003)]]

Add both XPath queries and save your settings. 

Lastly, Microsoft Defender for Cloud has a great feature that automatically installs agents onto your virtual machines. Let's make sure our Windows agent is installed by going to the 'Log Analytics workspace' and selecting 'Agents'. Again, having an agent installed is crucial to make sure our logs are being forwarded to our workspace. 

![img](https://i.imgur.com/Hsnv10L.png)

## Testing Our Connection

Let's take a look to see if we were able to capture any logs coming from our Windows virtual machine. Go into the 'Log Analytics workspace', head over to 'Logs', and let's type in a simple query using KQL. 
- SecurityEvent
  | where EventID == 4625

  Your results may produce something similar to this:

  ![img](https://i.imgur.com/sRWOPJP.png)

  This shows that our logs are coming in!

## Windows World Map For Failed RDP Authentications 

Now that we've got our logs flowing from our Windows virtual machine, we can create a map that displays where attackers might be trying to authenticate from. To get started, head over to Microsoft Sentinel. Select your Log Analytics Workspace that was created earlier. To the left, under 'Threat Management', select 'Workbooks'. This is where our maps will be created. If there are already pre-made elements in your workbook, go ahead and delete them. Next, select 'Add' and 'Add Query'. Select the 'Advanced Editor' tab and paste the following JSON script found [here](https://github.com/joshmadakor1/Cyber-Course-v2/blob/main/Sentinel-Maps(JSON)/windows-rdp-auth-fail.json) (Provided by cybersecurity professional, Josh Madakor) This attack map will show RDP failures from around the world.  Your attack map should look similar to this: 

![img](https://i.imgur.com/GYh0j4C.png) 

*Please note that my Windows VM has been running for 24hrs. 

From Microsoft Sentinel, we were able to query logs from the Log Analytics workspace that generated the attack map in our workbook. 

![img](https://i.imgur.com/xQ1qI65.png)

## Conclusion

For this project, we were able to set up our own SEIM for deployment within Azure. The Log Analytics workspace serves as a central hub for collecting and analyzing data as it integrates with Microsoft Defender for the Cloud to ensure comprehensive threat detection. Once a workspace is set up and a connection is established through Microsoft Defender for Cloud, the amount of threat hunting one can do is infinite. With the creation of a customized workbook, tailored to your VMs, we can get a visualized look of the geodata we ingested. Enabling us to query locations from around the world. 




