---
author: janvanmeirvenne
comments: true
date: 2014-02-04 15:19:28+00:00
layout: post
link: http://scug.be/jan/2014/02/04/applying-update-rollup-5-to-operations-manager-2012-service-pack-1/
slug: applying-update-rollup-5-to-operations-manager-2012-service-pack-1
title: Applying update rollup 5 to Operations Manager 2012 Service Pack 1
wordpress_id: 200
categories:
- '#scom'
- '#sysctr'
---

Hey guys, this is a quick procedure based on the official KB guide @ [http://support.microsoft.com/kb/2904680](http://support.microsoft.com/kb/2904680)

This guide assumes that you are running with administrative rights.



	
  * 


**Validate the sources
**



	
    * 


**_Download all updates from Microsoft Update
_**



	
      * Open an internet explorer window and browse to [http://catalog.update.microsoft.com/v7/site/Search.aspx?q=2904680](http://catalog.update.microsoft.com/v7/site/Search.aspx?q=2904680)
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd110.png)_
_

	
      * Add all listed downloads to your basket and download them to a local location
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd21.png)![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd31.png)_
_




	
    * **_Extract the appropriate cab files
_**For each downloaded update, enter its downloaded folder and extract the cab file. If more than one cab file is present then extract each file containing your language code (usually ENU) and / or processor architecture (x64 or x86). The other cab files can be deleted. **_
_**

	
    * 


**_Create folder structure
_**



	
      * 


Create a folder structure like this:


![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd41.png)

	
      * Move the extracted MSP files in their appropriate folder







	
  * 


**Apply the update to
**



	
    * 


**_Management Servers
_**



	
      * 


Perform these steps for each management server



	
        * Log in on the system

	
        * Use a file explorer to go to the remote or local location of the patch file structure you created

	
        * Go to 'Server', right-click the MSP file and click 'Apply'
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd51.png)

	
        * 


When the installation completes, you will be prompted for a reboot, do this as soon as you can.
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd61.png)








	
    * 


**_Gateway Servers_**



	
      * 


Perform these steps for each gateway server



	
        * Log in on the system

	
        * Use a file explorer to go to the remote or local location of the patch file structure you created

	
        * Go to 'Server', right-click the MSP file and click 'Apply'
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd71.png)

	
        * When the installation completes, you might be prompted for a reboot, do this as soon as you can.
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd81.png)







	
    * 


**_ACS
_**



	
      * 


Perform these steps for each gateway server



	
        * Log in on the system

	
        * Use a file explorer to go to the remote or local location of the patch file structure you created

	
        * Go to 'ACS', right-click the MSP file and click 'Apply'
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd91.png)







	
    * 


**_Web console servers
_**



	
      * 


Perform these steps for each web console server



	
        * Log in on the system

	
        * Use a file explorer to go to the remote or local location of the patch file structure you created

	
        * Go to 'WebConsole', right-click the MSP file and click 'Apply'
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd101.png)

	
        * 


Use a file explorer to navigate to 'C:\Windows\Microsoft.NET\Framework64\v2.0.50727\CONFIG' and open the web.config file
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd111.png)


	
        * Add the following line under the system.web node in the xml structure:










![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd121.png)








<table style="border-collapse: collapse;" border="0" > 
<tbody valign="top" >
<tr >

<td style="padding-left: 7px; padding-right: 7px; border: solid 0.5pt;" >_<machineKey validationKey="AutoGenerate,IsolateApps" decryptionKey="AutoGenerate,IsolateApps" validation="3DES" decryption="3DES"/>_
</td>
</tr>
</tbody>
</table>






	
  * 


**_Operations Console servers
_**



	
    * 


Perform the following steps on all systems with an Operations Console**_
_**



	
      * Log in on the system

	
      * Use a file explorer to go to the remote or local location of the patch file structure you created

	
      * 


Go to 'Console', right-click the appropriate (x86 or x64)MSP file and click 'Apply'
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd131.png)















	
  * **Update the databases
**








	
  * **_Data Warehouse_**

	
  * Connect to the Operations Manager Data Warehouse using SQL management studio

	
  * Use the studio to load the following query-file from a Operations Manager management server:





<table style="border-collapse: collapse;" border="0" > 
<tbody valign="top" >
<tr >

<td style="padding-left: 7px; padding-right: 7px; border: solid 0.5pt;" >< Base Installation Location of SCOM>\Server\SQL Script for Update Rollups\UR_Datawarehouse.sql
</td>
</tr>
</tbody>
</table>





![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd141.png)






	
  * Execute the loaded query against the Operations Manager Data Warehouse database



	
  * 


**_Operational_**



	
    * Connect to the Operations Manager database using SQL management studio

	
    * Use the studio to load the following query-file from a Operations Manager management server:








<table style="border-collapse: collapse;" border="0" > 
<tbody valign="top" >
<tr >

<td style="padding-left: 7px; padding-right: 7px; border: solid 0.5pt;" >< Base Installation Location of SCOM>\Server\SQL Script for Update Rollups\update_rollup_mom_db.sql
</td>
</tr>
</tbody>
</table>





![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd151.png)






	
  * Execute the loaded query against the Operations Manager database



	
  * 


**Import the management packs
**



	
    * Open the Operations Console and connect to the management group you are updating

	
    * In the left hand pane, go to 'Administration', right-click on 'Management Packs' and choose 'Import Management Packs'
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd161.png)

	
    * 


In the dialog, click 'Add' and select 'Add from disk…'. When prompted to use the online catalog, choose 'no'.
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd171.png)


	
    * In the file select dialog, navigate to the following location on a management server and select all files, click 'Open':








<table style="border-collapse: collapse;" border="0" > 
<tbody valign="top" >
<tr >

<td style="padding-left: 7px; padding-right: 7px; border: solid 0.5pt;" ><Root SCOM Installation Directory>\Server\Management Packs for Update Rollups
</td>
</tr>
</tbody>
</table>





![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd181.png)






	
  * The import dialog will complain about a missing dependency management pack called 'Microsoft.SystemCenter.IntelliTraceCollectorInstallation.mpb'. You must use add this dependency by using the same method as the previous management packs. This file can be found under the 'ManagementPacks' folder on the SCOM installation media.

	
  * When all management packs are ready to import, click 'Import' to import them.
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd191.png)

	
  * When prompted for security consense, click 'Yes'
![](http://scug.be/jan/files/2014/02/020414_1519_Applyingupd201.png)


