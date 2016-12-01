---
author: janvanmeirvenne
comments: true
date: 2014-04-01 13:45:45+00:00
layout: post
link: http://scug.be/jan/2014/04/01/fixing-missing-stored-proecdures-for-dashboards-in-the-datawarehouse/
slug: fixing-missing-stored-proecdures-for-dashboards-in-the-datawarehouse
title: Fixing missing stored procedures for dashboards in the datawarehouse
wordpress_id: 251
categories:
- '#scom'
- '#sysctr'
---

When you are creating a dashboard in SCOM, you might receive strange errors stating that a certain stored procedure was not found in the database. This is an issue I often encounter and which indicates that something went wrong during the import of the visualization management packs. These management pack bundles contain the SQL scripts that create these procedures. By extracting the MPB's and manually executing each SQL script, you can fix this issue rather easily. However, this is not a supported fix and you should make a backup of your DW in case something explodes!

Extract the following MPB files using [this](http://nocentdocent.wordpress.com/2012/02/28/opsmgr-2012-how-to-dump-management-pack-bundles/) script:

- Microsoft.SystemCenter.Visualization.Internal.mpb
(found on the installation media under 'ManagementPacks')

- Microsoft.SystemCenter.Visualization.Library.mpb
(use the one from the patch folder if you applied one: %programfiles%\System Center 2012\Operations Manager\Server\Management Packs for Update Rollups)

First, execute the single SQL script that came from the Internal MPB (scripts with 'drop' in the name can be ignored) on the Datawarehouse

Secondly, execute each SQL script from the Library MPB on the DW (again, ignore the drop scripts). The order does not matter.

The dashboards should now be fully operational. Enjoy SCOM at its finest!
