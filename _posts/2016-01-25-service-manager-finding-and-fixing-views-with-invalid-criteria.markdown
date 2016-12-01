---
author: janvanmeirvenne
comments: true
date: 2016-01-25 08:24:09+00:00
layout: post
link: http://scug.be/jan/2016/01/25/service-manager-finding-and-fixing-views-with-invalid-criteria/
slug: service-manager-finding-and-fixing-views-with-invalid-criteria
title: 'Service Manager: finding and fixing views with invalid criteria'
wordpress_id: 382
---

 

With every Service Manager project, there are some free addons I almost always implement together with the platform. Posting an overview blog with a ‘best of’ overview of these tools is on my to-blog list (which gets usually longer than shorter over time alas).

 

One of these tools is the [Advanced View Editor](https://gallery.technet.microsoft.com/Advanced-View-Editor-20-377353f5), a small addon that allows you to define views much more specific (column names, more criteria) than the standard view editor. There is a [pro version](http://itnetx.ch/products/itnetx-advanced-view-editor-for-scsm/) which contains even more features, but usually the free edition is sufficient for the job.

 

The tool has the useful feature to allow direct XML editing of the criteria instead of using the common GUI:

 

[![clip_image001](http://scug.be/jan/files/2016/01/clip_image001_thumb.png)](http://scug.be/jan/files/2016/01/clip_image001.png)[![clip_image002](http://scug.be/jan/files/2016/01/clip_image002_thumb.png)](http://scug.be/jan/files/2016/01/clip_image002.png)

 

However, there is one downside to this: should the criteria of a view become invalidated (usually an enumeration that is deleted but is still specified in the view), then the editing mode will only allow XML-based modifications. This can be confusing if the service administrator is not familiar with XML, especially as it is not clear why the view is deemed invalid. The regular view editor (built-in in the console) can show the missing enumeration, but doesn’t work well with views created by the addon (I usually recommend to only use the AVE editor or the regular one, but not both).

 

The best way to prevent such an issue is to get your service management processes straight before implementing them in SCSM. Changing workitem categories (incident area / resolution / …) wreak havoc on both reporting and scoping and thus poses a severe impact on the efficiency of the platform. However, these cases can not always be prevented as company organisations and politics change over time, bringing new structures in play that were not foreseen during the initial design of the processes.

 

How do you know I am impacted by this issue? Well, when you open a view in the AVE, you will get this message when you go to the criteria-section, not allowing you to use the GUI-mode editor:

 

[![image](http://scug.be/jan/files/2016/01/image_thumb.png)](http://scug.be/jan/files/2016/01/image.png)

 

_Remember however, you will notice the issue faster with the AVE addon, but a regular view will be impacted just the same, showing a blank field in the criteria-GUI were an invalid item is present_.

 

How do you fix it? Well, you’ll need to check the XML and identify which enumerations are used in the criteria. An enumeration is an item based in a SCSM list (eg Incident Area), and is represented internally as a GUID.

 

 

[![image](http://scug.be/jan/files/2016/01/image_thumb1.png)](http://scug.be/jan/files/2016/01/image1.png)

 

You then need to use the Powershell-based [smlets](http://smlets.codeplex.com/) to connect to the SCSM management and enter the following command: ‘Get-SCSMEnumeration –Id ENUMERATIONIDFROMXML’

 

Once you get an error, or nothing returned, then you found the culprit of your problem. However, take into account that there can be multiple missing enumerations for a view, so it is best to check each one before considering the job done. when you identified all missing enums, then you need to delete them from the XML-code. Keep in mind that if you have an and/or clause in your criteria then you might need to remove the <and>/<or> expression and replace it with a regular expression.

 

[![image](http://scug.be/jan/files/2016/01/image_thumb2.png)](http://scug.be/jan/files/2016/01/image2.png)

 

would become

 

[![image](http://scug.be/jan/files/2016/01/image_thumb3.png)](http://scug.be/jan/files/2016/01/image3.png)

 

‘That’s all nice and sweet, but do you really expect me to go over each view and check if I am impacted? I have over a hundred views!!!’

 

Well, first, you’re views will keep working, so there is no immediate threat to the end-user experience. So this only becomes a problem when you need to perform changes to the views. However, since you once set the criteria you expect a certain set of workitems to return. When that criteria becomes invalid some work-items that you need to show up in a certain view might just not be there, increasing the risk of having a blind spot. A blind spot is something you don’t want in a service desk tool!

 

So, to allow you to quickly pinpoint impacted views, I created a script that scans all views that have a criteria, loops over all the guids and tests that they exist as an enum in the Service Manager lists. It then outputs any views that need fixing in the following manner:

 

**_Missing guid '{a12f6e91-d223-4741-3ca9-41088c314b84}' in view 'Microsoft Outlook' of type 'Problem' in folder-path 'Work Items/Problem Management - SEC/Problem per Service/Communication/'_**

 

Ok, nice! Now you not only know which views are impacted, but also their relative location in your console-structure! It is still monkey-work to fix the views, but for now I won’t add a ‘autofix’ feature in the script as it is hard to intelligently perform without risking breaking the views alltogether.

 

You can find the script in my github account: [https://github.com/JanVanMeirvenne/SCSMRunbooks/blob/master/Runbooks/Get-cSCSMInvalidView.ps1](https://github.com/JanVanMeirvenne/SCSMRunbooks/blob/master/Runbooks/Get-cSCSMInvalidView.ps1)

 

Note: you need to have smlets (and thus the SCSM console) installed on the computer from where you want to run the script + the necessary credentials to access the SCSM management group.

 

Feel free to contact me by mail ([jan@jvm-net.com](mailto:jan@jvm-net.com)) should you have any questions on this matter!

 

Untill next time! Jan Out. 
