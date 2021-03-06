<properties 
   pageTitle="Manage Azure Data Lake Analytics using Azure PowerShell | Azure" 
   description="Learn how to manage Data Lake Analytics jobs, data sources, users. " 
   services="data-lake-analytics" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>
 
<tags
   ms.service="data-lake-analytics"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="10/27/2015"
   ms.author="jgao"/>

# Manage Azure Data Lake Analytics using Azure PowerShell

[AZURE.INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Learn how to manage Azure Data Lake Analytics accounts, data sources, users, and jobs using the Azure PowerShell. To see management topics using other tools, click the tab select above.

**Prerequisites**

Before you begin this tutorial, you must have the following:

- **An Azure subscription**. See [Get Azure free trial]https://azure.microsoft.com/en-us/pricing/free-trial/).
- **Azure PowerShell 1.0 or above**. See [Install and configure Azure PowerShell](../install-configure-powershell.md). After you have installed Azure PowerShell 1.0 or above, you should run the following cmdlet to install the Azure Data Lake Analytics module.
	
		Install-Module AzureRM.DataLakeStore
		Install-Module AzureRM.DataLakeAnalytics

	For more information on the **AzureRM.DataLakeStore** module, see [PowerShell Gallery](http://www.powershellgallery.com/packages/AzureRM.DataLakeStore). 
    For more information on the **AzureRM.DataLakeAnalytics** module, see [PowerShell Gallery](http://www.powershellgallery.com/packages/AzureRM.DataLakeAnalytics). 

	If you are creating a Data Lake account for the first time, run:

		Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
		Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.DataLakeAnalytics"

	To connect to Azure, use the following cmdlets:

		Login-AzureRmAccount
		Get-AzureRmSubscription  # for finding the Azure Subscription ID
		Set-AzureRmContext -SubscriptionID <Azure Subscription ID>
		
**To list the cmdlets**:

	Get-Command *Azure*DataLakeAnalytics*

<!-- ################################ -->
<!-- ################################ -->



<!-- ################################ -->
<!-- ################################ -->
## Manage accounts

Before running any Data Lake Analytics jobs, you must have a Data Lake Analytics account. Unlike Azure HDInsight, you don't pay for an Analytics account when it is not 
running a job.  You only pay for the time when it is running a job.  For more informaiton, see 
[Azure Data Lake Analytics Overview](data-lake-analytics-overview.md).  

###Create accounts

	$resourceGroupName = "<ResourceGroupName>"
	$dataLakeStoreName = "<DataLakeAccountName>"
	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	$location = "<Microsoft Data Center>"
	
	Write-Host "Create a resource group ..." -ForegroundColor Green
	New-AzureRmResourceGroup `
		-Name  $resourceGroupName `
		-Location $location
	
	Write-Host "Create a Data Lake account ..."  -ForegroundColor Green
	New-AzureRmDataLakeStoreAccount `
		-ResourceGroupName $resourceGroupName `
		-Name $dataLakeStoreName `
		-Location $location 
	
	Write-Host "Create a Data Lake Analytics account ..."  -ForegroundColor Green
	New-AzureRmDataLakeAnalyticsAccount `
		-Name $dataLakeAnalyticsAccountName `
		-ResourceGroupName $resourceGroupName `
		-Location $location `
		-DefaultDataLake $dataLakeStoreName
	
	Write-Host "The newly created Data Lake Analytics account ..."  -ForegroundColor Green
	Get-AzureRmDataLakeAnalyticsAccount `
		-ResourceGroupName $resourceGroupName `
		-Name $dataLakeAnalyticsAccountName  

You can also use an Azure Resource Group template. A tempalte for creating a Data Lake Analytics account and the dependent Data Lake Store account is in [Appendix A](#appendix-a). Save the template into a file with .json template, and then use the following PowerShell script to call it:


	$AzureSubscriptionID = "<Your Azure Subscription ID>"
	
	$ResourceGroupName = "<New Azure Resource Group Name>"
	$Location = "EAST US 2"
	$DefaultDataLakeStoreAccountName = "<New Data Lake Store Account Name>"
	$DataLakeAnalyticsAccountName = "<New Data Lake Analytics Account Name>"
	
	$DeploymentName = "MyDataLakeAnalyticsDeployment"
	$ARMTemplateFile = "E:\Tutorials\ADL\ARMTemplate\azuredeploy.json"  # update the Json template path 
	
	Login-AzureRmAccount
	
	Select-AzureRmSubscription -SubscriptionId $AzureSubscriptionID
	
	# Create the resource group
	New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Location
	
	# Create the Data Lake Analytics account with the default Data Lake Store account.
	$parameters = @{"adlAnalyticsName"=$DataLakeAnalyticsAccountName; "adlStoreName"=$DefaultDataLakeStoreAccountName}
	New-AzureRmResourceGroupDeployment -Name $DeploymentName -ResourceGroupName $ResourceGroupName -TemplateFile $ARMTemplateFile -TemplateParameterObject $parameters 

 
###List account

List Data Lake Analytics accounts within the current subscription

	Get-AzureRmDataLakeAnalyticsAccount
	
The output:

	Id         : /subscriptions/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/resourceGroups/learn1021rg/providers/Microsoft.DataLakeAnalytics/accounts/learn1021adla
	Location   : eastus2
	Name       : learn1021adla
	Properties : Microsoft.Azure.Management.DataLake.Analytics.Models.DataLakeAnalyticsAccountProperties
	Tags       : {}
	Type       : Microsoft.DataLakeAnalytics/accounts

List Data Lake Analytics accounts within a specific resource group

	Get-AzureRmDataLakeAnalyticsAccount -ResourceGroupName $resourceGroupName

Get details of a specific Data Lake Analytics account

	Get-AzureRmDataLakeAnalyticsAccount -Name $adlAnalyticsAccountName

Test existence of a specific Data Lake Analytics account

	Test-AzureRmDataLakeAnalyticsAccount -Name $adlAnalyticsAccountName

The cmdlet will return either **True** or **False**.

###Delete Data Lake Analytics accounts

	$resourceGroupName = "<ResourceGroupName>"
	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	
	Remove-AzureRmDataLakeAnalyticsAccount -Name $dataLakeAnalyticsAccountName 

Delete a Analytics account will not delete the dependent Data Lake Storage account. The following example deletes the Data Lake Analytics account and the default Data Lake Store account

	$resourceGroupName = "<ResourceGroupName>"
	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	$dataLakeStoreName = (Get-AzureRmDataLakeAnalyticsAccount -ResourceGroupName $resourceGroupName -Name $dataLakeAnalyticName).Properties.DefaultDataLakeAccount

	Remove-AzureRmDataLakeAnalyticsAccount -ResourceGroupName $resourceGroupName -Name $dataLakeAnalyticName 
	Remove-AzureRmDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStoreName

<!-- ################################ -->
<!-- ################################ -->
## Manage account data sources

Data Lake Analytics currently supports the following data sources:

- [Azure Data Lake Storage](data-lake-storage-overview.md)
- [Azure Storage](storage-introduction.md)

When you create an Analytics account, you must designate an Azure Data Lake Storage account to be the default 
storage account. The default Data Lake Store account is used to store job metadata and job audit logs. After you have 
created an Analytics account, you can add additional Data Lake Storage accounts and/or Azure Storage account. 

### Find the default Data Lake Store account

	$resourceGroupName = "<ResourceGroupName>"
	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	$dataLakeStoreName = (Get-AzureRmDataLakeAnalyticsAccount -ResourceGroupName $resourceGroupName -Name $dataLakeAnalyticName).Properties.DefaultDataLakeAccount


### Add additional Azure Blob storage accounts

	$resourceGroupName = "<ResourceGroupName>"
	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	$AzureStorageAccountName = "<AzureStorageAccountName>"
	$AzureStorageAccountKey = "<AzureStorageAccountKey>"
	
	Add-AzureRmDataLakeAnalyticsDataSource -ResourceGroupName $resourceGroupName -AccountName $dataLakeAnalyticName -AzureBlob $AzureStorageAccountName -AccessKey $AzureStorageAccountKey

### Add additional Data Lake Store accounts

	$resourceGroupName = "<ResourceGroupName>"
	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	$AzureDataLakeName = "<DataLakeStoreName>"
	
	Add-AzureRmDataLakeAnalyticsDataSource -ResourceGroupName $resourceGroupName -AccountName $dataLakeAnalyticName -DataLake $AzureDataLakeName 

### List data sources:

	$resourceGroupName = "<ResourceGroupName>"
	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"

	(Get-AzureRmDataLakeAnalyticsAccount -ResourceGroupName $resourceGroupName -Name $dataLakeAnalyticName).Properties.DataLakeStoreAccounts
	(Get-AzureRmDataLakeAnalyticsAccount -ResourceGroupName $resourceGroupName -Name $dataLakeAnalyticName).Properties.StorageAccounts
	


<!-- ################################ -->
<!-- ################################ -->
## Manage jobs

You must have an Data Lake Analytics account before you can create a job.  For more information, see [Manage Data Lake Analytics accounts](#manage-data-lake-analytics-accounts).

### List jobs

	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName
	
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName -State Running, Queued
	#States: Accepted, Compiling, Ended, New, Paused, Queued, Running, Scheduling, Starting
	
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName -Result Cancelled
	#Results: Cancelled, Failed, None, Successed 
	
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName -Name <Job Name>
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName -Submitter <Job submitter>

	# List all jobs submitted on January 1 (local time)
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName `
		-SubmittedAfter "2015/01/01"
		-SubmittedBefore "2015/01/02"	

	# List all jobs that succeeded on January 1 after 2 pm (UTC time)
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName `
		-State Ended
		-Result Succeeded
		-SubmittedAfter "2015/01/01 2:00 PM -0"
		-SubmittedBefore "2015/01/02 -0"

	# List all jobs submitted in the past hour
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName `
		-SubmittedAfter (Get-Date).AddHours(-1)

### Get job details

	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"
	Get-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName -JobID <Job ID>
	
### Submit jobs

	$dataLakeAnalyticsAccountName = "<DataLakeAnalyticsAccountName>"

	#Pass script via path
	Submit-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName `
		-Name $jobName `
		-ScriptPath $scriptPath

	#Pass script contents
	Submit-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName `
		-Name $jobName `
		-Script $scriptContents

> [AZURE.NOTE] The default priority of a job is 1000, and the default degree of parallelism for a job is 1.


### Cancel jobs

	Stop-AzureRmDataLakeAnalyticsJob -AccountName $dataLakeAnalyticName `
		-JobID $jobID


## Manage catalog items

The U-SQL catalog is used to structure data and code so they can be shared by U-SQL scripts. The catalog enables the highest performance possible with data in Azure Data Lake. For more information, see [Use U-SQL catalog](data-lake-analytics-use-u-sql-catalog.md).

###List catalog items

	#List databases
	Get-AzureRmDataLakeAnalyticsCatalogItem `
		-AccountName $adlAnalyticsAccountName `
		-ItemType Database
	
	
	
	#List tables
	Get-AzureRmDataLakeAnalyticsCatalogItem `
		-AccountName $adlAnalyticsAccountName `
		-ItemType Table `
		-Path "master.dbo"

###Get catalog item details 

	#Get a database
	Get-AzureRmDataLakeAnalyticsCatalogItem `
		-AccountName $adlAnalyticsAccountName `
		-ItemType Database `
		-Path "master"
	
	#Get a table
	Get-AzureRmDataLakeAnalyticsCatalogItem `
		-AccountName $adlAnalyticsAccountName `
		-ItemType Table `
		-Path "master.dbo.mytable"

###Test existence of  catalog item

	Test-AzureRmDataLakeAnalyticsCatalogItem  `
		-AccountName $adlAnalyticsAccountName `
		-ItemType Database `
		-Path "master"

###Create catalog secret
	New-AzureRmDataLakeAnalyticsCatalogSecret  `
			-AccountName $adlAnalyticsAccountName `
			-DatabaseName "master" `
			-Secret (Get-Credential -UserName "username" -Message "Enter the password")

### Modify catalog secret
	Set-AzureRmDataLakeAnalyticsCatalogSecret  `
			-AccountName $adlAnalyticsAccountName `
			-DatabaseName "master" `
			-Secret (Get-Credential -UserName "username" -Message "Enter the password")



###Delete catalog secret
	Remove-AzureRmDataLakeAnalyticsCatalogSecret  `
			-AccountName $adlAnalyticsAccountName `
			-DatabaseName "master"


## Use Azure Resource Manager groups

Applications are typically made up of many components, for example a web app, database, database server, storage,
and 3rd party services. Azure Resource Manager (ARM) enables you to work with the resources in your application 
as a group, referred to as an Azure Resource Group. You can deploy, update, monitor or delete all of the 
resources for your application in a single, coordinated operation. You use a template for deployment and that 
template can work for different environments such as testing, staging and production. You can clarify billing 
for your organization by viewing the rolled-up costs for the entire group. For more information, see [Azure 
Resource Manager Overview](resource-group-overview.md). 

An Data Lake Analtyics service can include the following components:

- Azure Data Lake Analytics account
- Required default Azure Data Lake Storage account
- Additional Azure Data Lake Storage accounts
- Additional Azure Storage accounts

You can create all these components under one ARM group to make them easier to manage.

![Azure Data Lake Analytics account and storage](./media/data-lake-analytics-manage-use-portal/data-lake-analytics-arm-structure.png)

An Data Lake Analytics account and the dependent storage accounts must be placed in the same Azure data center.
The ARM group however can be located in a different data center.  

##See also 

- [Overview of Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
- [Get started with Data Lake Analytics using Azure Preview Portal](data-lake-analytics-get-started-portal.md)
- [Manage Azure Data Lake Analytics using Azure Preview portal](data-lake-analytics-use-portal.md)
- [Monitor and troubleshoot Azure Data Lake Analytics jobs using Azure Preview Portal](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)

##Apendix A - Data Lake Analytics ARM template

The following ARM template can be used to deploy a Data Lake Analytics account and its dependent Data Lake Store account.  Save it as a json file, and then use PowerShell script to call the template. For more information, see
[Deploy an application with Azure Resource Manager template](resource-group-template-deploy.md#deploy-with-powershell) and [Authoring Azure Resource Manager templates](resource-group-authoring-templates.md).

	{
		"$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
		"contentVersion": "1.0.0.0",
		"parameters": {
			"adlAnalyticsName": {
				"type": "string",
				"metadata": {
					"description": "The name of the Data Lake Analytics account to create."
				}
			},
			"adlStoreName": {
				"type": "string",
				"metadata": {
					"description": "The name of the Data Lake Store account to create."
				}
			}
		},
		"resources": [{
			"name": "[parameters('adlAnalyticsName')]",
			"type": "Microsoft.DataLakeAnalytics/accounts",
			"location": "East US 2",
			"apiVersion": "2015-10-01-preview",
			"dependsOn": ["[concat('Microsoft.DataLakeStore/accounts/',parameters('adlStoreName'))]"],
			"tags": {
				
			},
			"properties": {
				"defaultDataLakeAccount": "[parameters('adlStoreName')]"
				}
			}
		},
		{
			"name": "[parameters('adlName')]",
			"type": "Microsoft.DataLakeStore/accounts",
			"location": "East US 2",
			"apiVersion": "2015-10-01-preview",
			"dependsOn": [],
			"tags": {
				
			}
		}],
		"outputs": {
			"adlAnalyticsAccount": {
				"type": "object",
				"value": "[reference(resourceId('Microsoft.DataLakeAnalytics/accounts',parameters('adlAnalyticsName')))]"
			},
			"adlStoreAccount": {
				"type": "object",
				"value": "[reference(resourceId('Microsoft.DataLakeStore/accounts',parameters('adlStoreName')))]"
			}
		}
	}