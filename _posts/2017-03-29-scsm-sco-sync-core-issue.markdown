---
layout:     post
title:      "SCSM 2016/Server 2016 Core: SCO connector sync issue"
subtitle:   "Possible bug with workaround"
date:       2017-03-29 12:00:00
author:     "Jan Van Meirvenne"
comments: true
---

I recently did a migration to Service Manager 2016 on a customer site. This was a side-by-side migration with a brand new management group and set of servers, with a PS-based migration of tickets.

The cool thing about the setup was that we used Server Core installations and configured everything with DSC! (More on that in a later post). Migrating the content was pretty easy aside from some code changes to get the form customizations going.

I noticed one particular issue that didn't seem to be around for SCSM 2012 or SCSM 2016 on Server GUI:

when setting up the SCO connector, the runbooks are not synchronized, even when pushing the 'synchronize now' button.

<img src="{{ site.url }}/assets/scsm-scosync/SCSM.png" />

After much troubleshooting, I had to abandon the issue for a while. But, when I came back on-site after a week, everything was OK. Apparently, the scheduled sync (at midnight) is working perfectly, but the on-demand one isn't.

I don't know exactly if this is a bug with SCSM 2016, either globally or in combination with Server Core 2016. After investigating the callstack behind the button 