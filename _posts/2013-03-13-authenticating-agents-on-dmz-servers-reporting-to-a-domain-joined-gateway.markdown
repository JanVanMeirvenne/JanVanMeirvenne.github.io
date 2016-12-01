---
author: janvanmeirvenne
comments: true
date: 2013-03-13 17:04:42+00:00
layout: post
link: http://scug.be/jan/2013/03/13/authenticating-agents-on-dmz-servers-reporting-to-a-domain-joined-gateway/
slug: authenticating-agents-on-dmz-servers-reporting-to-a-domain-joined-gateway
title: Authenticating agents on DMZ-servers reporting to a domain-joined gateway
wordpress_id: 79
categories:
- '#scom'
- '#sysctr'
---

 

I heard of this issue when one of my colleagues was attaching a customer to our centralized SCOM infrastructure:

 

**SCENARIO**

 

A SCOM gateway is deployed and domain-joined on the customer site. It is provisioned with a root and client certificate to allow communication with the central platform.

 

However, the customer has DMZ-systems which are not joined to a domain. When they are provisioned with SCOM agents which are pointed to the gateway, they fail to communicate:

 

Event on the agent:

 <table cellpadding="2" width="400" border="1" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

Log Name: Operations Manager             
Source: OpsMgr Connector              
Date:   
Event ID: 20070              
Task Category: None              
Level: Error              
Keywords: Classic              
User: N/A              
Computer: XXXXXX.domain.com              
Description:              
The OpsMgr Connector connected to XXXXXXXXX.com, but the connection was closed immediately after authentication occurred. The most likely cause of this error is that the agent is not authorized to communicate with the server, or the server has not received configuration. Check the event log on the server for the presence of 20000 events, indicating that agents which are not approved are attempting to connect.              
see below error on agent opsmgr event log:

      
</td>     </tr>   </tbody></table>  

Events on the gateway:

 <table cellpadding="2" width="400" border="1" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >A device at IP XX.XX.XX.XX attempted to connect but could not be authenticated, and was rejected.
</td>     </tr>   </tbody></table>  

 

**CAUSE**

 

The probable cause is the gateway cannot use kerberos or certificates to provide authentication to the DMZ-agents and thus refuses to allow communication with the management group.

 

**SOLUTION**

 

The solution was to import the same root certificate which was used for the gateway in the trusted root certificate store of the DMZ-servers. This allowed a secure link to be established with the gateway server and the communication with the management group to commence.
