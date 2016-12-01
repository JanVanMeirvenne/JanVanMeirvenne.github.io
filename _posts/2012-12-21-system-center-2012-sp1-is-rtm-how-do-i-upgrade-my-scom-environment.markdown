---
author: janvanmeirvenne
comments: true
date: 2012-12-21 13:39:16+00:00
layout: post
link: http://scug.be/jan/2012/12/21/system-center-2012-sp1-is-rtm-how-do-i-upgrade-my-scom-environment/
slug: system-center-2012-sp1-is-rtm-how-do-i-upgrade-my-scom-environment
title: System Center 2012 SP1 is RTM! How do I upgrade my SCOM environment?
wordpress_id: 50
categories:
- '#scom'
- sysctr
---

Hey all!

I am pleased to announce that System Center 2012 SP1 has reached RTM and is now available for Technet and MSDN Subscribers!

I downloaded the binaries for SCOM and here I provide a quick list of steps on how to upgrade your RTM-version of SCOM 2012 to SP1.

First, make a good read of the [release notes](http://technet.microsoft.com/en-us/library/jj656651.aspx), so you know what issues or extra after-installation steps are present.



	
  * First, take a backup of your SCOM-databases. If the upgrade goes wrong your database is toast in most cases, and a restore is inevitable.

	
  * Okay, get the SP1 binaries on your first management server and launch setup.exe, a splash-screen appears:
[![image](http://scug.be/jan/files/2012/12/image_thumb.png)](http://scug.be/jan/files/2012/12/image.png)

	
  * Okay, the most obvious option is the one we want :‘Install’! We are greeted by an overview of the upgrade wizard-steps (because it detects an RTM installation of SCOM 2012) and an extra warning that we should backup our databases.
[![image](http://scug.be/jan/files/2012/12/image_thumb1.png)](http://scug.be/jan/files/2012/12/image1.png)

	
  * The next screen is the usual licensing text. After reading it through (all of it of course), lets continue!
[![image](http://scug.be/jan/files/2012/12/image_thumb2.png)](http://scug.be/jan/files/2012/12/image2.png)

	
  * Just plain and simple, where do you want to install the new binaries? Take a pick and go!
[![image](http://scug.be/jan/files/2012/12/image_thumb3.png)](http://scug.be/jan/files/2012/12/image3.png)

	
  * Ah, the both dreaded and loved prerequisite scan! And what’s this, we are missing something! This means a prerequisite was added for the service pack. We need the HTTP Activation feature, a sub feature of the .NET Framework 3.5.1 role. (Might be different or not needed on Server 2012)
[![image](http://scug.be/jan/files/2012/12/image_thumb4.png)](http://scug.be/jan/files/2012/12/image4.png)

	
  * No problem, we just install the prerequisite[![image](http://scug.be/jan/files/2012/12/image_thumb5.png)](http://scug.be/jan/files/2012/12/image5.png)

	
  * Lets recheck. Much better![![image](http://scug.be/jan/files/2012/12/image_thumb6.png)](http://scug.be/jan/files/2012/12/image6.png)

	
  * Next step, providing the SDK service account. I filled in the same one that was used for the RTM-installation.[![image](http://scug.be/jan/files/2012/12/image_thumb7.png)](http://scug.be/jan/files/2012/12/image7.png)

	
  * A summary? Are we done already? Wow, no effort at all (almost). Get ready, set, go & upgrade!
[![image](http://scug.be/jan/files/2012/12/image_thumb8.png)](http://scug.be/jan/files/2012/12/image8.png) [![image](http://scug.be/jan/files/2012/12/image_thumb9.png)](http://scug.be/jan/files/2012/12/image9.png)
[![image](http://scug.be/jan/files/2012/12/image_thumb10.png)](http://scug.be/jan/files/2012/12/image10.png)
And it actually did take this long…
[![image](http://scug.be/jan/files/2012/12/image_thumb11.png)](http://scug.be/jan/files/2012/12/image11.png)

	
  * But what about my second management server?
[![image](http://scug.be/jan/files/2012/12/image_thumb12.png)](http://scug.be/jan/files/2012/12/image12.png)
[![image](http://scug.be/jan/files/2012/12/image_thumb13.png)](http://scug.be/jan/files/2012/12/image13.png)
Oh oh…!
[![image](http://scug.be/jan/files/2012/12/image_thumb14.png)](http://scug.be/jan/files/2012/12/image14.png)
That looks nasty! Lets upgrade the second management server asap!
[![image](http://scug.be/jan/files/2012/12/image_thumb15.png)](http://scug.be/jan/files/2012/12/image15.png)
You know how the rest goes :)

	
  * After the 2nd upgrade: much better!
[![image](http://scug.be/jan/files/2012/12/image_thumb16.png)](http://scug.be/jan/files/2012/12/image16.png)

	
  * Now, the only thing left to do is upgrade the agents.
[![image](http://scug.be/jan/files/2012/12/image_thumb17.png)](http://scug.be/jan/files/2012/12/image17.png)


And that’s it, you’re done!

**TAKEAWAYS**



	
  * Do not forget to go through the release notes

	
  * New prerequisite (on 2008 R2): HTTP Activation (under .NET Framework 3.5.1)

	
  * Non-upgraded management servers cannot communicate with an upgraded management group

	
  * Overall, the upgrade is straightforward and foolproof


**This was my last blogpost for this year. Happy holidays and see you later!**
