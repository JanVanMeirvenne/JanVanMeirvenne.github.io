---
author: janvanmeirvenne
comments: true
date: 2014-10-23 10:31:16+00:00
layout: post
link: http://scug.be/jan/2014/10/23/scom-quick-query-logical-disk-space-for-my-environment/
slug: scom-quick-query-logical-disk-space-for-my-environment
title: 'SCOM Quick Query: Logical Disk Space For My Environment'
wordpress_id: 271
categories:
- '#scom'
- '#sysctr'
tags:
- '#scom'
- '#sysctr'
---

 

Sometimes I get questions in the style of “What is the current state of my environment in terms of…”. If there is no report in SCOM I can point to I usually create a quick query on the Data Warehouse and provide the data as an excel sheet to the requestor. Afterwards, should the question be repeated over and over, I create a report for it and provide self-service information.

 

In order to both prevent forgetting these kind of ‘quick and dirty’ queries, and also sharing my work with you I will occasionally throw in a post if I have a query worth mentioning.

 

Here we go for the first one!

 

If you are not interested in using the extended [Logical Disk MP](http://www.systemcentercentral.com/logical-disk-extension-management-pack/) you can use this query on your DW to quickly get a free space overview of all logical disks in your environment :

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

select max(time) as time,server,disk,size,free,used from              
(              
select perf.DateTime as time,e.path as server, e.DisplayName as disk, round(cast(EP.PropertyXml.value('(/Root/Property[@Guid="A90BE2DA-CEB3-7F1C-4C8A-6D09A6644650"]/text())[1]', 'nvarchar(max)') as int) / 1024,0) as size, round(perf.SampleValue / 1024,0) as free, round(cast(EP.PropertyXml.value('(/Root/Property[@Guid="A90BE2DA-CEB3-7F1C-4C8A-6D09A6644650"]/text())[1]', 'nvarchar(max)') as int) / 1024,0) - round(perf.SampleValue / 1024,0) as used from perf.vPerfRaw perf inner join vManagedEntity e on perf.ManagedEntityRowId = e.ManagedEntityRowId              
inner join vPerformanceRuleInstance pri on pri.PerformanceRuleInstanceRowId = perf.PerformanceRuleInstanceRowId              
inner join vPerformanceRule pr on pr.RuleRowId = pri.RuleRowId              
inner join vManagedEntityProperty ep on ep.ManagedEntityRowId = e.ManagedEntityRowId              
where              
pr.ObjectName = 'LogicalDisk'              
and              
pr.CounterName = 'Free Megabytes'              
and              
ep.ToDateTime is null              
and Perf.DateTime > dateadd(HOUR,-1,GETUTCDATE())              
) data              
group by data.server,data.disk,data.size,data.free,data.used              
order by server,disk              


      
</td>     </tr>   </tbody></table>  

 

Available fields:

 

Time: the timestamp of the presented data     
Server: the server the disk belongs to      
Disk: The name of the logical disk      
Size: the size of the disk in GB      
Free: the free space on the disk in GB      
Used: the used space on the disk in GB

 

 

Please note that I am not a SQL guru, so if you find a query containing war crimes against best practices, don’t hesitate to let me know!

 

 

See you in another knowledge dump!   
