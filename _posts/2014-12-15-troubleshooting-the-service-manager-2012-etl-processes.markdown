---
author: janvanmeirvenne
comments: true
date: 2014-12-15 15:53:29+00:00
layout: post
link: http://scug.be/jan/2014/12/15/troubleshooting-the-service-manager-2012-etl-processes/
slug: troubleshooting-the-service-manager-2012-etl-processes
title: Troubleshooting the Service Manager 2012 ETL processes
wordpress_id: 275
categories:
- '#scsm'
- '#sysctr'
tags:
- '#scsm'
- '#sysctr'
---

This post will aid in troubleshooting the following issues concerning the Service Manager Data Warehouse:     
- Slow execution of ETL jobs      
- ETL jobs failing to complete      
- ETL jobs failing to start

 

# 1. Troubleshooting

 

# 

 

- Open a remote desktop session to the Service Manager Management Server

 

- Open the service manager management shell

 

- Request the data-warehouse jobs   <table cellpadding="0" border="1" cellspacing="0" ><tbody >       <tr >         
<td width="623" valign="top" >           

**Get-SCDWJob –computername <Your DW Server>|ft Name, Status, CategoryName,IsEnabled**

        
</td>       </tr>     </tbody></table>

 

- This will result in a list of data warehouse jobs and their state

 

[![image](http://scug.be/jan/files/2014/12/image_thumb.png)](http://scug.be/jan/files/2014/12/image.png)

 

- If there are jobs with a ‘stopped’status, then resume them:   <table cellpadding="0" border="1" cellspacing="0" ><tbody >       <tr >         
<td width="623" valign="top" >           

**Start-SCDWJob –jobname <The name of the job to start (eg ‘DWMaintenance’) –computername ****<Your DW Server>**

        
</td>       </tr>     </tbody></table>

 

- If there are jobs that are not enabled (IsEnabled column is ‘false’) AND the MPSyncJob or DWMaintenance jobs are not running (they disable some jobs at runtime) then re-enable them:   <table cellpadding="0" border="1" cellspacing="0" ><tbody >       <tr >         
<td width="623" valign="top" >           

**Enable-SCDWJob –jobname <The name of the job to start (eg ‘DWMaintenance’) –computername ****<Your DW Server>**

        
</td>       </tr>     </tbody></table>

 

- Run the following script to reset the jobs (it will rerun all jobs in the correct order). This script exists thanks to [Travis Wright](https://gallery.technet.microsoft.com/PowerShell-Script-to-Run-a4a2081c/view/Discussions).

 

  <table cellpadding="0" width="881" border="1" cellspacing="0" ><tbody >       <tr >         
<td width="879" valign="top" >           

           

$DWComputer = “<Your DW Server>

           

$SMExtractJobName = "<Operational Management Group Name> "

           

$DWExtractJobName = "<DW Management Group Name> "

           

Import-Module 'C:\Program Files\Microsoft System Center 2012\Service Manager\Microsoft.EnterpriseManagement.Warehouse.Cmdlets.psd1'

           

function Start-Job ($JobName, $Computer)

           

{

           

$JobRunning = 1

           

while($JobRunning -eq 1)

           

{

           

$JobRunning = Start-Job-Internal $JobName $Computer

           

}

           

}

           

function Start-Job-Internal($JobName, $Computer)

           

{

           

$JobStatus = Get-JobStatus $JobName

           

if($JobStatus -eq "Not Started")

           

{

           

Write-Host "Starting the $JobName Job..."

           

Enable-SCDWJob -JobName $JobName -Computer $Computer

           

Start-SCDWJob -JobName $JobName -Computer $Computer

           

Start-Sleep -s 5

           

}

           

elseif($JobStatus -eq "Running")

           

{

           

Write-Host "$JobName Job is already running. Waiting 30 seconds and will call again."

           

Start-Sleep -s 30

           

return 1

           

}

           

else

           

{

           

Write-Host "Exiting since the job is in an unexpected status"

           

exit

           

}

           

$JobStatus = "Running"

           

while($JobStatus -eq "Running")

           

{

           

Write-Host "Waiting 30 seconds"

           

Start-Sleep -s 30

           

$JobStatus = Get-JobStatus $JobName

           

Write-Host "$JobName Job Status: $JobStatus"

           

if($JobStatus -ne "Running" -and $JobStatus -ne "Not Started")

           

{

           

Write-Host "Exiting since the job is in an unexpected status"

           

exit

           

}

           

}

           

return 0

           

}

           

function Get-JobStatus($JobName)

           

{

           

$Job = Get-SCDWJob -JobName $JobName -Computer $Computer

           

$JobStatus = $Job.Status

           

return $JobStatus

           

}

           

#DWMaintenance

           

Start-Job "DWMaintenance" $DWComputer

           

#MPsyncJob

           

Start-Job "MPSyncJob" $DWComputer

           

#ETL

           

Start-Job $SMExtractJobName $DWComputer

           

Start-Job $DWExtractJobName $DWComputer

           

Start-Job "Transform.Common" $DWComputer

           

Start-Job "Load.Common" $DWComputer

           

#Cube processing

           

Start-Job "Process.SystemCenterConfigItemCube" $DWComputer

           

Start-Job "Process.SystemCenterWorkItemsCube" $DWComputer

           

Start-Job "Process.SystemCenterChangeAndActivityManagementCube" $DWComputer

           

Start-Job "Process.SystemCenterServiceCatalogCube" $DWComputer

           

Start-Job "Process.SystemCenterPowerManagementCube" $DWComputer

           

Start-Job "Process.SystemCenterSoftwareUpdateCube" $DWComputer 

        
</td>       </tr>     </tbody></table>

 

- If a particular job keeps stalling / failing during or after the script execution, check which job-module is having problems:   <table cellpadding="0" border="1" cellspacing="0" ><tbody >       <tr >         
<td width="623" valign="top" >           

Get-SCDWJobModule –jobname <The name of the job experiencing issues> –computername <Your DW Server>

        
</td>       </tr>     </tbody></table>

 

- Check how long the jobs has been failing / stalling   <table cellpadding="0" border="1" cellspacing="0" ><tbody >       <tr >         
<td width="623" valign="top" >           

Get-SCDWJob –jobname <The name of the job experiencing issues> -NumberOfBatches 10 –computername <Your DW Server>

        
</td>       </tr>     </tbody></table>

 

- Check the ‘Operations Manager’ eventlog on the data warehouse server. Look for events with as source ‘Data Warehouse’. Error or Warning events might pinpoint the issue with the job.

 

- Check the CPU and Memory of the data warehouse server, and check if one or both are peaking a lot.

 

 

# 2. Common possible causes

 

 

#### 2.1. Resource Pressure

 

The data warehouse server takes up a lot of resources to process data. Job duration and reliability can be greatly increased by providing sufficient CPU and memory resources. Exact requirements depend on each individual setup, but these are some guidelines:   <table cellpadding="0" border="1" cellspacing="0" ><tbody >       <tr >         
<td width="208" >           

**CPU**

        
</td>          
<td width="208" >           

**Memory**

        
</td>          
<td width="208" >           

**Hard Drive**

        
</td>       </tr>        <tr >         
<td width="208" >           

4-core 2.66Ghz

        
</td>          
<td width="208" >           

Server Component: 8-16GB

           

Databases: 8-32Gb

        
</td>          
<td width="208" >           

Server Component: 10Gb

           

Databases: 400Gb

        
</td>       </tr>     </tbody></table>

 

#### 2.2. Service Failure

 

The ETL process of the Data Warehouse depends on multiple services to function correctly:

 

- Microsoft Monitoring Agent

 

- System Center Data Access

 

- System Center Management Configuration

 

- SQL Server SCSMDW

 

- SQL Serer Analysis Services

 

- SQL Server Agent

 

- SQL Server

 

Verify if these services are running correctly (the ‘Application’ and / or ‘Operations Manager’ event logs can hold clues as to why a service can not run correctly.

 

#### 2.3. Authentication Failure

 

Various runas-accounts are used to execute the ETL jobs:

 

- A workflow account that executes program logic on the data warehouse server. This account must have local administrator privileges on the data warehouse server.

 

- An operational database account that has access to the SCSM databases for data extraction. This account must be owner of all databases.

 

- A runas-account that has administrator privileges on both the operational and the data warehouse management groups.

 

Most of these accounts are entered during setup and should not be changed afterwards. If these accounts do not have the required permissions then some or all functionalities related to the ETL process can be impacted. 

 

Should error events indicate that a permission issue is the cause, then verify and repair the necessary permissions for these accounts.
