This project aim to help organizations with multiple gateway clusters centralize all their gateway logs and reports into a central storage (ADLS Gen 2) and allow easy and quick exploration of those logs either by:

- Easily access all the gateway logs without having to remote access to the gateway server
- Explore the logs with a Power BI Report
- Explore the logs using a SPARK Engine like Azure Synapse Analytics

![image](./Images/Architecture.png)

Blog Post: https://www.linkedin.com/pulse/power-bi-gateway-monitoring-troubleshooting-solution-rui-romano/ 

# Setup

## Requirements

- [Azure Data Lake Storage Account (ADLS Gen 2)](https://docs.microsoft.com/en-us/azure/storage/blobs/create-data-lake-storage-account) with [Hierarchical Namespace](https://docs.microsoft.com/en-us/azure/storage/blobs/create-data-lake-storage-account#enable-the-hierarchical-namespace) enabled
- [PowerShell 7](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2) on the Gateway Server, with the following modules installed: [Az.Accounts](https://www.powershellgallery.com/packages/Az.Accounts), [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage)

### Azure Data Lake Storage Account (ADLS Gen 2)

Using your Azure Subscription create a new Azure Data Lake Storage resource, follow the steps of following link:

https://docs.microsoft.com/en-us/azure/storage/blobs/create-data-lake-storage-account

Enable the [Hierarchical Namespace](https://docs.microsoft.com/en-us/azure/storage/blobs/create-data-lake-storage-account#enable-the-hierarchical-namespace) when creating the storage account.

![image](./Images/AzurePortal_StorageHierarchicalNamespace.png)

### PowerShell Modules

On the gateway server ensure [PowerShell 7](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2) is installed and install the following required modules: 
- [Az.Accounts](https://www.powershellgallery.com/packages/Az.Accounts)
- [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage)

To install the modules above, open a PowerShell 7 prompt and run the following commands:

```powershell
Install-Module Az.Accounts -MinimumVersion "2.8.0" -verbose

Install-Module Az.Storage -MinimumVersion "4.6.0" -verbose

Install-Module MicrosoftPowerBIMgmt -MinimumVersion "1.2.1077" -verbose
```

## Deploy scripts to Gateway Server

On each Gateway Server you should clone/copy this repo powershell scripts into a local folder (ex: c:\PBIGTWMonitor)

### Change Config.Json

Open the [Configuration file](.\Config.json) and configure the following settings:

- StorageAccountConnStr
  
  Open the ADLS Gen 2 storage account, go to "Access Keys" tab and copy the "Connection String" field:

  ![image](./Images/AzurePortal_StorageConnStr.png)

- GatewayLogsPath
  
  Location of the Gateway logs & reports files.

  Confirm if the 'GatewayLogsPath' point to the correct path of the gateway logs - [more info](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-log-files)

  The script automatically discovers the GatewayId and Number of Cores of the gateway and stores this information on the /metadata/gatewayproperties.json file, but its possible to override these values on the 'GatewayLogsPath' property of the configuration file:

  ![image](./Images/ConfigFile_PathProperty.png)

- OutputPath

    Temporary location of the gateway logs before copying to blob storage

- StorageAccountContainerName

    Name of the container in the storage account

- StorageAccountContainerRootPath

    Root path on the storage container to where the log files will be written to

### Schedule Task

Configure a Windows Schedule Task to run the script [Run.ps1](./Run.ps1) every hour/day

![image](./Images/Setup_ScheduleTask.png)

# Power BI Template

After loading the data to the data lake you can open the [Power BI template](./Gateway%20Monitor%20-%20FromLake.pbit) and refresh to get a report with insights on the gateways.

## Template Parameters

After opening the Power BI Template file (.pbit) the following parameter window will popup:

![image](./Images/PBI_TemplateParams.png)

| Parameter      | Description
| ----------- | -------- 
| DataLocation      | URL Path to the root folder including the container name (default: '/pbigatewaymonitor/RAW'), using the Distributed File System (DFS) endpoint of the storage account, ex: https<span>://storage.<strong>dfs</strong>.core.windows.net/<strong>pbigatewaymonitor/raw</strong>
| NumberDays | Filter to the log files to be fetched, if '10' Power BI will read only the latest 10 days of logs/queries/counters Default: null (all days)
| SinceDate | Only read log/query files since the specified date. This parameter overrides 'NumberDays' Default: null (all days)
| MaxLogTextLength | Max size of text column of logs. Default: 1000
| LogFilters | Comma separated file names of log files to be fetched. Default: "gatewayerrors,gatewayinfo" If 'None' log files will be excluded 
| GatewayFilters | Comma separated gateway id's. Default: All Gateways

## Template from Disk

Its possible to use the Power BI template directly over the gateway server log folder or export files from the [Export Logs](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-tshoot#collect-logs-from-the-on-premises-data-gateway-app) feature.

Use the [Gateway Monitor - FromDisk.pbit](Gateway%20Monitor%20-%20FromDisk.pbit) template file and set the "DataLocation" parameter to the root folder where the logs are stored. Its possible to include logs from multiple gateways, just ensure the logs for each gateway have their own folder. Ex: c:\temp\gatewaymonitor\gateway1; c:\temp\gatewaymonitor\gateway2

## Logs Page

![image](./Images/PBI_LogPage.png)

## Queries Page

![image](./Images/PBI_QueriesPage.png)

## Gateway Profile
![image](./Images/PBI_GatewayProfile.png)

## Counters Page

![image](./Images/PBI_Counters.png)

## Requests Page

![image](./Images/PBI_RequestsPage.png)

## Mashups Profiles Page

![image](./Images/PBI_MashupProfiles.png)

## Mashups Logs Page

![image](./Images/PBI_MashupLogs.png)

## Theme
Theme Background Images here: https://alluringbi.com/gallery/