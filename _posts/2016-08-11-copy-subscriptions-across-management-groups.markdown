---
author: janvanmeirvenne
comments: true
date: 2016-08-11 22:06:51+00:00
layout: post
link: http://scug.be/jan/2016/08/11/copy-subscriptions-across-management-groups/
slug: copy-subscriptions-across-management-groups
title: Copy subscriptions across management groups
wordpress_id: 493
---

It is common to have multiple SCOM management groups in an organization. For example a test MG to stage changes and a production one for day-to-day operations. If your monitoring processes rely heavily on SCOM subscriptions (for automation with command channels, or just mail for example), it might be tedious to keep them in sync across all the different management groups. Creating or copying subscriptions by GUI is click-intensive and prone to error. By automating this part of the monitoring process, it can greatly speed-up and simplify things.

 

A customer of mine relies on subscriptions for integration with other systems. Per monitored service, a subscription exists that passes all relevant service alerts through an executable that performed a part of the integration. Just like the service monitoring itself, the subscription is created in a test environment, validated and then copied to production. The script I am showcasing has limited (only single-channel mail- or command channel based subscriptions are supported) but worry-free capabilities for quickly copying a subscription. This is done by passing the to-copy subscription’s displayname, a management server for the source management group and one for the target management group. For now, you need to run the script with an account that has administrator rights in both locations. A nice feature is that if the source subscription already exists in the target MG, it will only sync the configuration without fully deleting or recreating it!

 

### [Check it out on GitHub](https://github.com/JanVanMeirvenne/SCOMRunbooks/blob/master/SCOMRunbooks/Functions/Copy-cSCOMSubscription.ps1)

 

I hope it can be of use!

 

Jan out
