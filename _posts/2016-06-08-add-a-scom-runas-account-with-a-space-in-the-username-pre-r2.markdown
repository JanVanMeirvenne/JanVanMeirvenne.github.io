---
author: janvanmeirvenne
comments: true
date: 2016-06-08 20:42:46+00:00
layout: post
link: http://scug.be/jan/2016/06/08/add-a-scom-runas-account-with-a-space-in-the-username-pre-r2/
slug: add-a-scom-runas-account-with-a-space-in-the-username-pre-r2
title: Add a SCOM runas-account with a space in the username (pre-R2)
wordpress_id: 485
---

 

Although it rarely occurs it is sometimes necessary to work with a runas-credential that contains a space in the username (eg a web-service authentication). In pre 2012 R2 environments, spaces are apparently not allowed in the username when entering it through the SCOM console GUI.

 

Lukcily, the PowerShell interface does not contains such restrictions, and if upgrading to R2 is not an option, you can use the following set of commands to create and distribute a runas-account that contains spaces:

 <table cellpadding="2" width="400" border="1" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

# setup variables             
$user = 'user with spaces'              
$pass = 'mypassword'              
$scomserver = 'scom01' 

         

            
# create a PS credential from the user/password variables   
$securepass = $pass|ConvertTo-SecureString -AsPlainText -force              
$cred = new-object System.Management.Automation.PSCredential ($user,$securepass)

         

# Connect to the SCOM environment             
Import-Module operationsmanager              
New-SCOMManagementGroupConnection $scomserver

         

# Retrieve the agents, management servers or pools to distribute the account to (can be an array of a combination of these object types, use Get-SCOMAgent, Get-SCOMManagementServer or Get-SCOMResourcePool to populate)             
$pool = Get-SCOMResourcePool

         

# create a simple (or basic) runas-account using the PS credential             
$runas = Add-SCOMRunAsAccount -Simple -Name $user -RunAsCredential $cred

         

# distribute the account             
Set-SCOMRunAsDistribution -RunAsAccount $runas -MoreSecure -SecureDistribution $pool

      
</td>     </tr>   </tbody></table>  

After running these commands, you can verify (but not modify!) the changes in the GUI:

 

[![image](http://scug.be/jan/files/2016/06/image_thumb5.png)](http://scug.be/jan/files/2016/06/image5.png)[![image](http://scug.be/jan/files/2016/06/image_thumb6.png)](http://scug.be/jan/files/2016/06/image6.png)

 

Assigning this account to a profile afterwards is perfectly doable with the GUI however!

 

Although this post revolves around bypassing a GUI restriction, you can use the same commands of course to administer any runas-account in the context of automation or general nerdyness ![Smile](http://scug.be/jan/files/2016/06/wlEmoticon-smile.png)

 

Let me know if you have further question on this topic! Jan out!
