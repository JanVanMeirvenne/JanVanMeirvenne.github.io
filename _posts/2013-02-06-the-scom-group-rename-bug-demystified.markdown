---
author: janvanmeirvenne
comments: true
date: 2013-02-06 15:20:15+00:00
layout: post
link: http://scug.be/jan/2013/02/06/the-scom-group-rename-bug-demystified/
slug: the-scom-group-rename-bug-demystified
title: The SCOM group rename bug demystified
wordpress_id: 76
categories:
- '#scom'
- '#sysctr'
---

This was a long time existing issue I had with SCOM which I finally figured out to the bottom.

 

Imagine the following scenario: you spend hours creating a management pack. You made a nice group with a very complex query which luckily performs perfectly. And then you see a typo…

 

You rename the groupname, but then notice the following:

 

- In the group overview of the SCOM console, the group name refuses to change     
- In the property window of the group, the group is named correctly

 

This can sometimes be corrected by [editing the management pack XML](http://thoughtsonopsmgr.blogspot.be/2011/05/strange-behavior-when-names-of.html).

 

But usually there is nothing wrong with the management pack, but the way the groupname is stored in the database:

 

There are 2 locations in the Operations Manager database where the displayname-information is stored:

 

- The displayname-field in the BaseManagedEntity table     
- The LTValue-field in the LocalizedText table

 

The localizedtext-table is meant to contain multiple language strings depending on the regional settings of the machine that is running the Operations Manager console. The value of the LTValue-field is the localized name for all objects in SCOM. It is here where the correct value is stored. However, this field is used only for the property window of the group, while the list-name (in the overview pane) is linked to the displayname-field of the basemanagedentity-table. Due to a bug in the DBCreateWizard-tool (used to generate the SCOM databases), these 2 values are not kept in sync during a rename. This is because of a missing languagecode in the _MOMManagementGroupInfo_-table, causing the display-information not to be updated correctly.

 

The fix for this issue is to add the missing language-code, and if you really don’t want to recreate the group with the correct name, manually sync the 2 display-fields in the database. This last action must not be done again after you added the missing language-code, because the rename-issue will not be present anymore for NEW groups that will be created in SCOM.

 

To fix the missing language code, run the following SQL-query on a backed up Operations Manager database:

 <table cellpadding="2" cellspacing="0" border="0" width="400" ><tbody >     <tr >       
<td width="400" valign="top" >update __MOMManagementGroupInfo__ set LanguageCode = 'ENU'
</td>     </tr>   </tbody></table>  

To fix any existing ‘rename-bugs’, run the following query on the same backed-up database:

 <table cellpadding="2" cellspacing="0" border="0" width="400" ><tbody >     <tr >       
<td width="400" valign="top" >         

update e             
set e.DisplayName = lt.LTValue              
from BaseManagedEntity e inner join ManagedType et on e.BaseManagedTypeId = et.ManagedTypeId inner join LocalizedText lt on e.FullName = lt.ElementName where et.BaseManagedTypeId = '4CE499F1-0298-83FE-7740-7A0FBC8E2449' and lt.LanguageCode = 'ENU' and lt.DisplayStringId is not NULL and lt.LTValue != e.DisplayName;

      
</td>     </tr>   </tbody></table>  

The GUID is the GUID of the InstanceGroup class, which is the base-class for all console-created groups in SCOM.

 

The best way to prevent this issue is to run the first query as soon as your SCOM environment is deployed. I am happy to finally have the full story on this issue that has caused a lot of frustration.
