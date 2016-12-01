---
author: janvanmeirvenne
comments: true
date: 2013-01-23 17:03:30+00:00
layout: post
link: http://scug.be/jan/2013/01/23/scom-excluding-items-from-a-group-based-on-hosted-roles/
slug: scom-excluding-items-from-a-group-based-on-hosted-roles
title: 'SCOM: excluding items from a group based on hosted roles'
wordpress_id: 68
---

A customer heavily uses groups to scope access and views to items in SCOM. One of the access-profiles was that for the SQL-team. They can only see the SQL servers they manage along with all objects hosted by those servers. However, SQL server is also a component of TMG, which they do not manage. Because of this they didn’t want to have the TMG servers clutter their clean view on their environment.

 

The original solution was to statically exclude the TMG-servers. I recommended against this for the following reasons:

 

- The environment is big and can be volatile, if a TMG server is added it will manually need to be added to the group every time. This requires a lot of maintenance and has a high risk of monitoring-inconsistencies.

 

- Since there are multiple SCOM management groups, having a management pack with static assignments will break portability. Since most custom management packs are kept in sync across all MG’s, this is a big disadvantage.

 

I proposed to use an advanced dynamic query that would only populate the group with servers running the SQL-role, but not the TMG one.

 

In SCOM-XML, you can use the following tags to filter objects in a group query:

 

**Contains: **this tag can be used to indicate that objects containing a certain class (for example: Windows Computers that contain a SQL role object) should be selected.

 

**Contained: **this tag is the reverse of ‘contains’, it is used to indicate that objects contained within a certain class should be selected (for example: all objects that are hosted in a certain group).

 

This resulted in the following XML:

 <table cellpadding="2" cellspacing="0" border="0" width="400" ><tbody >     <tr >       
<td width="400" valign="top" >         

<RuleId>$MPElement$</RuleId>             
<GroupInstanceId>$MPElement[Name="Customer.SQL.MP.Group_SQLComputers"]$</GroupInstanceId>              
<MembershipRules>              
<MembershipRule>              
<MonitoringClass>$MPElement[Name="MicrosoftWindowsLibrary!Microsoft.Windows.Computer"]$</MonitoringClass>              
<RelationshipClass>$MPElement[Name="Customer.SQL.MP.Group_SQLComputersRelationShipType"]$</RelationshipClass>              
<Expression>              
**_Add all windows computers to this group that…_**              
<And>              
<Expression>              
**_…host the SQL server role…_**              
<Contains maxDepth="1">              
<MonitoringClass>$MPElement[Name="MicrosoftSQLServerLibrary!Microsoft.SQLServer.ServerRole"]$</MonitoringClass>              
</Contains>              
</Expression>              
<Expression>              
**_…AND do not host the TMG role._**              
<NotContains maxDepth="1">              
<MonitoringClass>$MPElement[Name="TMG!Microsoft.Forefront.TMG.ServerRole"]$</MonitoringClass>              
</NotContains>              
</Expression>              
</And>              
</Expression>              
</MembershipRule>              
<MembershipRule>              
<MonitoringClass>$MPElement[Name="SystemCenter!Microsoft.SystemCenter.HealthServiceWatcher"]$</MonitoringClass>              
<RelationshipClass>$MPElement[Name="Customer.SQL.MP.Group_SQLComputersRelationShipType"]$</RelationshipClass>              
**_Also add all health service watchers that are hosting a Windows Computer object…_**              
<Expression>              
<Contains>              
<MonitoringClass>$MPElement[Name="SystemCenter!Microsoft.SystemCenter.HealthService"]$</MonitoringClass>              
<Expression>              
<Contained>              
<MonitoringClass>$MPElement[Name="MicrosoftWindowsLibrary!Microsoft.Windows.Computer"]$</MonitoringClass>              
<Expression>              
**_…and where the hosted computer object was added by the previous membership rule._**              
<Contained maxDepth="1">              
<MonitoringClass>$Target/Id$</MonitoringClass>              
</Contained>              
</Expression>              
</Contained>              
</Expression>              
</Contains>              
</Expression>              
</MembershipRule>              
</MembershipRules>

      
</td>     </tr>   </tbody></table>  

 

Note that I used the ‘maxDepth’-attribute in the contains- and contained tags. This limits the recursive search of the query, improving performance.

 

Using advanced group query, selecting a subset from your SCOM-inventory within a group can become very precise and provide a set-and-forget experience!
