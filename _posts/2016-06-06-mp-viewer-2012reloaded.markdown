---
author: janvanmeirvenne
comments: true
date: 2016-06-06 21:42:58+00:00
layout: post
link: http://scug.be/jan/2016/06/06/mp-viewer-2012reloaded/
slug: mp-viewer-2012reloaded
title: MP Viewer 2012…reloaded!
wordpress_id: 482
---

One of the great things that comes forth out of a technology community are the custom tools. Tools that make life a little easier and provide functionalities not always found in the original product.

 

One of the tools I use very often is [MPViewer](https://blogs.msdn.microsoft.com/dmuscett/2012/02/19/boriss-opsmgr-tools-updated/). A simple tool that can read and report on management pack contents without needing a management group connection or other heavy interface. Just run this little gem, point to a MP / XML / MPB Management Pack, and boom…you see what’s inside. It is a great value to quickly create a report of do a pre-import assessment.

 

Big was my disappointment when I heard that the original authors [abandonded the project](https://blogs.msdn.microsoft.com/dmuscett/2015/09/15/so-long-and-thanks-for-all-the-fish/).

 

Since the original code was shared on [Github](https://github.com/dani3l3/mpviewer) (thank god!) I decided to use my own developer knowledge to research, maintain and extend the code. I got to the point were I am confident to do a [first release](https://github.com/JanVanMeirvenne/mpviewer/releases/tag/R1) to the world, and I hope you have as much fun using it as I had creating it. Bear in mind though that I am not a professional developer, so the ride may be bumpy and dirty from time to time.

 

## 

 

# New Features

 

  
  * **The MPViewer can now open multiple management packs in one go, allowing easier documentation of an entire workload instead of separate files**
        
    * Each item will have a column stating the MP it resides in, so you will always know the source MP!
     
    * A separate table ‘Management Packs’ shows all the MP’s that are loaded
    

[![image](http://scug.be/jan/files/2016/06/image_thumb.png)](http://scug.be/jan/files/2016/06/image.png)

 

[![image](http://scug.be/jan/files/2016/06/image_thumb1.png)](http://scug.be/jan/files/2016/06/image1.png)

 

  
  * **Management Packs can now be loaded from a live Management Group in addition of files. Multiple MP’s can be selected, just like the file-based approach!**
 

[![image](http://scug.be/jan/files/2016/06/image_thumb2.png)](http://scug.be/jan/files/2016/06/image2.png)

 

[![image](http://scug.be/jan/files/2016/06/image_thumb3.png)](http://scug.be/jan/files/2016/06/image3.png)

 

  
  * **Modules are now included in the overview, including the full XML configuration!**
 

[![image](http://scug.be/jan/files/2016/06/image_thumb4.png)](http://scug.be/jan/files/2016/06/image4.png)

 

  
  * **OpenWith is now supported: associate the MPViewer with the XML, MP or MPB file extension and MPViewer will load the file when you double-click on it in explorer!**
 

## 

 

## Known Issues

 

  
  * First release, so expect some bugs here and there. Feedback is welcome through the [issues-page](https://github.com/JanVanMeirvenne/mpviewer/issues)
   
  * Command-Line support removed due to the addition of the OpenWith feature. I will research the possibility of combining both features
   
  * GUI is quirky at some points
   
  * Code is a mess in some areas
 

# [Download](https://github.com/JanVanMeirvenne/mpviewer/releases/latest)

 

Thank you in advance for giving the tool a spin and providing your own contributions and feedback! I hope to be able to keep this small gem alive and up-to-date so that there will be a continued benefit for those in need of a good Management Pack tool!

 

Jan Out!
