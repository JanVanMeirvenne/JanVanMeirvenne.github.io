---
author: janvanmeirvenne
comments: true
date: 2012-12-20 13:29:24+00:00
layout: post
link: http://scug.be/jan/2012/12/20/troubleshooting-can-not-attach-new-agents-to-a-management-group/
slug: troubleshooting-can-not-attach-new-agents-to-a-management-group
title: 'Troubleshooting: Can not attach new agents to a management group'
wordpress_id: 11
categories:
- '#scom'
- sysctr
tags:
- IFTTT
---

Some time ago I was asked for help by a fellow engineer who was troubleshooting a rather spicy SCOM-issue:

 

He had removed some agents from the management group because their server was reimaged to another OS. Afterwards he wanted to re-approve the agents, but instead of accepting the agents the RMS refused to let them connect. Even waiting for an entire night and only then approving the agents did not resolve the issue. After some time, the agents could neither be approved or declined, because the pending actions didn’t show up anymore. The RMS produced a lot of sickening events:

 

  
  * Event 20000: A device which is not part of this management group has attempted to access this Health Service. 
   
  * Event 21042: Operations Manager has discarded X items in management group xxxxx, which came from xxxx. These items have been discarded because no valid route exists at this time. This can happen when new devices are added to the topology but the complete topology has not been distributed yet. The discarded items will be regenerated. 
   
  * Many more distressing events 
 

I performed the standard diagnostics and recovery attempts (flushing the agent cache, denying the agents and then approve them,…) but to no avail. This is where I thought to myself ‘Lets backup the databases and go dirty’. In the end this where the steps that helped:

 

(BTW: This is NOT a supported procedure, so make a backup before touching anything!)

 

  
  * Stop all RMS services (Data Access, Configuration and Management)                      
      * Delete the health service state folder 
         
      * Start all 3 services back up 
              
 

These first steps did a lot to ‘unclog’ the RMS. The bad to good event ratio became a lot more balanced. This led me to believe that the RMS’ cache had become corrupt, and needed some cleaning to get all things running again.

 

However, did didn’t automagically fix the issue with adding agents. So I started doing some database actions:

 

  
  * [Delete](http://sethisageek.blogspot.be/2011/08/stalegray-objects-in-operations-manager.html) any traces of the problematic agents 
 <table cellpadding="2" cellspacing="0" border="0" width="400" ><tbody >     <tr >       
<td width="400" valign="top" >USE [OperationsManager]            
UPDATE dbo.[BaseManagedEntity]             
SET             
[IsManaged] = 0,             
[IsDeleted] = 1,             
[LastModified] = getutcdate()             
WHERE FullName like '%computername%'
</td>     </tr>   </tbody></table>  

  
  * Groom out the marked-for-removal items 
 <table cellpadding="2" cellspacing="0" border="0" width="400" ><tbody >     <tr >       
<td width="400" valign="top" >DECLARE @GroomingThresholdUTC datetime          

SET @GroomingThresholdUTC = DATEADD(d,-2,GETUTCDATE())

         

UPDATE BaseManagedEntity

         

SET LastModified = @GroomingThresholdUTC

         

WHERE [IsDeleted] = 1

         

UPDATE Relationship

         

SET LastModified = @GroomingThresholdUTC

         

WHERE [IsDeleted] = 1

         

UPDATE TypedManagedEntity

         

SET LastModified = @GroomingThresholdUTC

         

WHERE [IsDeleted] = 1

         

EXEC p_DataPurging

      
</td>     </tr>   </tbody></table>  

  
  * Remove [hidden](http://blogs.technet.com/b/kevinholman/archive/2008/09/29/agent-pending-actions-can-get-out-of-synch-between-the-console-and-the-database.aspx) pending actions 
 <table cellpadding="2" cellspacing="0" border="0" width="400" ><tbody >     <tr >       
<td width="400" valign="top" >exec p_AgentPendingActionDeleteByAgentName 'agentname.domain.com'
</td>     </tr>   </tbody></table>  

Okay, now everything related to our problematic agents was flushed from the database. And as I expected: another restart of the agents and some approvals later, the communication with the RMS was flawless and error free.

 

Although I have not an exact idea about how the database and/or RMS could get so confused, I suspect that at some point in time corruption introduced a series of sync-issues between the RMS and database.
