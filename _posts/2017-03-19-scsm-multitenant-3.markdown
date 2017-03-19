---
layout:     post
title:      "Setting up Service Manager for multi-tenancy"
subtitle:   "Part 2: extending work- and configuration items for isolation"
date:       2017-03-19 12:00:00
author:     "Jan Van Meirvenne"
comments: true
---
This is a n-part blog series about how to setup and extend Service Manager to allow for a secure, multi-tenant ITSM process execution.

* [Part 1: Out-of-the-box options *(overview of the basic RBAC features)*](http://www.jvm-net.com/2017/03/01/scsm-multitenant-1/)
* [Part 2: Isolating work- and configuration items *(Creating custom controls / lists to extend sementation)*](http://www.jvm-net.com/2017/03/09/scsm-multitenant-2/)
* **Part 3: Restricting forms and controls *(Altering default controls to enforce RBAC)***
* Part 4: Scoped work- and configuration item creation *("Create From Template" task: how to apply everywhere)*
* Part 5: Control work-item movements *(Showcase of community tool which allows scoped transfer of tickets)*
* Part 6: Setting up an optimized, shared view structure *(how to increase your users' efficiency)*
* Part 7: Business Intelligence *(how to present data in the multi-tenant ITSM platform)*

In the previous post, I gave a detailed explanation on all possible means to add custom fields / lists to SCSM forms.

In this post, we will use the same techniques to modify existing controls instead of new ones.

In our scenario, we need to prevent that users can assign items directly to an analyst that is not in their department. All cross-department assignments should occur in a formal way, meaning that the ticket support group should be set to the target department and the assigned user needs to be cleared. This will make the ticket pop up in the unassigned filter/view of the concerning department, allowing them to allocate an analyst on it using their own processes.

While it is possible to enforce this behavior through addons (more on that in a later post), it is hard to do so in the standard work-item forms. One would need to do a lot of manipulations to get it to behave like we want to. The quickest solution is to disable these pesky controls and force users to use tasks to perform the ticket assignment manipulations. Tasks are much easier to control as we can create a brand new shiny button with only the code we need vs replacing/hacking code in the form.

<img src="{{ site.url }}/assets/scsm-multitenant/IRControlsDisabled.png" />

Disabling controls are done in the same way as we add controls to a form. But instead of creating a control, we obtain a reference to an existing one and modify its properties.

I already explained the difference between Form Customization in XML and through a custom C# user-control. The added benefit of using C# in this scenario is that we can do conditional control disabling.

For example, I can only have the controls disabled if a non-admin user is using the form, or the form is not being used for template creation. Should it be really necessary, you could do a per-user/role enable/disable operation if needed!

I have posted a [small sample project](https://github.com/JanVanMeirvenne/SCSM-IncidentFormLockDown) that shows how to get this done for the incident form. Feel free to poke/probe it for your own needs!

Jan out!


