---
author: janvanmeirvenne
comments: true
date: 2015-11-01 22:49:32+00:00
layout: post
link: http://scug.be/jan/2015/11/01/oms-overview-chapter-1-introduction/
slug: oms-overview-chapter-1-introduction
title: 'OMS overview: chapter 1– introduction'
wordpress_id: 320
tags:
- '#msoms #sysctr'
---

Hi all!

 

Before I dabble you under in the realm of Microsoft cloud platform management, I first want to bother you with some personal announcements!

 

It has been a long time time since I posted consistently for a while now, but I have some perfect excuses for this:

 

First off, I got married all the way back in April to my now wonderful wife Julie!

 

[![image](http://scug.be/jan/files/2015/11/image_thumb.png)](http://scug.be/jan/files/2015/11/image.png)

 

And if that was not enough of a life achievement, meet my son Julian, born in August (yes, Julie is the mother ![Winking smile](http://scug.be/jan/files/2015/11/wlEmoticon-winkingsmile.png))! This is one of the rare pictures where he smiles because he likes to keep things very serious generally.

 

[![image](http://scug.be/jan/files/2015/11/image_thumb1.png)](http://scug.be/jan/files/2015/11/image1.png)

 

And to complete this combo-high score, we will soon be on the lookout for our own home where we can develop ourselves as a family and live happily ever after!

 

Despite all of this I am still dedicated to acquire, produce and share knowledge regarding the Microsoft cloud technologies and while new balances will of course need to be sought, I pledge to continue this passion both on- and offline, whether at customer sites or community events! So let’s kick the tires again and start off with an introduction to the newborn cloud management platform, [Operations Management Suite](http://www.microsoft.com/en-us/server-cloud/operations-management-suite/overview.aspx)!

 

# 

 

# OMS Blog Series Index

 

Since there is much to say about each of the Operations Management Suite aka OMS I will break up this post in a blog series:

 

**Chapter 1: Introduction to OMS (this post)        
[Chapter 2: Disaster Recovery with OMS](http://scug.be/jan/2015/11/02/oms-overview-chapter-2-disaster-recovery/)        
Chapter 3: Backup with OMS         
Chapter 4: Automation with OMS         
Chapter 5: Monitoring and Analysis with OMS         
Chapter 6: Conclusion and additional resources overview**

 

This series is actually a recap of a live presentation I gave on the last [SCUG event](http://scug.be/events/2015/09/30/scugbe-system-center-night-fall-edition-2210/) which was shared with another session on SCOM – SCCM better together scenario’s presented by SCUG colleagues [Tim](https://twitter.com/Tim_DK) and [Dieter](https://twitter.com/DieterWijckmans). You can find my slides [here](http://www.slideshare.net/JanVanMeirvenne/oms-overview-54622885), demo content that is shareable (the ones I don’t need to pay for ![Smile](http://scug.be/jan/files/2015/11/wlEmoticon-smile.png)) will be made available in the applicable chapters later on.

 

## 

 

## 

 

# Chapter 1: Introduction

 

So what is OMS? Well, it is the management-tool answer to the [hybrid cloud scenario](http://www.microsoft.com/en-us/server-cloud/solutions/hybrid-cloud.aspx)

 

[![image](http://scug.be/jan/files/2015/11/image_thumb2.png)](http://scug.be/jan/files/2015/11/image2.png)

 

The hybrid cloud scenario entails a synergy between both your on-premise platforms, Microsoft cloud technologies like Azure and O365 and any 3rd party cloud platforms you might consume like Amazon for example.

 

This kind of ‘cloud of clouds’ is emerging everywhere and there are many companies that are already using cloud-based services today. This scenario provides a highly elastic and flexible way of working: quickly spin up additional business app instances in the cloud on-demand or have a full DR site ready to go with the push of a button, all with just the swipe of a credit card, and this are just 2 examples! However, a nice quote I read somewhere comes into play: ‘As the thing become easier on the front-end, as hard do they become on the back-end’. Essentially, things are easy when they are just contained on one platform and in one place. The hybrid cloud smashes this ideal by dictating that services should be deliverable through any platform from anywhere. This raises the important question: how do I distribute my services across all these platforms running in various locations while still being able to have my single pane of glass?

 

If you answer ‘System Center’ you are right, but partially, due to the following reasons:

 

- The System Center tools were born for on-prem management and although they can interface with cloud and cross-platform technology, their core platform is and will for now remain Windows Server, an on-prem platform.

 

- One of the goals of the hybrid cloud is to achieve hyper-scale: being able to spin up service instances in a matter of seconds. The kind of data that needs to be managed might be overwhelming for the current System Center tools. Have you checked your SCOM DW size and performance recently? Or did performance tests on your Orchestrator runbooks on high-volume demands? Or tried managing both Azure and Hyper-V VMs as a single unit? Don’t get me wrong, they can cope, but as these platforms were designed for on-prem scenario’s they can not always provide the hyper-elasticity and agility that their targets impose nowadays.

 

‘So what are you trying to say? That System Center is becoming an obsolete relic?’ Hell no! But just as the managed platforms evolve, so must the management ones do! This is why OMS has been developed, to close the gap, and not replace but extend the System Center story into the cloud.

 

Actually OMS is nothing new under the sun. It is just like its older brother [EMS](http://www.microsoft.com/en-us/server-cloud/enterprise-mobility/overview.aspx) (cloud-based workplace management) a competitively priced bundle of Azure-based services which form a single management platform together. When you open the OMS site ([www.microsoft.com/oms](http://www.microsoft.com/oms), one of the easiest URLs ever) for the first time you’ll see this:

 

[![image](http://scug.be/jan/files/2015/11/image_thumb3.png)](http://scug.be/jan/files/2015/11/image3.png)

 

Pretty abstract, I agree, but in fact the technology hiding behind these concepts are very simple and straightforward:

 

[![image](http://scug.be/jan/files/2015/11/image_thumb4.png)](http://scug.be/jan/files/2015/11/image4.png)

 

**MAPPING**

 <table cellpadding="2" width="862" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="440" valign="top" >**_Concept_**
</td>        
<td width="420" valign="top" >**_Technology_**
</td>     </tr>      <tr >       
<td width="450" valign="top" >Backup & Recovery
</td>        
<td width="425" valign="top" >[Azure Backup](https://azure.microsoft.com/en-us/services/backup/) / [Azure Site Recovery](https://azure.microsoft.com/en-us/services/site-recovery/)
</td>     </tr>      <tr >       
<td width="450" valign="top" >IT Automation
</td>        
<td width="425" valign="top" >[Azure Automation](https://azure.microsoft.com/en-us/services/automation/)
</td>     </tr>      <tr >       
<td width="450" valign="top" >Log Analytics
</td>        
<td width="425" valign="top" >[OMS (previously known as Microsoft Operational Insights)](https://azure.microsoft.com/en-us/services/operational-insights/)
</td>     </tr>      <tr >       
<td width="450" valign="top" >Security & Compliance
</td>        
<td width="425" valign="top" >[OMS (previously known as Microsoft Operational Insights)](https://azure.microsoft.com/en-us/services/operational-insights/)
</td>     </tr>   </tbody></table>  

 

As you see, both well and lesser known Azure services which are already in existence for quite some time power the platform. What I do like about bundling this in one suite is that the services are placed in a broader, general concept of management and are priced and presented in a much more coherent way! Just like System Center is a suite of separate software platforms, so are their Azure counterparts now!

 

In the next posts to come I will attempt to provide a thorough overview on each of these services and provide some scenario’s where they might be positioned perfectly in your environment.

 

Thanks for giving this post a read and I hope to catch you later! And should you have questions or remarks, don’t hesitate to provide me feedback!

 

Jan out.
