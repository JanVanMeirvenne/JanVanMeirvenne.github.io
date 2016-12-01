---
author: janvanmeirvenne
comments: true
date: 2014-03-11 17:24:22+00:00
layout: post
link: http://scug.be/jan/2014/03/11/keeping-things-tidy-while-authoring-1st-edition/
slug: keeping-things-tidy-while-authoring-1st-edition
title: Keeping things tidy while authoring (1st edition)
wordpress_id: 241
categories:
- '#scom'
- '#sysctr'
tags:
- '#scom'
- '#sysctr'
---

 

I enjoy authoring a management pack. Not only because it is a really fun process to go through, but also to puzzle with the various techniques that you can use to make your work of art lean and mean.

 

Here are some of the things I try to do every time I embark on a new authoring adventure:

 

**_1) Use a naming convention_**

 

Using a naming convention internally in your management pack greatly helps when skimming through your XML code, associating multiple items (overrides, groups,…) or creating documentation. It is also a big advantage when you are working together with other authors: using the same convention improves readability (provided everybody involved is knowledgeable with the convention) and guarantees a standard style in your code.

 

For example, at my current customer we have the following naming convention:

 

<Customer>.<Workload>.MP.<ObjectType>_<Component>_<Name>

 

The object type is actually an abbreviation for the various possible objects a management pack can contain:

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="10" valign="top" >**Abbreviation**
</td>        
<td width="405" valign="top" >**Object Type**
</td>     </tr>      <tr >       
<td width="10" valign="top" >C
</td>        
<td width="405" valign="top" >Class
</td>     </tr>      <tr >       
<td width="10" valign="top" >D
</td>        
<td width="405" valign="top" >Discovery
</td>     </tr>      <tr >       
<td width="10" valign="top" >R
</td>        
<td width="405" valign="top" >Rule
</td>     </tr>      <tr >       
<td width="10" valign="top" >M
</td>        
<td width="405" valign="top" >Monitor
</td>     </tr>      <tr >       
<td width="10" valign="top" >REC
</td>        
<td width="405" valign="top" >Recovery
</td>     </tr>      <tr >       
<td width="10" valign="top" >G
</td>        
<td width="405" valign="top" >Group
</td>     </tr>      <tr >       
<td width="10" valign="top" >SR
</td>        
<td width="405" valign="top" >Secure Reference
</td>     </tr>      <tr >       
<td width="10" valign="top" >REP
</td>        
<td width="405" valign="top" >Report
</td>     </tr>      <tr >       
<td width="10" valign="top" >…
</td>        
<td width="405" valign="top" >…
</td>     </tr>   </tbody></table>  

So lets just say I work at company ‘Foo’ and I am working on a management pack for the in-house application ‘Bar’. I have to create a single class for it as it has only one component. It requires a service monitor and a rule that alerts on a certain NT Event. My objects would be called as followed.

 

- Management Pack: **Foo.Bar.MP          
**- Class: **Foo.Bar.MP.C_Application**         
- Discovery: **Foo.Bar.MP.D_Application**         
- Monitor: **Foo.Bar.MP.M_Application_ServiceStatus**         
- Rule: **Foo.Bar.MP.R_Application_EventAlert**

 

      


 

_**2) Avoid manual-reset monitors**_

 

For me, the blight in Operations Manager are manual reset monitors. It requires the SCOM operator to be alert and notice that a monitor is not healthy not because there is a genuine issue, but because nobody pushed the button to clear the state. This monitor is only an option in one scenario: There really is no way to detect when an issue is gone for the monitored application, and the issue must affect the health state of the SCOM object (for uptime or SLA reporting). But still, this might produce inaccurate reports because you don’t know how long ago the issue was solved when you finally reset that monitor.

 

If you can’t find a good reason to use manual reset monitors, then I suggest you take an alert rule instead. If the reason to go for a manual reset monitor was to suppress consecutive alerts, then just use a repeat count on the rule (more on that in a jiffy). 

 

_**3) Enable repeat count on alert rules**_

 

One of the biggest risks when going for an alert rule is bad weather: an alert storm. Imagine you have a rule configured to generate an alert on a certain NT Event, and suddenly the monitored application goes haywire and starts spitting out error events like crazy. Enable the repeat count feature to only create one alert for this that uses a counter to indicate the number of occurrences. Is it really useful to have SCOM show 1000 alerts to indicate 1 problem? Well, I had some requests stating: we measure the seriousness of a problem using the SCOM mails…when there are a lot of mails we now we need to take a look and else we don’t.

 

This brings me to a small tip when dealing with a workload intake meeting with the app owner (maybe I’ll write a full post on this): ALWAYS RECOMMEND AGAINST E-MAIL BASED MONITORING. This is a trend that lives in IT teams not custom to centralized and pro-active monitoring (let the problem come to us). Emphasize the importance of using the SCOM interfaces directly as this keeps the monitoring platform efficient and clean. Mails should only be used to notify on serious (fix me now or I die) issues. Also, either make the alerts in the subscription either self-sustaining (auto-resolve) or train the stakeholders to close the alert themselves after receiving the mail and dealing with the problem.

 

_**4) Create a group hierarchy**_

 

I love groups. They are the work-horses behind SCOM scoping. For every management pack I make a base group that contains all servers or other objects related to the workload. I can then scope views and overrides to this group as I please, and create a container specific for the workload. To prevent creating a trash bin group and also to be able to do more specific scoping (eg different components, groups for overrides,…) I often do the following:

 

Create a base group to contain all other groups for the workload. I use a dynamic expression that dictates that all objects of the class ‘Group’ with a DisplayName containing ‘G_<Application>_’ should be added as a subgroup. This is a set once and forget configuration. As long as I adhere to the naming convention, any groups relevant to the application workload should be automatically added. However, watch out with this because you can easily cause either a circular reference (the base group is configured to contain itself, does not compute well with SCOM) or to contain groups you did not intend to add as a subgroup.

 

Example:

 

- **G_Foo** (the base group)         
- **G_Foo_Servers** (all servers used in the workload)         
- **G_Foo_Bar** (all core component objects of the Bar application, this is useful if you want to scope access to operators on the application objects but not the underlying server ones)         
- **G_Foo_DisabledDiscovery** (This must be seen like a GPO/OU structure, create groups for your overrides and simple control which objects receive the override by drag-and-dropping them as a member.

 

**_5) When you expect a high monitored volume of a certain type, use cookdown_**

 

Lets just say our Foo application can host a high amount of Bar components on a single system and that we use a script to run a specific diagnostic command and alert on the results. Do we want to run this script for every Bar-instance? It sounds like this will cause a lot of redundant polls and increased overhead on the system. Why won’t we create a single script data source and let the [cookdown](http://technet.microsoft.com/en-us/library/ff381335.aspx) principle do its job? Cookdown implies that if multiple monitors and/or rules use the same datasource, they will sync-up. This will result in the datasource only executing once on the monitored system and it will feed all attached rules and monitors with data. There are a lot of requirements to enable this functionality, but here are the basics:

 

- The script must be cookdown compatible, this means that multiple property bags must be created (preferably one for each instance) and that the consuming rules and monitors are able to extract the property bag of their target-instance (I usually include a value containing the instance-name and use an expression-filter to match this to the name-attribute of the instance in SCOM). Remember, this script must retrieve and publish all data for all attached instances in one go.

 

- the configuration-parameters of the depending rules and monitors that use the data source must be identical. Every parameter that is passed to the script, and the interval settings must be the same or SCOM will split up the execution into 2 or more runtimes.

 

- You can use the cookdown analyzer to determine whether your setup will support cookdown. In Visual Studio you can do this by right-clicking your MP project and choosing “Perform Cookdown Analysis”.

 

[![image](http://scug.be/jan/files/2014/03/image_thumb2.png)](http://scug.be/jan/files/2014/03/image2.png)

 

This will build your management pack and then let you specify which classes will have few instances. The selected items will not be included in the report as cookdown is not beneficial for low-volume classes.

 

[![image](http://scug.be/jan/files/2014/03/image_thumb3.png)](http://scug.be/jan/files/2014/03/image4.png)

 

When you click ‘Generate Report’ an HTML file will be rendered containing a color coded report specifying all modules and whether they support cookdown (green) or not (red).

 

[![image](http://scug.be/jan/files/2014/03/image_thumb4.png)](http://scug.be/jan/files/2014/03/image5.png)       


 

**_6) Don’t forget the description field and product knowledge_**

 

Something I made myself guilty of: not caring about description or product knowledge fields. These information-stores can be a great asset when documenting your implementation. I make the following distinction between descriptions and product knowledge: a description contains what the object is meant to do, and product knowledge is what the operator is meant to do (although some information will be reused from the description to provide context). This are invaluable functions that can speed-up the learning process for a new customer, author or operator, informing them on what they are working with and why things are as they are. This does not replace a good design and architecture however, as the information is decentralized and easily modified without control. Always create some documentation external to your management pack, and store it in a safe and versioned place.

 

An additional and often overlooked tip: you can also provide a description for your overrides. Use it to prevent having to figure out why you put an override on a certain workload years ago.

 

Bottom line: always provide sufficient context to why something is present in SCOM!

 

**_7) For every negative trigger, try to find a positive one_**

 

When you are designing the workload monitoring, and you notice the customer gave an error condition for a certain scenario but not a clearing one…challenge him. The best monitoring solution is one that sustains itself: keeping itself clean and up-to-date.

 

There is one specific type of scenario where rule-based alerts are unavoidable: informative alerts that indicate a past issue or event (eg unexpected reboot, bad audit), although you must consider event collection if this is merely used for reporting purposes.        


 

**_8) Don’t use object-level overrides_**

 

Although it seems useful for quick, precise modifications, I find object-level scoped overrides dangerous. Internally, an override on an object contains a GUID, representing the Id of the overridden object in the database. When this object disappears for some reason (decommissioning, replacement,…) the override will still be there, but only target thin air. Unless you documented the override, you will have no clue why the override is there, and if you have a bunch of these guys, they can contaminate your management group quickly. Like I said earlier, I like to generalize my overrides using groups: instead of disabling a monitor for one specific object, I create a group called ‘G_Foo_Bar_DisabledMonitoringForDisk’, put my override on that group and then use a dynamic query (static works also of course, but contains some of the same issues as object-level overrides) to provision my group. This allows you to centralize your override management, track back why stuff is there and to quickly extend existing overrides without having the risk of duplicating them. Like I said previously, I highly recommend using the override description fields to provide additional information.

 

 

This was a first edition of my lessons learned regarding SCOM authoring, I’ll try to add new editions the moment I learn some new tricks! Do you have tips of your own, or do you do things differently? Let me know in the comments! Thanks for reading and until next time!

 

 

 

 

 
