---
author: janvanmeirvenne
comments: true
date: 2014-07-01 07:38:59+00:00
layout: post
link: http://scug.be/jan/2014/07/01/scom-dw-not-being-updated-with-operational-data/
slug: scom-dw-not-being-updated-with-operational-data
title: SCOM DW not being updated with operational data?
wordpress_id: 265
categories:
- '#scom'
- '#sysctr'
---

With this blogpost I want to make a 'catch-all' knowledge-article containing problems and fixes I learned in the field regarding SCOM DW synchronization issues. I will update this post regularly if I encounter any new phenomenon on this subject.

**Possible Cause 1: The synchronization objects and settings are missing from the management group**

**Diagnosis**

Run the following powershell-commands in the SCOM powershell interface of the affected management group:

_get-SCOMClass -name:Microsoft.SystemCenter.DataWarehouseSynchronizationService|Get-ScomClassInstance_

If no objects are returned it means that the workflows responsible for synchronizing data are not running.

_Add-pssnapin microsoft.enterprisemanagement.operationsmanager.client_
_Set-location OperationsManagerMonitoring::_
_New-managementgroupconnection <SCOM management server>
get-DefaultSetting ManagementGroup\DataWarehouse\DataWarehouseDatabaseName $DataWarehouseDatabaseName_
_get-DefaultSetting ManagementGroup\DataWarehouse\DataWarehouseServerName $DataWarehouseSqlServerInstance_

If the default settings are not set this indicates that the DW registration has been broken.

**Causes**

The breakage and disappearance of the DW synchronization objects and settings can happen when a SCOM 2007->2012 upgrade fails and you have to recover the RMS. The issue is hard to detect (especially if you do not use reporting much) as no errors are generated.

**Solution**

The settings and objects need to be regenerated manually using the script below. This will add all necessary objects to SCOM with the correct server and database references. The DW properties will also be added to the default-settings section.

You will have to edit the script and enter the Operations Manager Database server name, Data Warehouse servername and console path in the script . This is a PowerShell script which needs to copied to text and rename to .ps1 after entering the required information to run under PowerShell.

_#Populate these fields with Operational Database and Data Warehouse Information_

_#Note: change these values appropriately_

_$OperationalDbSqlServerInstance = "<OpsMgrDB server instance. If its default instance, only server name is required>"_

_$OperationalDbDatabaseName = "OperationsManager"_

_$DataWarehouseSqlServerInstance = "<OpsMgrDW server instance. If its default instance, only server name is required>"_

_$DataWarehouseDatabaseName = "OperationsManagerDW"_

_$ConsoleDirectory = "<OpsMgr Console Location by default it will be C:\Program Files\System Center 2012\Operations Manager\Console"_

_$dataWarehouseClass = get-SCOMClass -name:Microsoft.SystemCenter.DataWarehouse_

_$seviewerClass = get-SCOMClass -name:Microsoft.SystemCenter.OpsMgrDB.AppMonitoring_

_$advisorClass = get-SCOMClass -name:Microsoft.SystemCenter.DataWarehouse.AppMonitoring _

_$dwInstance = $dataWarehouseClass | Get-SCOMClassInstance_

_$seviewerInstance = $seviewerClass | Get-SCOMClassInstance_

_$advisorInstance = $advisorClass | Get-SCOMClassInstance _

_#Update the singleton property values_

_$dwInstance.Item($dataWarehouseClass.Item("MainDatabaseServerName")).Value = $DataWarehouseSqlServerInstance_

_$dwInstance.Item($dataWarehouseClass.Item("MainDatabaseName")).Value = $DataWarehouseDatabaseName _

_$seviewerInstance.Item($seviewerClass.item("MainDatabaseServerName")).Value = $OperationalDbSqlServerInstance_

_$seviewerInstance.Item($seviewerClass.item("MainDatabaseName")).Value = $OperationalDbDatabaseName_

_$advisorInstance.Item($advisorClass.item("MainDatabaseServerName")).Value = $DataWarehouseSqlServerInstance_

_$advisorInstance.Item($advisorClass.item("MainDatabaseName")).Value = $DataWarehouseDatabaseName _

_$dataWarehouseSynchronizationServiceClass = get-SCOMClass -name:Microsoft.SystemCenter.DataWarehouseSynchronizationService_

_#$dataWarehouseSynchronizationServiceInstance = $dataWarehouseSynchronizationServiceClass | Get-SCOMClassInstance _

_$mg = New-Object Microsoft.EnterpriseManagement.ManagementGroup -ArgumentList localhost_

_$dataWarehouseSynchronizationServiceInstance = New-Object Microsoft.EnterpriseManagement.Common.CreatableEnterpriseManagementObject -ArgumentList $mg,$dataWarehouseSynchronizationServiceClass _

_$dataWarehouseSynchronizationServiceInstance.Item($dataWarehouseSynchronizationServiceClass.Item("Id")).Value = [guid]::NewGuid().ToString()_

_#Add the properties to discovery data_

_$discoveryData = new-object Microsoft.EnterpriseManagement.ConnectorFramework.IncrementalDiscoveryData _

_$discoveryData.Add($dwInstance)_

_$discoveryData.Add($dataWarehouseSynchronizationServiceInstance)_

_$discoveryData.Add($seviewerInstance)_

_$discoveryData.Add($advisorInstance)_

_$momConnectorId = New-Object System.Guid("7431E155-3D9E-4724-895E-C03BA951A352")_

_$connector = $mg.ConnectorFramework.GetConnector($momConnectorId) _

_$discoveryData.Overwrite($connector)_

_#Update Global Settings. Needs to be done with PS V1 cmdlets_

_Add-pssnapin microsoft.enterprisemanagement.operationsmanager.client_

_cd $ConsoleDirectory_

_.\Microsoft.EnterpriseManagement.OperationsManager.ClientShell.NonInteractiveStartup.ps1_

_Set-DefaultSetting ManagementGroup\DataWarehouse\DataWarehouseDatabaseName $DataWarehouseDatabaseName_

_Set-DefaultSetting ManagementGroup\DataWarehouse\DataWarehouseServerName $DataWarehouseSqlServerInstance_

If the script ran successfully and you run the commands specified in the diagnosis-section you should receive valid object- and settings information. The synchronization should start within a few moments.

**Sources**

[http://support.microsoft.com/kb/2771934](http://support.microsoft.com/kb/2771934)
