---
author: janvanmeirvenne
comments: true
date: 2013-01-03 16:36:58+00:00
layout: post
link: http://scug.be/jan/2013/01/03/scom-troubleshooting-constant-high-cpu-usage-of-monitoringhost-exe-and-event-3006/
slug: scom-troubleshooting-constant-high-cpu-usage-of-monitoringhost-exe-and-event-3006
title: 'SCOM Troubleshooting: constant high cpu usage of monitoringhost.exe and event
  3006'
wordpress_id: 53
categories:
- '#scom'
- '#sysctr'
---

 

**Symptoms**

 

On a couple of Exchange 2010 servers monitored by Operations Manager, a constantly high CPU usage is reported for the monitoringhost.exe process. Restarting the agent including clearing the cache do not fix the issue. Meanwhile, the application log is being flooded with events with id 3006 and source EvntAgnt.

 

**Solution**

 

The solution is to restart the SNMP service on the affected machine, this will stop the eventlog-flooding and drop the monitoringhost’s CPU usage to normal levels.

 

The exact same issue and solution where [blogged](http://blogs.catapultsystems.com/cfuller/archive/2008/12/30/application-log-filling-with-eventid-3006-from-source-evntagnt.aspx) a long time ago by Cameron Fuller for a Server 2003 system, but the issue can apparently also occur on a Server 2008 R2 SP1 machine.
