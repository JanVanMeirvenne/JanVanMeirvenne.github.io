---
layout: post
title: "System.UnauthorizedAccessException when using the SCSM 2016 console"
subtitle: "Elusive bug explained"
date: 2017-06-26 12:00:00
author: "Jan Van Meirvenne"
header-img: "img/scsmbg.jpg"
comments: true
---

After having done several SCSM upgrades over the past weeks, I noticed the following issue showing up from time to time:

User runs the SCSM console under a runas-credential, and attempts to create a new workitem.

The console crashes with the following exception:

> Creating an instance of the COM component with CLSID {7AB36653-1796-484B-BDFA-E74F1DB7C1DC} from the IClassFactory failed due to the following error: 80070005 Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED)).

Apparently, this is a bug with the new spellcheck-component that SCSM 2016 uses in the console. This is a standard .NET-library which has as bug in its 4.6 iteration, causing it to
crash ungracefully if it is being used under a runas-credential.

There is a [connect-record](https://connect.microsoft.com/VisualStudio/feedback/details/2237337/spelling-checker-fails-with-access-denied-on-4-6-1-when-using-run-as-different-user) for this issue, which is still open currently.

There are 2 possible fixes:

* Easiest one: don't use runas with the SCSM console :)
* Uninstall .NET 4.6 (make sure you don't have other applications needing this version) and install .NET 4.5.1 (the offline setup is present on the SCSM install media)

Happy bugsplatting!

Jan out
