---
author: janvanmeirvenne
comments: true
date: 2013-07-02 15:57:52+00:00
layout: post
link: http://scug.be/jan/2013/07/02/objects-in-scom-do-not-disappear-when-removing-an-enable-override-from-a-default-disabled-discovery/
slug: objects-in-scom-do-not-disappear-when-removing-an-enable-override-from-a-default-disabled-discovery
title: Objects in SCOM do not  disappear when removing an enable-override from a default
  disabled discovery
wordpress_id: 157
categories:
- '#scom'
- '#sysctr'
---

**Symptom:**

 

If you have a discovery which is disabled by default and you remove the override that enabled it, objects that where discovered are not removed. If you run ‘remove-disabledmonitoringobject’ or ‘Remove-SCOMDisabledClassInstance’ (depending on your version of SCOM) the objects remain.

 

**Cause:**

 

This is a design ‘flaw’. Only objects discovered by natively enabled discoveries are scrubbed by the grooming process. If a discovery is disabled by default any objects it created won’t get purged.

 

**Resolution:**

 

The easiest solution is to create a temporary override for the discovery in which you enforce the disabled setting by setting the enabled-property to false and ticking the ‘enforced’ checkbox. If the override is active you can rerun the PowerShell commands mentioned above to successfully groom the stale objects from the database. After this is done you can remove the override.
