---
author: janvanmeirvenne
comments: true
date: 2016-05-02 08:38:16+00:00
layout: post
link: http://scug.be/jan/2016/05/02/scom-kb-you-get-a-cyclicdependencyexception-when-querying-management-pack-information-using-the-powershell-or-sdk-interfaces/
slug: scom-kb-you-get-a-cyclicdependencyexception-when-querying-management-pack-information-using-the-powershell-or-sdk-interfaces
title: 'SCOM KB: You get a CyclicDependencyException when querying Management Pack
  information using the Powershell or SDK interfaces'
wordpress_id: 423
---

**Symptom:
**

When you attempt to use the get-scommanagementpack cmdlet to retrieve SCOM management pack info (or SDK equivalent), you get the following error:


<table style="border-collapse:collapse" border="0" ><tbody valign="top" ><tr >
<td style="padding-left: 7px; padding-right: 7px; border-top:  solid 0.5pt; border-left:  solid 0.5pt; border-bottom:  solid 0.5pt; border-right:  solid 0.5pt" >

Exception of type 'Microsoft.EnterpriseManagement.Common.CyclicDependencyException' was thrown.

</td></tr></tbody></table>


 

![](http://scug.be/jan/files/2016/05/050216_0838_SCOMKBYouge1.png)
	

**Cause:
**

This error means that there at least 2 management packs in the SCOM management group that depend on each other (Cyclic Dependency). This situation is not supported (but does not break functionality of the management packs themselves), and should be rectified if possible.


**Resolution:
**

You can use the following SQL query on the SCOM operational database to identify cyclic MP pairs:


<table style="border-collapse:collapse" border="0" ><tbody valign="top" ><tr >
<td style="padding-left: 7px; padding-right: 7px; border-top:  solid 0.5pt; border-left:  solid 0.5pt; border-bottom:  solid 0.5pt; border-right:  solid 0.5pt" >

SELECT mpas.MPName +
								' <-> '
								+ mpbs.MPName as Cycle



								FROM [OperationsManager].[dbo].[ManagementPackReferences] a



								inner
								join [ManagementPackReferences] b on a.ManagementPackIdReffedBy= b.ManagementPackIdSource



								and a.ManagementPackIdSource = b.ManagementPackIdReffedBy inner
								join ManagementPack mpas on a.ManagementPackIdSource = mpas.ManagementPackId inner
								join ManagementPack mpar on a.ManagementPackIdSource = mpar.ManagementPackId inner
								join ManagementPack mpbs on b.ManagementPackIdSource = mpbs.ManagementPackId


</td></tr></tbody></table>


 

This will show the cycle MP's:


![](http://scug.be/jan/files/2016/05/050216_0838_SCOMKBYouge2.png)
	

You can then use this information to either contact the MP manufacturer for a bug submission, or remove the cyclic reference from your code if you have built the MP's.


After removing the cycle, the exception should not occur again.



 
