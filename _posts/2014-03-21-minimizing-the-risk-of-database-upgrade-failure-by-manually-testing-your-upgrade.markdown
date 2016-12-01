---
author: janvanmeirvenne
comments: true
date: 2014-03-21 18:14:31+00:00
layout: post
link: http://scug.be/jan/2014/03/21/minimizing-the-risk-of-database-upgrade-failure-by-manually-testing-your-upgrade/
slug: minimizing-the-risk-of-database-upgrade-failure-by-manually-testing-your-upgrade
title: '[SCOM 2007–>2012] Minimizing the risk of failures by manually testing your
  database upgrade'
wordpress_id: 243
categories:
- '#scom'
- '#sysctr'
tags:
- '#scom'
- '#sysctr'
---

When you upgrade the last management server (the RMS) to SCOM 2012, the setup will also upgrade the database. As this step is very sensitive to inconsistencies, it is advised to test this update on a copy of your Operational database.

 

The following steps are done during the database upgrade:

 

**1. Database infrastructure updates (build_mom_db_admin.sql)**

 

-_ Sets up recovery mode, snapshots, partitions_

 

**2. Database upgrade / creation (build_mom_db.sql)**

 

_- Populates a new database, or converts a 2007 database_

 

**3. Enter localization data (build_momv3_localization.sql)**

 

_- Inserts localized message strings_

 

_- This is the last step where things can go wrong, it is also the last action you can perform manually_

 

**4. Management Pack Import**

 

_- This step is performed by the setup itself. It imports new management packs into the database._

 

**5. Post Management Pack import operations (Build_mom_db_postMPimport.sql)**

 

_- This step performs some database changes that are needed after the MP import_

 

**6. Configuration Service Database setup (Build_om_db_ConfigSvcRole.sql)**

 

_- This steps sets up the tables and roles needed for the configuration service. The SCOM configuration service used to be a file-based database containing the current configuration of workflows for the management group. For performance and scalability reasons, this role has been transferred to the database-level._

 

All the SQL files can be found on the SCOM 2012 installation media in the server\amd64 folder.

 

First, should you have encountered an upgrade failure, restore your database to the original location, you cannot reuse the current version as it probably will be corrupted beyond repair. Restore the RMS binaries, use the identical database and service account information of the original setup. The RMS will restore itself in the management group. The subscriptions and administrator role will be reset however.

 

Take another backup copy of the Operational Database, restore it to a temporary database and execute the aforementioned SQL scripts in the correct order. You will usually get a more precise error message in the SQL management studio than in the SCOM setup log (stored procedure names, table names…).

 

A common cause for upgrade failure is an inconsistent management pack. Pay close attention to the affected items mentioned in the error message. They usually contain a table or column reference. Lookup this reference and try to get hold of a GUID. Query the GUID in the live database using the PowerShell interface (Get-MonitoringClass –Id <GUID> or Get-MonitoringObject –Id <GUID> for example). This way you can find out which element(s) are causing issues and remove the management pack(s) that contain them.

 

You can uninstall suspect management packs by executing the stored procedure “p_RemoveManagementPack” providing the management pack GUID as parameter. If the stored procedure returns a value of 1 then check if there are any depending management packs you should remove first.

 

It will be rinse and repeat from here on: restore the backup of the original database again to the temporary database, remove the problematic management pack using the stored procedures and re-execute the scripts until no more errors occur. After you have listed up all troublesome management packs, you can remove them from the live management group, wait for the config churn to settle and retry the upgrade. Usually, the subsequent upgrade steps are pretty safe and you shouldn’t have big issues beyond this point.
