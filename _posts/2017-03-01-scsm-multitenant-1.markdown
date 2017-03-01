---
layout:     post
title:      "Setting up Service Manager for multi-tenancy"
subtitle:   "Part 1: Out-of-the box options"
date:       2017-03-01 12:00:00
author:     "Jan Van Meirvenne"
comments: true
---
This is a n-part blog series about how to setup and extend Service Manager to allow for a secure, multi-tenant ITSM process execution.

* **Part 1: Out-of-the-box options (overview of the basic RBAC features)**
* Part 2: Isolating work- and configuration items *(Creating custom controls / lists to extend sementation)*
* Part 3: Restricting forms and controls *(Altering default controls to enforce RBAC)*
* Part 4: Scoped work- and configuration item creation *("Create From Template" task: how to apply everywhere)*
* Part 5: Control work-item movements *(Showcase of community tool which allows scoped transfer of tickets)*
* Part 6: Setting up an optimized, shared view structure *(how to increase your users' efficiency)*
* Part 7: Business Intelligence *(how to present data in the multi-tenant ITSM platform)*

# Concept

ITSM is often conceived as the process-framework for IT management. While this is true
(I mean 'IT Service Management' doesn't leave much to discussion), many concepts can be reused in other business processes.
One of the benefits of implementing the mindset across all regions in a company is that everyone talks and understands
the same language. This makes it easier to train people in how to deal with issues, requests and ideas. Whether they need a new cellphone or need a light fixture fixed.

Service Manager allows you to set it up in a way that it allows for the aforementioned mindset implementation.

# Scenario

To establish the exact environment for which we'll need to adapt SCSM, hereby a summary:

* There are multiple departments within the company which all implement ITIL and need access to SCSM
* Departments should only be able to assess work- and configuration items that are under their responsability
* Departments should be able to transfer items to eachother, but without directly assigning specific people to them
* Departments use the end-user portal to log tickets with other departments (as an end-user)

# Role Definition

I won't go into too much detail on how to setup the basic roles. Here are some links to help you get started:

* Kenneth van Surksum has an extensive [article on SCSM RBAC](http://www.vansurksum.com/implementing-rbac-in-system-center-2012-r2-service-manager/)
* My colleague Kurt has an old, but gold [article](http://scug.be/scsm/2010/03/21/service-manager-role-based-security-scoping/) on how to setup the various RBAC-elements


First, we need to define which roles exist and what permissions they should have.

## The Department Analyst role

In order to let everyone process tickets in their department, we will need to define a role for each department.

The role must be sufficient in level to allow access to all operations and items that need to occur within the defined scopes.

I usually go with 'Advanced Operator' level as this is the most lenient role without full admin permissions.
There are other roles, but they might be too restrictive based on requirements.
Remember, this only determines the GLOBAL access permissions. We will lock-down the specific types of tickets / CI's with the RBAC functions.
For an overview of all roles, see [this link](https://technet.microsoft.com/en-us/library/hh519690(v=sc.12).aspx).

Next, we need to setup the specific scoping to make sure users only see items they are eligible to.

### Work items

Work Items can be scoped by using queues (see referenced posts above). Queues use dynamic criteria to include work-items that have a specific properties.
Incidents and Service Requests have a property 'Support Group', which specifies to which logical support group the work-item is currently assigned.
This property is populated by a list which can be setup in a multi-level structure:

* Department 1..n
    * L1
    * L2
    * L3

This is perfect for use in our RBAC-model, as this property determines for which department the work-item is meant!

Each department should have 1 queue that holds tickets with their support group set to the target department.

By adding the queues to the respective roles, users will only be able to see the tickets they need in their consoles.

> Note: not all work-item types have a "support group" property. A later blog post will explain how to fix this.

### Configuration items

Configuration Items can be scoped with the "Configuration Item Groups" feature (again, see referenced posts).
These are groups which are very similar to SCOM Dynamic Groups. Just like with queues, we can choose a CI-type and criteria to specify which items should be included in a group.
This group can then be added in the role to make sure that users can only access a subset of the CI's.

Out-of-the box, there is not a generic CI property that can be used as identifier/tag for the department ownership. You could setup multiple criteria for multiple CI-types, but this becomes unmanageble really quick.
In a next blogpost, I will show you how you can add global properties to use for more generic scoping.

> Note: You need to be careful with your criteria as some CI-types need to stay global (users and computers are also CI's for example, which might hamper usability when restricted).

### Tasks

Tasks allow users to apply actions to the items in SCSM. This ranges from updating a ticket's status, to creating a CI or sending out a mail.

It is very important to make sure you restrict tasks that can break the RBAC-layer:

* Tasks that create items
    * Creating an item should only be possible through a set of scoped templates (more on this in a next blogpost)
* Tasks that change the support-group / assigned user property of a CI / Ticket (there are extensions that allow you to perform restricted changes, described in a later post)

You can exclude tasks from a role by deselecting them from the role

> Note: never restrict the generic Edit CI / WI tasks as this will prevent the user from opening any item in a form!

### Form templates

Form Templates are a godsend in a multi-tenant setup. They allow us to define default forms (and values) for both both CI's and tickets.

The rule of thumb in our RBAC-story is to provide one template for each item-type / support group combination ([automation](https://smlets.codeplex.com/) / templating can help in doing this quickly)

You can then include the templates in each department role. This, combined with the scoped "Create From Template" tasks will ensure that users can only create items within their support group.

> Note: be careful with restricting template provided or used by third party management packs. Some templates are used for licensing or configuration and will cause issues for your users when not included in the scope.

### Views

Try to make your views as reusable and generic as possible. Avoid having to setup dedicated views per departmend, but instead rely on the RBAC to show the correct content.

I often create a new view node in the console and setup a basic set of views which I extend once extra requirements come in:

* Company ITSM
    * My Incidents
    * My Requests
    * Incidents
        * Active
        * All
        * Unassigned
    * Requests
        * Active
        * All
        * Unassigned

Every user will use these views and thanks to RBAC they will only see the tickets applicable to them!

Keep it simple and stupid is key to providing a pleasant experience in this regard.

## Department Admin role

Optionally, There can be an 'Advanced Operator' role with identical scoping, save for the CI / Tickets.

This can be useful for cases where limited admin rights are needed to check what happened to a ticket, or do an out-of-process operation.

Alternatively, but more complex, you could use the 'Read-Only Operator' role for this functionality to allow managers to see all tickets, but prevent them from modifying items out-of-their scope.

## Global (SCSM Admin Role)

There should be a single global admin role with unrestricted permissions. To be used only by the technical owner / operators of the SCSM platform.

> This role should only be used for maintenance or extension purposes!

## End-User Role

To allow users from consuming offerings through the end-user portal, we need to setup the end-user role.

However, do not use the default one! It has way too many permissions, which can not be changed, and will mess up the scoping of your other roles.
Make sure to remove any user/group from this role!

Setup a new role with the 'end-user' profile and exclude anything except for the request offerings (provided through a [Catalog Item Group](https://technet.microsoft.com/en-us/library/hh519595(v=sc.12).aspx)) you want your end-users to consume.

This can be one or more roles depending on the structure of your offerings.

# Linking roles with your identity backend

Since we are talking about departments, chances are high that specific AD groups already exist for them.

However, don't use them directly with the SCSM roles! Create a dedicated AD group per SCSM role and add the department groups as a member. This will keep your roles flexible!

# Summary

This was the basic approach to a multi-tenant SCSM setup.
A full flow of this model can be found in this [Visio](https://janjvmnet-my.sharepoint.com/personal/jvm_jvm-net_com/_layouts/15/guestaccess.aspx?docid=02d0ff542be5a4904a5681e2cf82ce88a&authkey=AaNA7Px5K12_ByXgkmRcsuk)

> Note: some technical implementations that are not possible out-of-the-box will be detailed in the later posts on this topic

Let me know your remarks in the comments!

Jan out!