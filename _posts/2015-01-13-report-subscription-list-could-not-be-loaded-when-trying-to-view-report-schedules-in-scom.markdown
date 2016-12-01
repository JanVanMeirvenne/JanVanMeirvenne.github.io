---
author: janvanmeirvenne
comments: true
date: 2015-01-13 12:59:13+00:00
layout: post
link: http://scug.be/jan/2015/01/13/report-subscription-list-could-not-be-loaded-when-trying-to-view-report-schedules-in-scom/
slug: report-subscription-list-could-not-be-loaded-when-trying-to-view-report-schedules-in-scom
title: '"Report subscription list could not be loaded" when trying to view report
  schedules in SCOM'
wordpress_id: 284
---

A small year ago, I performed a very troublesome upgrade from SCOM 2007 to SCOM 2012 on a large company site. One of the issues forced us to reinstall the SCOM reporting component. In an attempt to retain the reports I backed up and restored the report server databases after the reinstall.

We did not use scheduled reports for a long time, that's why the problem surfaced only when an application owner asked for a periodic performance report. When trying to open the 'Scheduled Reports' view in the reporting pane of the console, I got the following error (the screenshot is from SCOM 2007, but the problem also can occur in SCOM 2012):

![](http://microsofttouch.fr/resized-image.ashx/__size/550x0/__key/communityserver-blogs-components-weblogfiles/00-00-00-00-16/SCOMScheduledReportSMTP_2D00_01.png)

After long trial and error and comparing settings with a fully functional reporting setup, I found the issue:

When opening the problematic view in the console, SCOM queries the 'Subscriptions' table in the reporting server database. Apparently, some entries were corrupted during the restore as some fields that sounded important like 'Report Deliver Extension' where blank. SCOM probably does not expect to have blanks returned, resulting in the aforementioned error.

I suspect that this might have been fixable, but because I had much on my todo-list and this was the first subscription needed on the report server, I deleted everything in the subscriptions table (present in the Report Server database):

**_delete  from Subscriptions
_**_(note that this is probably unsupported and might be a showstopper when needing Microsoft support afterwards!)_

After this action, the console could open the schedule-view without issues, and when I created a new schedule using the console, it appeared in the view.

I don't suspect this is an issue you will encounter on a normal operational day, but if you were having a rough upgrade as well I hope this helps you out!
