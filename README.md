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
