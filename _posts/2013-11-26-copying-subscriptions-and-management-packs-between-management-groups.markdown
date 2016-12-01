---
author: janvanmeirvenne
comments: true
date: 2013-11-26 16:46:27+00:00
layout: post
link: http://scug.be/jan/2013/11/26/copying-subscriptions-and-management-packs-between-management-groups/
slug: copying-subscriptions-and-management-packs-between-management-groups
title: 'Tool: copying subscriptions and management packs between management groups'
wordpress_id: 165
categories:
- '#scom'
- '#sysctr'
---

With SCOM setups it is not uncommon to have multiple management groups which adhere to the release management methodology. For example, you can have a development, QA or other non-production management group used for developing and testing management packs before throwing them in your production group. This is important as getting 1000+ alerts on a badly written MP will raise a lot of eyebrows in your ops department, so testing is key!

Since you might use subscriptions for [alert enrichment ](http://blog.scomfaq.ch/2012/04/17/scom-2012-using-alert-customfields/)through command channels or simply alert relevant application owners on issues, these need to be moved over as well when your monitoring solution is greenlighted for production.

I don't know about you, bit I dread repetetive manual labour! I can spend hours creating a Powershell script, drawing out a monitoring approach on paper or even create a set of dynamic groups in XML, but creating a bunch of subscriptions in the SCOM console, brrr...

I wouldn't be a scripting/programming geek if I didn't think of a way to alleviate this ailment :) therefore I present the SCOMCopier-tool!

What does this tool do? It copies a subscription, channel, subscriber or unsealed management pack between 2 management groups. It does this by taking the original item from the source management group, copying all properties to a new item and add it to the target management group. For management packs it just does an export/import.

The commandline:

"SCOMCopier <Source MS> <Target MS> (SourceCredentials <Source User> <Source Password>) (TargetCredentials <Target User> <Target Password> <ItemType> <ItemDisplayName>"

Description of the parameters:

- **SourceMS**: the management server of the source management group you wish to use
- **TargetMS**: the management server of the target management group you wish to use
- **SourceCredentials**: optional, announces that the following 2 parameters will be credentials for the source management group, if left out the running user's credentials are used
- **Source User**: optional, the source user account to use
- **Source Password**: optional, the source password to use
- **TargetCredentials**: optional, announces that the following 2 parameters will be credentials for the target management group, if left out the running user's credentials are used
- **Source User**: optional, the target user account to use
- **Source Password**: optional, the target password to use
- **ItemType: **the type of item you want to copy, can be either Subscription, Subscriber, Channel or ManagementPack
**- ItemDisplayName: **the (display)name of the item you want to copy, names with spaces must be enclosed with ".

You can download the tool here: [download id="1"]. As always, I am not responsible if this blows up your environment. I used all functions without causing damage, but you choose whether you'll trust me on this^;)

Questions, feedback and feature requests can be done in the comment section, although I can't guarantee the timely implementation of fixes and features (that's why the source code is included in the download :)).
