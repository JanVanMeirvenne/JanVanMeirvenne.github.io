---
author: janvanmeirvenne
comments: true
date: 2014-01-13 18:12:53+00:00
layout: post
link: http://scug.be/jan/2014/01/13/detecting-and-fixing-corrupt-subscriptions/
slug: detecting-and-fixing-corrupt-subscriptions
title: Detecting and fixing corrupt subscriptions
wordpress_id: 173
categories:
- '#scom'
- '#sysctr'
---

We will all encounter it at least once in our SCOM career: [the dreaded corrupt subscription bug](http://blogs.technet.com/b/randymonteleone/archive/2010/12/17/scom-notifications-guid-should-contain-32-digits-with-4-dashes-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-console-error.aspx).

What is this bug and how can you reproduce it?

It is pretty simple: when you initially create a subscription with some alert rules or monitors specified in the criteria, and afterwards you delete one of the specified rules or monitors, you are in trouble :) Strangely, when doing the same with a group criterium the subscription does get modified cleanly without corruption occuring.

SCOM won't show or run the subscription. This can be an issue that won't be noticed for a long time as long as no one opens the subscription or starts to wonder why there are so few notifications lately. Sometimes, no news = bad news!

The solution is to manually export the notifications management pack, sweep through its xml inards and weed out the guids of the items that no longer exists, and then reimport the altered xml. This can be a tedious process prone to error and frustration.

I decided to see what could be done to automate this process. This tool is the result: the Scom Subscription Fixer!

You launch this CLI tool by only providing the name of a (root) management server. The tool will perform the following actions (provided that you are using administrator credentials):

- check each subscription's specified monitors and rules against the SCOM inventory. Valid items are reported as well.
- when an invalid item is found you are notified and can choose to remove the reference from the subscription
- additionally, you are warned when no criteria is specified for a subscription, meaning the subscription triggers on every alert
- when a reference deletion causes the criteria to become empty, you are notified and have the option to delete the entire subscription.
- The fixes are applied without the need for export/import actions. The subscriptions will be available again minus the corrupted items of course.

The tool uses only the Scom SDK with some xml trickery to keep the subscription xml valid. Feel free to reply to this post when experiencing bugs or other animals, I will provide an update as the need arises. As always, the source code is included so feel free to make the tool more awesome ;)

[SCOM Subscription Fixer](http://scug.be/jan/files/2014/01/SCOM-Subscription-Fixer.zip)
