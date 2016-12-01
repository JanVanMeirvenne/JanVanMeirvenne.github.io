---
author: janvanmeirvenne
comments: true
date: 2015-06-15 19:40:56+00:00
layout: post
link: http://scug.be/jan/2015/06/15/scom-data-warehouse-troubles-2-the-missing-objects/
slug: scom-data-warehouse-troubles-2-the-missing-objects
title: 'SCOM data warehouse troubles #2: The missing objects'
wordpress_id: 298
---

The previous week I noticed that my customer's reports were missing a lot of data in terms of recently added servers and their underlying objects. turns out they didn't exist in the data warehouse at all, while they were certainly a couple of days old.

I troubleshooted the issue and found that there was a conflict between 2 tables in the data warehouse, effectively blocking the entire syncing process of SCOM!

You can read my adventure including the happy ending [here](http://www.jvm-net.com/?p=1434)
