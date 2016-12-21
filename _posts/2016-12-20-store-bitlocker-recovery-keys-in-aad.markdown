---
layout:     post
title:      "Store Bitlocker Keys in Azure AD"
subtitle:   "Without needing InstantGo"
date:       2016-12-20 12:00:00
author:     "Jan Van Meirvenne"
comments: true
---

The [Azure AD Device Join](https://blogs.technet.microsoft.com/enterprisemobility/2015/05/28/azure-ad-join-on-windows-10-devices/) is a beautiful feature allowing the secure integration of personal devices into the corporate network.

During the join-process, the device's volumes are automatically encrypted and the recovery information is stored in Azure AD.

[However](https://blogs.technet.microsoft.com/home_is_where_i_lay_my_head/2016/03/14/automatic-bitlocker-on-windows-10-during-azure-ad-join/), this is only the case if the device supports "InstantGo", which is rare (currently only Surface Pro 3+).

Of course it is possible to Bitlocker the volumes and store the info in AAD manually, but this last action is only exposed through a GUI.

<img src="{{ site.url }}/assets/BitlockerAADButton.png" />

I used [Fiddler](http://www.telerik.com/fiddler) to find out which communication occurs with AAD when this button is pressed, and found out that it is a rather easy proces to wrap in a PowerShell script.

I put this script in the [Powershell Gallery](https://www.powershellgallery.com/packages/Enable-AADBitlocker/0.0.0.2/Content/Enable-AADBitlocker.ps1). It enables TPM- and RecoveryPassword-based Bitlocker encrypton on the OS disk, and then uploads the information to Azure AD. Your device needs to be joined, and the script has to be run as an admin.

You can [wrap](http://126kr.com/article/90t7z4t57b6) this script in an executable so that it can be deployed through Intune or another MDM to allow for a smooth end-user experience!
