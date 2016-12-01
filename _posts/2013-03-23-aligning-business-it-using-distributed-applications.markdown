---
author: janvanmeirvenne
comments: true
date: 2013-03-23 12:59:38+00:00
layout: post
link: http://scug.be/jan/2013/03/23/aligning-business-it-using-distributed-applications/
slug: aligning-business-it-using-distributed-applications
title: Aligning Business & IT using distributed applications
wordpress_id: 90
categories:
- '#scom'
- '#sysctr'
---

There is no bigger struggle within the IT market than making IT transparent and understandable for the business it supports. We often fail to realize that IT has a supporting function within an organization, and that it only exists to make the core business processes easier to execute. This principle doesn’t always reflect on the way management software is used.

 

Specifically for monitoring, there are often misunderstandings or disconnections between the data the business owner is interested in and the data the IT administrator wants to see.

 

Examples:

 

 

**Business Owner**

 

[![image](http://scug.be/jan/files/2013/03/image_thumb.png)](http://scug.be/jan/files/2013/03/image.png)   
_Which departments and business processes are impacted when this application goes down?        
How much downtime has my HR department experienced due to IT problems?         
How do I now which business processes own this application?_

 

**IT Administrator**

 

[![image](http://scug.be/jan/files/2013/03/image_thumb1.png)](http://scug.be/jan/files/2013/03/image1.png)   
_What components fall under this application?        
What happens if this database goes down?         
Did I reach my SLA regarding my database-farm?_

 

__

 

I believe that is is rather easy to satisfy both these guys their needs by only using the distributed application concept of Operations Manager:

 

 

**Creating the business organigram**

 

[![image](http://scug.be/jan/files/2013/03/image_thumb2.png)](http://scug.be/jan/files/2013/03/image2.png)

 

A distributed application is normally a skeleton for an application consisting of a set of components, where underlying components rollup their health to the overall state of the application. This has many similarities to a business: a set of departments and processes which report to an executive board.. So by just porting the company’s organigram to a DA you can very quickly set the first step towards a business-focused approach towards application monitoring.

 

**Creating the application models**

 

[![image](http://scug.be/jan/files/2013/03/image_thumb3.png)](http://scug.be/jan/files/2013/03/image3.png)

 

This will be a bigger hassle. If you really want a proper Business to IT overview, all applications used in the business have to be modeled in SCOM. This can be a small or big project depending on the company size, but Rome wasn’t built in a day either.

 

**Bringing the worlds together**

 

Eventually, you can start making the application DA’s components of your business DA’s, effectively linking them together. This will enable the following scenario:

 

[![image](http://scug.be/jan/files/2013/03/image_thumb4.png)](http://scug.be/jan/files/2013/03/image4.png)

 

By correctly scoping this system and using the correct medium (Sharepoint, Dashboard, Webconsole,…) to bring it to the respective consumer, you satisfy both the business’ as the IT department’s needs. Reporting and health-management is now possible on 2 levels without any chance on inconsistency. As long as the correct processes are implemented to keep this DA-hierarchy up-to-date this is a long-term solution for aligning IT and Business regarding business continuity. Note that not all applications have to be linked to business objects if they have only meaning to the IT department (eg an overview of all servers).
