---
author: janvanmeirvenne
comments: true
date: 2013-09-24 17:19:30+00:00
layout: post
link: http://scug.be/jan/2013/09/24/standardizing-overrides/
slug: standardizing-overrides
title: Standardizing overrides
wordpress_id: 161
categories:
- '#scom'
- '#sysctr'
---

Referring back to [an old post about SCOM and MOF](http://janscman.wordpress.com/2012/11/06/using-mof-as-management-framework-for-operations-manager/), SCOM is a (awesome) tool to monitor your environment, but is a glorified logging tool without a process-framework to tie it to.

 

During my long-term assignment at a large customer, I got tasked with providing a delegation of the ability to define ’monitoring modifications’ towards the second line support unit.

 

At this point, all overrides were controlled by us and people just had to send us an email to get their modification. There was no guideline, no review, just clickclick and done. This had the following risks:

 

- Wild growth of (mostly undocumented) overrides     
- Second line did not have insight into a server’s actual monitoring configuration when handling tickets      
- No lifecycle management: are overrides made a year ago still needed etc?

 

Since SCOM is going to be the main Wintel monitoring tool, this had to change. I had several meeting with second line to define a process that encompassed transparency, clarity and efficiency. The pilot target of this process in development would be disk monitoring.

 

First, we had to clear a way through the bushes: get rid of the dozens of wild overrides and cast them into a shape that was clearly documented. We decided to define several ‘templates’ for disk monitoring:

 

- DEFAULT (no overrides)     
- LOW (Warning at 5% or 500Mb free space, Critical at 2% or 200Mb free space)      
- HIGH (Warning at 15% or 2000Mb free space, Critical at 10% or 1000Mb free space)      
- HIGH-2 (still unsure about this name) (Warning at 25% or 10000Mb free space, Critical at 20% or 5000Mb free space)

 

These were the ‘override sets’ we defined. Depending on the type of disk (system disk, temp disk, scratch disk, fileserver disk,…) it would get assigned to one of these profiles. To facilitate this, we made 3 groups, one for each profile (except the default one), and scoped the overrides to them.

 

This would clear up a lot of confusion: only 3 sets of overrides were possible. Second line also uses these profiles to discuss noisy disks with application owners and advises them on one of these 3 profiles, no exceptions are allowed.

 

Because SCOM isn’t very granular in what an user can do and what not, it wasn’t possible to make second line able to modify the groups without giving them full administrator rights.

 

We decided to create an excel on SharePoint where second line could request overrides in batch by stating the server, disk(s) and profile to assign. We (the SCOM team) would then bi-weekly process the requests (gradually creating the overrides in the testing environment and then propagate them to production) and log the action in our action list, which is a SharePoint list used to log SCOM modifications. The action id that is created by this then gets logged in the excel and the status of the requests is set to fulfilled. The version of the override management pack (which only contains disk overrides and the profile groups) is increased and in its product knowledge article a reference to the action id is made as well.

 

Finally, a message is sent to second line that their requests were processed, after which they validate the overrides. The important thing is that every override is logged in the excel, and references in the management pack and action list make sure that it is known when and by who an override was applied. This allows us to keep a record of all overrides without the need to create a report or view in SCOM, which is often not very exact about which system / disk receives an override.

 

Finally, I proposed one of the key items of the SMC aspect of MOF: the operational health review. A monthly meeting will be held between second line and the SCOM team to talk about new and ongoing issues with alerting, including overrides. This way overrides can be reviewed on validity, effectiveness and usefulness. While the SCOM team might know their platform the best, it is the second line that works with its output every day and experiences first hand what is working properly and what not.

 

This process is still ‘in the works’, but the benefits are already very noticeable. The next step is to finalize the process in an official document and communicate it to all stakeholders. Afterwards, this process will be extended to other monitoring, like CPU and Memory.

 

While this may sound a but exhaustive to implement, here are some basic best practices I use regarding overrides, which is a good start to keep things clean:

 

- Do not place overrides on specific objects, but make a group which receives the override and put the objects into that one. It will prevent forgotten overrides and gives you a central point of control for the specific override.

 

- Document the override, you can use the comment section while making the override, the product knowledge of the override MP, or just store the information in an excel or notepad. Review these list from time to time and see what’s still applicable and what not.

 

- Use tools like [Effective Configuration Viewer](http://www.microsoft.com/en-us/download/details.aspx?id=6742), [MPTuner](http://mpwiki.viacode.com/default.aspx?g=mptuner) or if you want an enterprise solution, [MP Studio](http://www.silect.com/products/mp-studio) to more easily navigate and document the overrides present in your environment.

 

I hope to have provided some ideas and insight in how to simplify your SCOM management, enjoy! Jan out.   
