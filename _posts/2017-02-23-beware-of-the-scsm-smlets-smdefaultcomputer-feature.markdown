---
layout:     post
title:      "Beware of the Service Manager SMLETS' SMDefaultComputer variable"
subtitle:   "More than meets the eye"
date:       2017-02-23 12:00:00
author:     "Jan Van Meirvenne"
comments: true
---

Without the [smlets-module](https://smlets.codeplex.com/), automating Service Manager would be a hard bullet to bite.
I use it all the time for both automation and management scenario's.

* Modifying CI / Work Item properties that are hard to reach by Console (closing test requests for example)
* Import data into a CI or WorkItem structure from an external source (CSV/AD/...)
* Copying User Role Changes

One of the more powerful things it offers (and is not possible in a supported way otherwise) is removing any item from SCSM without leaving a trace.
This is done using the following command:
{% highlight PowerShell %}
Get-SCSMObject -Id $MyObjectId|Remove-SCSMObject -Force
{% endhighlight %}

You can target a specific SCSM management group in multiple ways:

* You can set a variable $SMDefaultComputer with the value of a SCSM management server, which will be used as default connection point for any command
* Each command supports the '-ComputerName' switch, which sets the management server to be used for that command

## However...

While using the smlets to copy workitems between 2 management group (deleting test work-items in the target management group before moving production data), I encountered the following behavior:

* I had the $SMDefaultComputer variable set to the source management group
* I issued a remove-scsmobject command with a -ComputerName parameter pointing the target management group

One would expect that a direct ComputerName parameter would override the default-variable.

This is NOT the case. 

When the $SMDefaultComputer variable is set, it overrides -ComputerName!!!
So in essence I deleted production data instead of my test data :(
Luckily, a restore of the DB and some fooling around with the Exchange Connector's mailbox queue allowed me to restore most items.

To be sure that I had not foobared my command, I decided to check out the smlets source code and found this section:
{% highlight csharp %}

PSVariable DefaultComputer = SessionState.PSVariable.Get("SMDefaultComputer");
                        if (DefaultComputer != null)
                        {
                            _mg = ConnectionHelper.GetMG(DefaultComputer.Value.ToString(), _credential, this._threeLetterWindowsLanguageName);
                        }
                        else
                        {
                            _mg = ConnectionHelper.GetMG(ComputerName, _credential, this._threeLetterWindowsLanguageName);
                        }
{% endhighlight %}

To summarize: SMDefaultComputer > ComputerName!

I do not intend to bash smlets, as it is one of the greatest tools in my belt. I recognize that carelessness on my part also contributed to the issue I encountered.
However, this behavior is counter-intuitive and makes a command not execute in the way you think it should.

Just a headsup to use a clean PowerShell session, or at least clear the SMDefaultComputer variable before doing any cross-environment work!

ps: The smlets are now also published in the [Powershell gallery](https://www.powershellgallery.com/packages/SMLets). This allows you to single-line install the module as follows:
{% highlight PowerShell %}
Install-Module smlets -scope currentuser
{% endhighlight %}

Jan out