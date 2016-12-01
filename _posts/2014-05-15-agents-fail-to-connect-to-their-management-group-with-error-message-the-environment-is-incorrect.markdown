---
author: janvanmeirvenne
comments: true
date: 2014-05-15 16:20:10+00:00
layout: post
link: http://scug.be/jan/2014/05/15/agents-fail-to-connect-to-their-management-group-with-error-message-the-environment-is-incorrect/
slug: agents-fail-to-connect-to-their-management-group-with-error-message-the-environment-is-incorrect
title: Agents fail to connect to their management group with error message “The environment
  is incorrect”
wordpress_id: 262
categories:
- '#scom'
- '#sysctr'
---

#### ****

 

**Symptoms**

 

- Agents stop receiving new monitoring configuration      
- When restarting the agent service, the following events are logged in the Operations Manager event log:

 

[![clip_image001](http://scug.be/jan/files/2014/05/clip_image001_thumb.jpg)](http://scug.be/jan/files/2014/05/clip_image001.jpg)

 

[![clip_image002](http://scug.be/jan/files/2014/05/clip_image002_thumb.jpg)](http://scug.be/jan/files/2014/05/clip_image002.jpg)

 

[![clip_image003](http://scug.be/jan/files/2014/05/clip_image003_thumb.jpg)](http://scug.be/jan/files/2014/05/clip_image003.jpg)

 

**Cause**

 

This indicates that the agent can not find the SCOM connection information in AD. This is usually because it is not permitted to do so.

 

**Resolution**

 

All connection info is found in the Operations Manager container in the AD root. If you do not see it using “Active Directory Users and Computers” then click “View” and enable “Advanced Features”.

 

[![image](http://elgwhoppo.files.wordpress.com/2012/07/image_thumb2.png?w=612&h=414)](http://elgwhoppo.files.wordpress.com/2012/07/image2.png)       
     
(Screenshot taken from [http://elgwhoppo.com/2012/07/25/scom-2012-ad-integration-not-populating-in-ad/](http://elgwhoppo.com/2012/07/25/scom-2012-ad-integration-not-populating-in-ad/))

 

The container will contain a subcontainer for each management group using AD integration. In the subcontainer there are a set of SCP-objects, containing the connection information for each management server, and 2 security groups per SP: PrimarySG… and SecondarySG…. These groups will be populated with computer objects using the LDAP queries you provided in the AD integration wizard of the SCOM console. So for example if your LDAP query specifies only servers ending with a 1, only those objects matching the criteria will be put in the group.

 

These security groups should normally both have read-access on their respective SCP-object (eg for management server “foobar” the groups with “PrimarySG_Foobar…” and “SecondarySG_Foobar…” should have read access on the SCP-object for this management server.

 

If the security is correct the agent can only see the SCP-objects to which it should connect in a normal and failover situation.

 

If these permissions are not correct then you can safely adjust them manually (only provide read access). The agents will almost immediately pick up the SCP once they have permission. If this is not the case, restart the agent service. 
