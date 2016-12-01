---
author: janvanmeirvenne
comments: true
date: 2013-01-10 16:44:17+00:00
layout: post
link: http://scug.be/jan/2013/01/10/write-actions-and-the-workflow-simulator-dont-do-it/
slug: write-actions-and-the-workflow-simulator-dont-do-it
title: 'Write Actions and the workflow simulator: don’t do it'
wordpress_id: 66
categories:
- '#scom'
- '#sysctr'
---

I am currently working on a module which combines 2 submodules, which are both write actions (VBS scripts). The reason for this is that one part of the action that needs to be performed needs special credentials, while another part must be run under localsystem.

 

Special runas-profile WA –>   
** WA**      
Local System WA->

 

[![image](http://scug.be/jan/files/2013/01/image_thumb.png)](http://scug.be/jan/files/2013/01/image.png)

 

When I attached this to a rule and started a simulation of the workflow, things went haywire:[![image](http://scug.be/jan/files/2013/01/image_thumb1.png)](http://scug.be/jan/files/2013/01/image1.png)

 

Where are my modules? Why doesn’t it show anything but the scheduler-actions?

 

Well, apparently, this is a limitation of the simulator:

 

[![image](http://scug.be/jan/files/2013/01/image_thumb2.png)](http://scug.be/jan/files/2013/01/image2.png)

 

Stupid me! But at least I learned something of this!

 

The workaround for me is to directly perform a trace on the workflow while it is running inside the management group after I imported the management pack. This is slower, but at least it works!
