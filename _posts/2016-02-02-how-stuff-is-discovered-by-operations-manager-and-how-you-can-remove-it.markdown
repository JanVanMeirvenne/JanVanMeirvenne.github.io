---
author: janvanmeirvenne
comments: true
date: 2016-02-02 21:43:38+00:00
layout: post
link: http://scug.be/jan/2016/02/02/how-stuff-is-discovered-by-operations-manager-and-how-you-can-remove-it/
slug: how-stuff-is-discovered-by-operations-manager-and-how-you-can-remove-it
title: How stuff is discovered by Operations Manager, and how you can remove it
wordpress_id: 404
---

SCOM is a very extensive platform, able to handle and maintain a vast inventory of monitored objects. This is done using a [model-based database](http://blogs.technet.com/b/servicemanager/archive/2009/01/27/the-system-center-platform-in-service-manager-part-2-the-model-based-database.aspx) where various relationships and attached workflows (monitors and rules) will cause the entire monitoring function to operate correctly. The way in which objects are created or deleted in this inventory can be two-fold:

 

[Discovery-based](http://social.technet.microsoft.com/wiki/contents/articles/14260.operations-manager-management-pack-authoring-discovery.aspx)

 

This is the most common way to control SCOM’s inventory. A discovery is a workflow defined in a management pack which function is to detect a certain condition on a machine (using the SCOM agent) and if true, report the collected data back to the SCOM management server, which will create an object based on a certain class. For day-to-day authoring this is the goto mechanism as it is very straightforward and does not require extensive programming knowledge to accomplish. Discoveries can use Registry, WMI, Powershell and other types of data sources to determine whether a condition is present or not.

 

Pro’s:

 

- Easy to understand

 

- Open in nature, modifications are possible through overrides

 

Cons:

 

- Complex discoveries (dense application topologies) require extensive Powershell scripting, which might eat away resources

 

[![ ](http://social.technet.microsoft.com/wiki/resized-image.ashx/__size/550x0/__key/communityserver-wikis-components-files/00-00-00-00-05/6266.DiscoveryProcess.gif)](http://social.technet.microsoft.com/wiki/cfs-file.ashx/__key/communityserver-wikis-components-files/00-00-00-00-05/6266.DiscoveryProcess.gif)

 

[Connector-based](https://msdn.microsoft.com/en-us/library/bb437519.aspx)

 

This is a more complex approach to control inventory. There are inbound and outbound connectors within Operations Manager. An outbound connector can send alerts to other management platforms (eg send Operations Manager alerts to Service Manager for incident creation). An inbound connector is not used to import exclusively alerts from other systems, but also to allow other systems to push an entire health model altogether. The way it works is that you import the management pack that comes alongside with the connector, and then setup the connector itself (this is usually done from the system that is connecting to Operations Manager). The program logic in the other system and connector will then use the structure which was defined in the management pack to create objects, set monitor states, input perdormance data,… It is a very powerful mechanism as it allows you to connect another platform to SCOM and have that platform do all the probing and collection while SCOM is just used to store and visualize that data. It prevents ‘wheel-reinventing’ and allows the use of the best tool for the job. The prime example for this method is the Operations Manager / Virtual Machine Manager integration: in that scenario you use SCVMM to first import some required management packs (the structure) and then setup multiple connectors which are used to push SCVMM’s inventory and corresponding health state into SCOM.

 

Pro’s:

 

- Instead of having SCOM collect the data itself in addition of the connected platform, have that platform push its own inventory and metrics instead. This saves resource cycles.

 

Cons:

 

- The connector developer is responsible for the amount of data and the interval at which it is being sent. There is no override mechanism to control the behavior.

 

- The connector’s inner workings is not open in nature. Figuring out why a certain health state is pushed can be difficult to find out.

 

[![image](http://scug.be/jan/files/2016/02/image_thumb.png)](http://scug.be/jan/files/2016/02/image.png)

 

 

**How can I find out if an object was added by a discovery or a connector?**

 

There are some tables in the OperationsManager database that can be used to exactly know who added which object when.

 

**_BaseManagedEntity_** – this table contains the base properties (id, type, path, …) of all objects that exist within the management group       
**_ManagedType_** – this table contains all object classes       
**_TypedManagedEntity_** – this table links the objects in the BaseManagedEntity table to the classes in the ManagedType table       
**_DiscoverySource_** – this table holds the information for each TypedManagedEntity which Connectors and/or Discoveries have contributed to the existence and status of the TME       
**_DiscoverySourceToTypedManagedEntity – _**This table links the various sources in the DiscoverySource table (multiple discoveries or connectors can be the source for a single object) to the entities in the TypedManagedEntity table       


 

By joining these tables together in a query, one can get a pretty good overview of the links between the discoveries / connectors and the objects:

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

**_select                  
bme.BaseManagedEntityId as 'ObjectID',                   
bme.FullName as 'ObjectFullName',                   
source.BaseManagedEntityId as 'HostObjectId',                   
source.FullName as 'HostObjectFullName',                   
case ds.DiscoverySourceType when 1 then 'Connector' when 0 then 'Discovery' when 3 then 'Singleton' end as 'ObjectCreationType',                   
case ds.DiscoverySourceType when 1 then c.ConnectorId when 0 then d.DiscoveryId end as 'ObjectCreationId',                   
case ds.DiscoverySourceType when 1 then bmec.DisplayName when 0 then d.DiscoveryName end as 'ObjectCreationName',                   
d.DiscoveryTarget,                   
ds.TimeGeneratedOfLastSnapshot as 'LastModified'                   
from                   
BaseManagedEntity bme                   
inner join                   
TypedManagedEntity tme on tme.BaseManagedEntityId = bme.BaseManagedEntityId                   
inner join                   
DiscoverySourceToTypedManagedEntity dstme on dstme.TypedManagedEntityId = tme.TypedManagedEntityId                   
inner join                   
DiscoverySource ds on ds.DiscoverySourceId = dstme.DiscoverySourceId                   
left join                   
BaseManagedEntity source on source.BaseManagedEntityId = ds.BoundManagedEntityId                   
inner join                   
ManagedType mt on tme.ManagedTypeId = mt.ManagedTypeId                   
left join                   
Discovery d on ds.DiscoveryRuleId = d.DiscoveryId                   
left join                   
DiscoveryClass dc on d.DiscoveryId = dc.DiscoveryId                   
left join                   
ManagedType target on target.ManagedTypeId = dc.ManagedTypeId                   
left join                   
Connector c on ds.ConnectorId = c.ConnectorId                   
left join                   
BaseManagedEntity bmec on c.BaseManagedEntityId = bmec.BaseManagedEntityId_**

         

**_order by bme.FullName_**

         

**__**

      
</td>     </tr>   </tbody></table>  

This shows the following information for each discovery source:

 <table cellpadding="2" width="844" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="169" valign="top" >**ObjectId**
</td>        
<td width="703" valign="top" >The id of the monitoringobject that was discovered
</td>     </tr>      <tr >       
<td width="169" valign="top" >**ObjectFullName**
</td>        
<td width="703" valign="top" >the full name (type + name) of the monitoringobject
</td>     </tr>      <tr >       
<td width="169" valign="top" >**HostObjectId**
</td>        
<td width="703" valign="top" >The id of the object that is hosting the monitoringobject
</td>     </tr>      <tr >       
<td width="169" valign="top" >**HostObjectFullName**
</td>        
<td width="703" valign="top" >the full name (type + name) of the hosting monitoringobject
</td>     </tr>      <tr >       
<td width="169" valign="top" >**ObjectCreationType**
</td>        
<td width="703" valign="top" >The type of discovery source (Connector = Connector-based,Discovery = Discovery-based,Singleton = static object like groups or distributed applications, which have no discovery)
</td>     </tr>      <tr >       
<td width="169" valign="top" >**ObjectCreationId**
</td>        
<td width="703" valign="top" >The id of the connector or discovery
</td>     </tr>      <tr >       
<td width="169" valign="top" >**ObjectCreationName**
</td>        
<td width="703" valign="top" >The name of the connector or discovery
</td>     </tr>      <tr >       
<td width="169" valign="top" >**DiscoveryTarget**
</td>        
<td width="703" valign="top" >In case of a discovery-based entry, shows the class that the discovery targets
</td>     </tr>      <tr >       
<td width="169" valign="top" >**LastModified**
</td>        
<td width="703" valign="top" >The last time a change was propagated from a discovery-source. This timestamp only changes when the object changes (creation or attribute update)!
</td>     </tr>   </tbody></table>  

 

**How do I prevent a certain object from being discovered?**

 

Well, with regular discoveries it is easy: just disable the discovery for the system you do not want to be probed for the existence of a certain type and you’re done.

 

Depending on the scope you want to exclude, you need to make sure you are targetting the correct level.

 

[![image](http://scug.be/jan/files/2016/02/image_thumb1.png)](http://scug.be/jan/files/2016/02/image1.png)

 

[![image](http://scug.be/jan/files/2016/02/image_thumb2.png)](http://scug.be/jan/files/2016/02/image2.png)

 

As an example: lets say you have a monitoring type ‘FinanceApp’ which represents a software installation of a finance platform. The type is associated with a discovey ‘FinanceApp Discovery’ which is targetted at the ‘Windows Computer’ class.

 

If you do not want to add any FinanceApp instances to your management group at all for the moment (you’re preparing overrides on the monitoring part of the management pack for example), you can create an override that sets ‘enabled’ to ‘false’ for the discovery on ‘all objects of class: Windows Computer’. This will disable the discovery globally and will prevent any instances from being detected and created in SCOM.

 

If you have a development farm where a lot of developers torture the Finance App constantly (developers are cruel beings, we all know it), and you do not want to have constant alerts and error-states from polluting your monitoring landscape, you can create a group where you add all Windows Computer objects that are deemed as development system and then create an override that sets ‘enabled’ to ‘false’ on ‘for a group…’. It is very important that you are using the Windows Computer type as membership type, as this is the target type of the discovery. Choosing a lower level type (eg logical disk) will not work. However, choosing a higher level type does (disabling a discovery with target ‘logical disk’ on a group consisting of Windows Computer instances will result in all logical disks contained in those Windows Computer objects being affectected).

 

If you have a single development server then you can just use the ‘For a specific instance of type “Windows Computer” and then choose an instance from the list.

 

[![image](http://scug.be/jan/files/2016/02/image_thumb3.png)](http://scug.be/jan/files/2016/02/image3.png)

 

For a connector, things become harder. Unless the connected system supports modification of the amount of data that is being synced, you will just need to go with the flow on that part.

 

**Well, too late, I got some stuff in my management group I do not want. How do I remove it?**

 

Again, the answer depends on how the data was inserted in the first place.

 

For items created by a discovery, you can just disable the discovery using the information provided in the previous section. You can then open up the Operations Manager PowerShell interface and run the command **_Remove-SCOMDisabledClassInstance_**. This will prompt you with a warning that this is a database-heavy operation (and it actually is, so take care). If you choose to continue, SCOM will effectively scrub all objects and relationships that have disabled discoveries.

 

[![image](http://scug.be/jan/files/2016/02/image_thumb4.png)](http://scug.be/jan/files/2016/02/image4.png)

 

For items created by a connector, you should use the connecting system to correctly shutdown the connectors (eg the SCVMM PowerShell command ‘**_Remove- SCOpsMgrConnection’ _**to remove the SCVMM integrations). Should this not be possible for some reason you can just delete the connector by again using PowerShell:

 

**Get-SCOMConnector –DisplayName ‘<the name of the connector you want to remove>’|Remove-SCOMConnector**

 

Simulated example with a SCVMM connector:      


 

[![image](http://scug.be/jan/files/2016/02/image_thumb5.png)](http://scug.be/jan/files/2016/02/image5.png)

 

Note: NEVER REMOVE THE ‘MOM Internal Connector’! This connector is used by SCOM itself to create objects in the database. Deleting this connector would mean certain death for your management group.

 

**Ok, I tried this, but still no cigar!**

 

There are some gotcha’s that exist when dealing with the SCOM inventory:

 

**_1) An object is only removed if ALL discovery sources are either disabled or report that the condition that evaluates if an object should be created equates to ‘false’._**

 

Lets say the FinanceApp has 2 discoveries: one that checks for a simple detection of the FinanceApp Windows Service for basic object creation, and another one that uses a PowerShell script to looks up the version information to populate the object’s version property. Only if both these discoveries are disabled before running the PowerShell cleanup or both have run and both report that there is ‘nothing to discover’ will the object be removed.

 

This is only the case for discoveries on the same level. If there are discoveries that target the FinanceApp object itself (eg to discover and monitor sub-components), they will recursively follow the deletion of the upper-level ones.

 

[![image](http://scug.be/jan/files/2016/02/image_thumb6.png)](http://scug.be/jan/files/2016/02/image6.png)

 

**_2) Sometimes SCOM doesn’t clean up after itself_**

 

I have seen this happen from time to time in 2 situations:

 

- The SCOM infrastructure is under pressure or fails at a bad time, causing an internal cleanup of objects to not come through completely

 

- Somewhat exotic, but when you use the SCOM SDK to update objects that were created initially through a discovery and you don’t create a dedicated connector, the ‘MOM Internal Connector’ is used. Since you can not remove this connector you’re basically screwed as the object now has a discovery source that is linked to a connector that will never be deleted, giving the object eternal life.

 

Some blogs mention using the [‘IsDeleted = 1’](http://sethisageek.blogspot.be/2011/08/stalegray-objects-in-operations-manager.html) trick to remove objects directly from the database. This will trigger a database workflow that recursively cleans up the object itself and its related descendants. This procedure is not supported by Microsoft however.

 

During a support call a Microsoft engineer mentioned that using the SDK is actually supported to do this. So I took some code from [a Service Manager SDK example](http://scsmnz.net/c-code-snippets-for-service-manager-1/) which demonstrates to delete workitems in order to make it work on SCOM objects (after all, technically they are the same things).

 

I put this in a [custom PowerShell module](https://github.com/JanVanMeirvenne/SCOMRunbooks/blob/master/SCOMRunbooks/Modules/CustomSCOM.psm1) which I will extend with more reusable scripts in the future.

 

This module autoloads the regular OperationsManager module, and will also load the SDK DLL files. For now, you need to have the SCOM console installed on the machine where you are using this.

 

To remove an object, you can use the module like this:

 

**_Import-Module <path to module>\CustomSCOM.psm1          
New-SCOMManagementGroupConnection –ComputerName <name of a SCOM management group server>           
Get-SCOMClassInstance –Id <Id of the object to remove>|Remove-cSCOMObject_**

 

This will leverage the SDK to create an incremental discovery data array, where the object will be added to as ‘needing removal’. This array will then be committed to the management group, which will mark the object as deleted in the database. Same result as the IsDeleted trick, but way easier to achieve and if really needed automate, cooler and more supported!

 

Please note that this is a very powerful function (think ‘the one ring’ powerful) that can destroy a management group if used incorrectly. Make sure to read its

 

prompts well to prevent any unintentional deletions. And of course, use at your own risk ![Smile](http://scug.be/jan/files/2016/02/wlEmoticon-smile.png).

 

Well, that’s it for this week. Until the next one! Jan out.

 

ps Feel free to reach out to me on @JanVanMeirvenne or [jan@jvm-net.com](mailto:jan@jvm-net.com) in case of feedback on this topic
