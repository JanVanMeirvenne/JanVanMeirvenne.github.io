---
author: janvanmeirvenne
comments: true
date: 2016-05-03 06:52:28+00:00
layout: post
link: http://scug.be/jan/2016/05/03/building-your-own-incident-update-form-for-service-manager/
slug: building-your-own-incident-update-form-for-service-manager
title: Building your own incident update form for Service Manager
wordpress_id: 414
---

 

I love developing…to the extent that I sometimes wonder why I chose the IT Pro life in the first place. But I love versatility as well: not being limited to a single set of skills. This empowers me to not only implement and maintain various processes and platforms, but also understand their core workings and if needed…modify them. The beautiful thing about the System Center and in extension the Azure stack is that it is extensible, moldable. Functionality can be added with relative ease as long as you have an understanding how code works and also tons of patience and perseverance.

 

The nice thing about having this skillset is that certain requests I receive might have been answered with a ‘not possible’ or ‘there is a paid addon’, but got the ‘yes, I can build it if you agree that it might take some time’ line instead.

 

One of these type of requests (they occur far too few sadly) involved an issue were the customer wanted to force the input of a resolution category and comment if an incident’s status was updated to the resolved-status. Although this functionality exists in some standard forms, it is not enforced consistently, posing a risk in process integrity.

 

So I drew the ‘I’ll built it’ card and went on my merry way. My idea was to create a new incident task which would contain a Windows Form that allows the analyst to choose a new incident status. There would also be an option to enter a comment. In the case that the analyst would select ‘resolved’ as status, an additional input would appear to allow the selection of the resolution category. Also, the comment entered would not be logged as analyst comment, but as resolution detail.

 

Some restrictions would need to be in effect to ensure the process was respected: if the status is set to ‘resolved’, choosing a resolution category and entering a comment must be a mandatory action, and if the analyst fails to do so, the update action needs to be blocked.

 

Although I have knowledge of the basic principles of development, I needed to get some insights and inspiration for SCSM-specific authoring. I came across the following interesting posts:

 

  
  * This excellent blog series by Travis Wright: [https://blogs.technet.microsoft.com/servicemanager/2010/02/11/tasks-part-1-tasks-overview/](https://blogs.technet.microsoft.com/servicemanager/2010/02/11/tasks-part-1-tasks-overview/)
   
  * This very similar addon by Oskar Landman: [https://gallery.technet.microsoft.com/SCSM-Incident-Change-04594d9b](https://gallery.technet.microsoft.com/SCSM-Incident-Change-04594d9b)
   
  * A great tutorial by Stefan Johner: [https://blog.jhnr.ch/2013/12/09/how-to-create-a-custom-scsm-console-task-by-using-some-c-and-xml-magic/](https://blog.jhnr.ch/2013/12/09/how-to-create-a-custom-scsm-console-task-by-using-some-c-and-xml-magic/)
 

### How does it work?

 

The SCSM SDK provides the option to define a [task handler](https://msdn.microsoft.com/en-us/library/hh964950.aspx). This handler provides an interface in both the management group the console is connected to, and the selected objects on which the task is activated. You can use these interfaces to query and modify virtually any aspect in your management group. In my setup, I build a task handler that gets the incident information from the selected item and uses it to display a custom form, where the analyst can update the status. The handler and the form are stored in a Class Library better known as a DLL.

 

Alongside this DLL I build a management pack that does 2 things: expose the task-handler as a button in the main SCSM console, and hide the default tasks that it replaces. This is contained in a SCSM management pack project (which comes with the [Visual Studio Authoring Extensions](https://www.microsoft.com/en-us/download/details.aspx?id=30169)). Important here is to create a reference to the DLL-project where the task-handler and form reside, and specify that they should be ‘packaged to bundle’. This will create a MPB when the MP project is build, containing both the MP XML as well as the DLL-file. When this MPB is imported, SCSM will know that the DLL in the MPB must be activated when the custom task in the MP is triggered (see Github comments for more info).

 

[![image](http://scug.be/jan/files/2016/05/image_thumb2.png)](http://scug.be/jan/files/2016/05/image2.png)

 

### 

 

## 

 

### How can I create my own awesome tasks?

 

Before I start evangelizing the power of Visual Studio, first I want to point out that there is a much easier way to get simple tasks going: by using the [standard custom task](https://technet.microsoft.com/en-us/library/hh499040(v=sc.12).aspx) feature of SCSM. You can use PowerShell or any other script type or executable to perform virtually any action. The benefit of using Visual Studio is that native, integrated code is easier to manage (no external tools that need to be present on a share somewhere, versioning,…). Also, while a [GUI can be build with PowerShell](http://www.drdobbs.com/windows/building-gui-applications-in-powershell/240049898), it is much easier in Visual Studio thanks to the visual tools. Finally, the feeling of accomplishment us much stronger when getting a native task to work ![Smile](http://scug.be/jan/files/2016/04/wlEmoticon-smile.png).

 

The tools you need to get going are:

 

Visual Studio 2012, 2013 or 2015 ([Preview 15](https://www.visualstudio.com/en-us/news/vs15-preview-vs.aspx) is not supported yet in the VSAE). If you want to build something for production use, then a Professional or Enterprise edition is required. If you just want to mess around a bit, then you can use the free [Community Edition](https://www.visualstudio.com/products/visual-studio-community-vs) (only available starting from VS 2013).

 

After VS is installed, you need to install the [System Center Visual Studio Authoring Extensions](https://www.microsoft.com/en-us/download/details.aspx?id=30169) (now that’s a mouthful). This enables you to create management packs for SCOM and SCSM right from VS.

 

I also recommend to install the [Service Manager Authoring Tool](https://www.microsoft.com/en-us/download/details.aspx?id=40896), as it brings a lot of DLL’s and MP’s that are vital / useful to reference in most SCSM projects.

 

Now, open up Visual Studio and create a new C# class library. Make sure that the target .NET Framework is 3.5. This is the version where SCSM has been compiled with. Name the solution and project accordingly (use a naming convention to standardize your code). Also, make sure to specify GIT as source control, this makes it easy to protect and version your code (more on this later).

 

[![image](http://scug.be/jan/files/2016/05/image_thumb.png)](http://scug.be/jan/files/2016/05/image.png)

 

[![image](http://scug.be/jan/files/2016/05/image_thumb3.png)](http://scug.be/jan/files/2016/05/image3.png)

 

In order to be able to use SCSM-specific functions, you’ll need to add references (similar to management pack references) to some SCSM DLL’s that contain the code we need to use.

 

Right-click on references, and then choose add.

 

[![image](http://scug.be/jan/files/2016/05/image_thumb4.png)](http://scug.be/jan/files/2016/05/image1.png)

 

Add the following DLL’s (having the SCSM authoring and regular console installed helps in easily acquiring the necessary files):

 <table cellpadding="2" width="591" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="589" valign="top" >**Name**
</td>     </tr>      <tr >       
<td width="589" valign="top" >Microsoft.EnterpriseManagement.UI.SdkDataAccess.dll
</td>     </tr>      <tr >       
<td width="589" valign="top" >Microsoft.EnterpriseManagement.UI.Foundation.dll
</td>     </tr>      <tr >       
<td width="589" valign="top" >Microsoft.EnterpriseManagement.UI.FormsInfra.dll
</td>     </tr>      <tr >       
<td width="589" valign="top" >Microsoft.EnterpriseManagement.UI.Core.dll
</td>     </tr>      <tr >       
<td width="589" valign="top" >Microsoft.EnterpriseManagement.ServiceManager.Application.Common.dll
</td>     </tr>      <tr >       
<td width="589" valign="top" >Microsoft.EnterpriseManagement.Core.dll
</td>     </tr>   </tbody></table>  

You should be able to locate them in “C:\Program Files (x86)\Microsoft System Center 2012\Service Manager Authoring\PackagesToLoad” or “C:\Windows\assembly\GAC_MSIL” folders.

 

Once all references are added you can start building your task!

 

The project starts by default with a single class creatively called ‘Class1’ and a namespace called ‘ClassLibrary1’. It is useful to change this asap to improve readability. You can do this in the project properties and the class1.cs file (you need to change both to ensure your custom namespace name is reused everywhere). Prompts for rename can safely be allowed in this case. Also, it is advised to rename the actual filename from Class1.cs to the name of your class (eg EditTask.cs)

 

Why a rename is important? The management pack we will see later on refers to both the namespace and classname of the custom task in order to execute its code. Having something clear like ‘MyTasks.EditTask’ will help in identifying which class performs which task. 

 

[![image](http://scug.be/jan/files/2016/05/image_thumb5.png)](http://scug.be/jan/files/2016/05/image4.png)

 

Before:

 

[![image](http://scug.be/jan/files/2016/05/image_thumb6.png)](http://scug.be/jan/files/2016/05/image5.png)

 

After:

 

[![image](http://scug.be/jan/files/2016/05/image_thumb7.png)](http://scug.be/jan/files/2016/05/image6.png)

 

In order to add a GUI to our task, we need to add a Windows Form class to our project. Windows Presentation Foundation, a more modern graphical platform would be a better choice, but for novice developers it is easier to use the older Windows Forms. You can do this by right-clicking on your project and adding a Windows Form:

 

[![image](http://scug.be/jan/files/2016/05/image_thumb8.png)](http://scug.be/jan/files/2016/05/image7.png)

 

You can now use the graphical tools to create a GUI that fulfills your needs:

 

[![image](http://scug.be/jan/files/2016/05/image_thumb9.png)](http://scug.be/jan/files/2016/05/image8.png)

 

There is a code-behind section where you can add code to the various form events (load, button click, close). You can also add parameters so that you can pass objects from and to the form. I use this to pass the SCSM connection object so that the form can retrieve data and the task code can retrieve the user-input afterwards. To access this code-behind, the simplest way is to double click on the form or one of the controls to instantly create a handler function for the clicked item (eg double-click on a button to immediately be able to add code for when it is clicked at runtime).

 

[![image](http://scug.be/jan/files/2016/05/image_thumb10.png)](http://scug.be/jan/files/2016/05/image9.png)

 

Notice the ‘using’-lines in my code. These are used to include pieces of the references we added to our projects, making it easier to use the SCSM specific code.

 

Now to make use of this form in the SCSM console, and bind it to data we select, we need to define a task handler. Remember the library class we renamed first after creating the project? We will use it to define a task handler.

 

First, we need to mark our class as a task handler. This is done by specifying the ConsoleCommand [interface](https://msdn.microsoft.com/en-us/library/87d83y5b.aspx). An interface is kind of a template you use to make a class adhere to the requirements of a platform / function. In this case we use it to make our class usable by SCSM by ensuring the correct functions are present. We can specify our interface by adding its name after our class-name, separated by a ‘:’. This will cause a prompt to implement the functions specified in the interface. You can use this to immediately add all necessary functions, in this case a function ‘ExecuteCommand’ will be added. ‘ExecuteCommand’ is the function SCSM will call when you click the linked task in the console, and pass data from the selected item to it.

 

[![image](http://scug.be/jan/files/2016/05/image_thumb11.png)](http://scug.be/jan/files/2016/05/image10.png)

 

From then on it is easy to build the task logic. The only things left worth mentioning are:

 

You can get a connection object which you can use to perform any SCSM SDK action using this code:

 

IManagementGroupSession emg = FrameworkServices.GetService<IManagementGroupSession>();

 

This will use the existing connection of the analyst console, including its security constraints.

 

You can get the selected item’s GUID (and use it to retrieve the object) this way:

 

EnterpriseManagementObject IncidentObject = emg.ManagementGroup.EntityObjects.GetObject<EnterpriseManagementObject>(new Guid(nodes[0]["$Id$"].ToString()), ObjectQueryOptions.Default);

 

You can summon a GUI and pass data from / to it like this:

 

AssignmentForm form = new AssignmentForm();      
form.emg = emg.ManagementGroup;       
// we need to convert the tier-class representing the incident status to a guid-format used by the GUI       
ManagementPackEnumeration SelectedStatus = (ManagementPackEnumeration)IncidentObject[IncidentClass, "Status"].Value;       
form.StatusTier = SelectedStatus.Id;       
// show the GUI as a dialog (popup) and record the outcome of it (OK or cancel)       
DialogResult r = form.ShowDialog();

 

You can use the Github link below to browse all the commented code, so I will not dig deeper into the C# part right now (phew ![Smile](http://scug.be/jan/files/2016/04/wlEmoticon-smile.png)). Lets take a look at the accompanying management pack instead, where we need to define our task and link it to our custom code.

 

First, you need to add a SCSM MP project (with a logical name) to the existing solution where your task library already resides.

 

[![image](http://scug.be/jan/files/2016/05/image_thumb12.png)](http://scug.be/jan/files/2016/05/image11.png)[![image](http://scug.be/jan/files/2016/05/image_thumb13.png)](http://scug.be/jan/files/2016/05/image12.png)

 

Add a reference to the existing task library project. This way we can reference the assembly in our MP XML.

 

[![image](http://scug.be/jan/files/2016/05/image_thumb14.png)](http://scug.be/jan/files/2016/05/image13.png)[![image](http://scug.be/jan/files/2016/05/image_thumb15.png)](http://scug.be/jan/files/2016/05/image14.png)

 

When the reference is added, make sure you set its ‘Package To Bundle’ property to ‘true’, so that it gets added to the MP Bundle when building the MP project.

 

[![image](http://scug.be/jan/files/2016/05/image_thumb16.png)](http://scug.be/jan/files/2016/05/image15.png)

 

Now you can add a MP Fragment, and start writing XML code!

 

[![image](http://scug.be/jan/files/2016/05/image_thumb17.png)](http://scug.be/jan/files/2016/05/image16.png)

 

The management pack is rather simple in comparison to the library part. It contains the following parts:

 

We need to define the MP as being a native SCSM one. This is important because otherwise it will be interpreted as a SCOM MP, resulting in tasks not being displayed among other things.

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

<Categories>             
<!-- We must specify the management pack as a SCSM type, else the tasks won't be shown (they would be perceived as SCOM tasks and hidden) -->              
<Category ID="Custom.SM.Incident.ChangeStatus.MP.CATEGORY_MP" Value="MESUC!Microsoft.EnterpriseManagement.ServiceManager.ManagementPack">              
<ManagementPackName>Custom.SM.Incident.ChangeStatus.MP</ManagementPackName>              
<ManagementPackVersion>1.0.0.40</ManagementPackVersion>              
<ManagementPackPublicKeyToken>1e1eb76c7a2df9e9</ManagementPackPublicKeyToken>              
</Category>

….
</td>     </tr>   </tbody></table>  

In order to replace the default incident status tasks, we need to hide them. This can be done by tagging them with a specific category, instructing the console not to show the items.

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

…

         

<Category ID="Custom.SM.Incident.ChangeStatus.MP.CATEGORY_HIDEConsole" Target="SIL!System.WorkItem.Incident.StatusGroup.Task" Value="MESUC!Microsoft.EnterpriseManagement.ServiceManager.UI.Console.DonotShowConsoleTask" />             
<Category ID="Custom.SM.Incident.ChangeStatus.MP.CATEGORY_HIDEForm" Target="SIL!System.WorkItem.Incident.StatusGroup.Task" Value="MESUC!Microsoft.EnterpriseManagement.ServiceManager.UI.Console.DonotShowFormTask" />              
<Category ID="Custom.SM.Incident.ChangeStatus.MP.CATEGORY_HIDEConsole2" Target="SIL!System.WorkItem.Incident.ChangeStatusCommand.Task" Value="MESUC!Microsoft.EnterpriseManagement.ServiceManager.UI.Console.DonotShowConsoleTask" />              
<Category ID="Custom.SM.Incident.ChangeStatus.MP.CATEGORY_HIDEForm2" Target="SIL!System.WorkItem.Incident.ChangeStatusCommand.Task" Value="MESUC!Microsoft.EnterpriseManagement.ServiceManager.UI.Console.DonotShowFormTask" />              
</Categories>

      
</td>     </tr>   </tbody></table>  

Next, we define our task. We target the incident-class in order to make our task show up if an incident is selected. We first refer to the built-in SDK assembly, in order to launch the framework code, and as parameter we pass our task’s assembly and class name. Notice that it is possible to add additional arguments which are passed to your code if needed.

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

<Presentation>             
<ConsoleTasks>              
<!-- This is the actual task definition for the custom form, it refers to the assembly we define later on so that it gets initialized when the users launches the task-->              
<ConsoleTask ID="Custom.SM.Incident.ChangeStatus.MP.T_Update" Target="SWIL!System.WorkItem.Incident" Accessibility="Public" RequireOutput="false">              
<Assembly>MESUC!SdkDataAccessAssembly</Assembly>              
<Handler>Microsoft.EnterpriseManagement.UI.SdkDataAccess.ConsoleTaskHandler</Handler>              
<Parameters>              
<Argument Name="Assembly">Custom.SM.Incident.ChangeStatus.UI</Argument>              
<Argument Name="Type">Custom.SM.Incident.ChangeStatus.UI.ConsoleTask</Argument>              
  
</Parameters>              
</ConsoleTask>              
  
</ConsoleTasks>

           
…
</td>     </tr>   </tbody></table>  

To make things more sexy, we assign some existing icon to our task.

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

…

         

<ImageReferences>             
<!-- We assign a standard image icon of the original task to our custom one (great artists steal :)-->              
<ImageReference ElementID="Custom.SM.Incident.ChangeStatus.MP.T_Update" ImageID="SIL!IncidentMgmt_IncidentStatusChange_16"/>              
</ImageReferences>              
</Presentation>

      
</td>     </tr>   </tbody></table>      

At the end of the management pack, we define our assembly. Thanks to the existing project reference, most of the definition should auto-complete when using IntelliSense:

 <table cellpadding="2" width="400" border="0" cellspacing="0" ><tbody >     <tr >       
<td width="400" valign="top" >         

<Resources>             
<!-- Here we define that we want to include a custom library in our management pack. It is important to reference the library project (that contains the task handler and GUI) in Visual Studio (it needs to be a project in the same solution as the MP), and to set the 'package to bundle' property to true -->              
<Assembly ID="Custom.SM.Incident.ChangeStatus.MP.ASS_Base" FileName="Custom.SM.Incident.ChangeStatus.UI.dll" QualifiedName="Custom.SM.Incident.ChangeStatus.UI, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" Accessibility="Public" />              
</Resources>

      
</td>     </tr>   </tbody></table>  

When building the MP, a MPB is created containing both the MP code as well as the assembly. No copying of DLL’s required! Make sure to copy the MPB to a temporary location as it will be locked by SCSM when importing it, preventing a subsequent build from succeeding.

 

If all went well, you should see your new task pop-up. Click it to let the magic start!

 

[![clip_image003](http://scug.be/jan/files/2016/05/clip_image003_thumb.png)](http://scug.be/jan/files/2016/05/clip_image003.png)

 

[![clip_image001](http://scug.be/jan/files/2016/05/clip_image001_thumb.png)](http://scug.be/jan/files/2016/05/clip_image001.png)[![clip_image002](http://scug.be/jan/files/2016/05/clip_image002_thumb.png)](http://scug.be/jan/files/2016/05/clip_image002.png)

 

### 

 

### Magic?! The only magic I get is a nasty exception. What now?

 

Debugging your library code is rather easy. You need to attach your Visual Studio solution to your SCSM console process and then force the exception to occur. 

 

[![image](http://scug.be/jan/files/2016/05/image_thumb18.png)](http://scug.be/jan/files/2016/05/image17.png)[![image](http://scug.be/jan/files/2016/05/image_thumb19.png)](http://scug.be/jan/files/2016/05/image18.png)

 

When it hits, you should enter into the stack trace in your code and be able to see where and why things go south. Fix it, rebuild it, and reimport it and test again! It might be useful to reopen your console with the [clearcache](https://social.technet.microsoft.com/Forums/systemcenter/en-US/890be49b-cb2e-430c-83f5-f5b7f1e34579/console-cache?forum=systemcenterservicemanager) switch to make sure you are using the latest code when running the task.

 

Patrick Sundqvist of Litware has written a [nice post](http://blogs.litware.se/?p=272) on debugging these kind of projects.

 

### 

 

### 

 

### 

 

### Keep your code safe!

 

Congratulations! You’ve made your first addon for SCSM! But how do your protect your precious code?

 

[![image](http://scug.be/jan/files/2016/05/image_thumb20.png)](http://scug.be/jan/files/2016/05/image19.png)

 

The benefit of using Visual Studio is the native integration possibilities with code repositories like [GitHub](https://visualstudio.github.com/) and [Visual Studio Online](https://www.visualstudio.com/en-us/products/what-is-visual-studio-online-vs.aspx). Both are free cloud services (if you can live with your code being publically viewable on GitHub). When finished, just commit and push your code to the cloud to keep it safe, and even better, be able to share it!

 

[![image](http://scug.be/jan/files/2016/05/image_thumb21.png)](http://scug.be/jan/files/2016/05/image20.png)

 

And….finally but not least:

 

### 

 

### [Check out the full code](https://github.com/JanVanMeirvenne/Custom.SM.Incident.ChangeStatus.Task)

 

If you have questions or other feedback on this topic, feel free to comment! Thanks for the read and until next time! Jan out.
