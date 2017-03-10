---
layout:     post
title:      "Setting up Service Manager for multi-tenancy"
subtitle:   "Part 2: extending work- and configuration items for isolation"
date:       2017-03-09 12:00:00
author:     "Jan Van Meirvenne"
comments: true
---
This is a n-part blog series about how to setup and extend Service Manager to allow for a secure, multi-tenant ITSM process execution.

* [Part 1: Out-of-the-box options *(overview of the basic RBAC features)*](http://www.jvm-net.com/2017/03/01/scsm-multitenant-1/)
* **Part 2: Isolating work- and configuration items *(Creating custom controls / lists to extend sementation)***
* Part 3: Restricting forms and controls *(Altering default controls to enforce RBAC)*
* Part 4: Scoped work- and configuration item creation *("Create From Template" task: how to apply everywhere)*
* Part 5: Control work-item movements *(Showcase of community tool which allows scoped transfer of tickets)*
* Part 6: Setting up an optimized, shared view structure *(how to increase your users' efficiency)*
* Part 7: Business Intelligence *(how to present data in the multi-tenant ITSM platform)*

# Linking work- and configuration items to their tenants

In the previous blogpost, I outlined the basic scenario and concepts towards providing a multi-tenant Service Manager platform for your company's departments.

In order to distinct between a ticket or CI belonging to the Finance or HR department, we need to tag each item with a value unique for each part of your company.

One of the fields that is a suitabe candidate for such use is the "support group" property. This is an out-of-the-box function for incidents and requests, and is used
to specify which IT team is currently task to treat it.

<img src="{{ site.url }}/assets/scsm-multitenant/incidentsupportgroupform.png" />

It is easy to extend this use to support multiple departments in a company instead of multiple teams within a single department by adding an additional layer in the support group hierarchy:

* HR
    * Tier 1
    * Tier 2
    * Tier 3
* Finance
    * Tier 1
    * Tier 2
* Facilities
    * General

> Note: as not each department might need multiple tiers for their service management execution, the sub-tiers may be variable in count. It is recommended to setup at least one sub-tier for each department (call it 'Default' or 'General' for example) to be consistent with the hierarchy structure.

This allows to not only isolate and identify tickets between departments, but also enable each department to use their own escalation chain! By using per-derpartment security roles and restricted forms (described in a later post), we can easily provide users with only the items they need to maintain.

However, the support group field is not present on all items. CI's, problems and changes do not have this field. This is not an issue when using the platform in a single department as problems and changes are usually not part of an hierarchic assignment process. In order to be able to isolate these types of items in the same way we can do with requests and incidents, we'll need to add this field where needed.

This can be done using some XML authoring and C# SDK usage.

# Scenario: adding a 'support group' field to the problem work-item type

If we want to have department users to utilize a problem management process, they need to be able to work with problems in SCSM without interference of items created by other teams.

## Extending the problem class

Before we can show anything in the console / portal, we need to add a new list 'Problem Support Group' and link it to the problem class.

{% highlight XML %}
<TypeDefinitions>
    <EntityTypes>
      <ClassTypes>
		 Add the support group field to the problem class by using an extension -->
        <ClassType ID="Custom.SM.PR.SupportGroupExtension" Abstract="false" Accessibility="Public" Hosted="false" Singleton="false" Base="SWPL!System.WorkItem.Problem" Extension="true">
          <Property ID="SupportGroup" EnumType="Custom.SM.PR.SupportGroupEnum" Type="enum" />
        </ClassType>
        
      </ClassTypes>
      <EnumerationTypes>
		<!-- The actual list is defined here -->
        <EnumerationValue ID="Custom.SM.PR.SupportGroupEnum" Accessibility="Public" />
      </EnumerationTypes>
    </EntityTypes>
    
  </TypeDefinitions>
  <Categories>
	<!-- These categories are needed to make the list show up in our console (authoring / lists) -->
    <Category ID="Custom.SM.PR.SupportGroupEnumCategory1" Target="Custom.SM.PR.SupportGroupEnum" Value="MESUA!Microsoft.EnterpriseManagement.ServiceManager.UI.Authoring.EnumerationViewTasks" />
    <Category ID="Custom.SM.PR.SupportGroupEnumCategory2" Target="Custom.SM.PR.SupportGroupEnum" Value="System!VisibleToUser" />
    <Category ID="Custom.SM.PR.SupportGroupManagementPackCategory" Value="MESUC!Microsoft.SystemCenter.ManagementPack">
      <ManagementPackName>Custom.SM.PR.SupportGroup</ManagementPackName>
      <ManagementPackVersion>1.0.0.0</ManagementPackVersion>
      <ManagementPackPublicKeyToken>6bd7dd9c936b3b55</ManagementPackPublicKeyToken>
    </Category>
  </Categories>
  <LanguagePacks>
    <LanguagePack ID="ENU" IsDefault="true">
      <DisplayStrings>
	  <!-- User-Friendly naming of our new property -->
        <DisplayString ElementID="Custom.SM.PR.SupportGroupEnum">
          <Name>Problem Support Group</Name>
        </DisplayString>
      </DisplayStrings>
    </LanguagePack>
  </LanguagePacks>
  {%endhighlight%}

The management pack snippet above will define a new list "Problem Support Group" and link it to a new field "SupportGroup" which is added to the problem type.

When we now open the problem form, we can see the new field in the 'extensions'-tab:

<img src="{{ site.url }}/assets/scsm-multitenant/problemsupportgroupextensiontab.png" />

We can also see and populate the list in the lists overview in the authoring pane:

<img src="{{ site.url }}/assets/scsm-multitenant/problemsupportgrouplist.png" />

"Ok, great, but I want to integrate this field with my problem form, instead of having it in a separate extension-tab"

This is where the seconds step comes in:

## Adding fields to existing forms

Adding fields to existing forms can be done in 2 ways:

### Using the management pack Form-tag

The SCSM management pack structure allows you to define overrides for existing forms like this:

{% highlight XML %}
<ManagementPackFragment SchemaVersion="SM2.0" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Categories>
    
    <Category ID="Custom.SM.PR.FormCustomizations.ManagementPackCategory" Value="MESUC!Microsoft.SystemCenter.ManagementPack">
      <ManagementPackName>Custom.SM.PR.FormCustomizations</ManagementPackName>
      <ManagementPackVersion>1.0.0.0</ManagementPackVersion>
      <ManagementPackPublicKeyToken>6bd7dd9c936b3b55</ManagementPackPublicKeyToken>
    </Category>
  </Categories>
  <Presentation>
    <Forms>
		<!-- Specify a new form which will be 'grafted' on an existing form (baseform). We need to specify the projection-type (to let the form know which data should be retrieved) and the internal name of the form (smlets can help with finding this value: Get-SCSMForm|select typename) -->
      <Form ID="Custom.SM.PR.FormCustomizations.Form" Accessibility="Public" BaseForm="SPL!ServiceManager.ProblemManagement.Library.Form.Problem" Target="SPL!System.WorkItem.Problem.ProjectionType" TypeName="Microsoft.EnterpriseManagement.ServiceManager.Applications.ProblemManagement.Forms.ProblemForm">
        <Category>Form</Category>
        <Customization>
		<!-- We can either use the XML customization tag to add controls and modify properties -->
       <!--   <AddControl Parent="StackPanel106" Assembly="PresentationFramework, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" Type="System.Windows.Controls.Label" Left="0" Top="0" Right="0" Bottom="0" Row="0" Column="0" /> -->
         <AddControl Parent="StackPanel106" Assembly="Microsoft.EnterpriseManagement.UI.SMControls, Version=7.0.5000.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" Type="Microsoft.EnterpriseManagement.UI.WpfControls.ListPicker" Left="0" Top="0" Right="0" Bottom="0" Row="0" Column="0" />
          <!-- To hook-up the list-picker to our custom list and field, we need to sets it parentcategorid to the id of our list item (use smlets: Get-SCSMEnumeration), and bind it to our custom class property -->
          <PropertyBindingChange Object="ListPicker_1" Property="SelectedItem">
            <NewBinding Enabled="True" Path="SupportGroup" Mode="TwoWay" BindsDirectlyToSource="False" UpdateSourceTrigger="PropertyChanged" />
          </PropertyBindingChange>
          <PropertyChange Object="ListPicker_1" Property="ParentCategoryId">
            <NewValue>e24ffbc7-78fd-a2da-e4b1-32312a8035ce</NewValue>
          </PropertyChange>
          <PropertyChange Object="ListPicker_1" Property="Width">
            <NewValue>Auto</NewValue>
          </PropertyChange>
          <PropertyChange Object="ListPicker_1" Property="Height">
            <NewValue>Auto</NewValue>
          </PropertyChange>
          <PropertyBindingChange Object="Label_1" Property="Content">
            <NewBinding Enabled="False" />
          </PropertyBindingChange>
          <PropertyChange Object="Label_1" Property="Content">
            <NewValue>Support Group:</NewValue>
          </PropertyChange>
          <PropertyChange Object="Label_1" Property="HorizontalAligment">
            <NewValue>Left</NewValue>
          </PropertyChange>
          <PropertyChange Object="Label_1" Property="MinWidth">
            <NewValue>100</NewValue>
          </PropertyChange>
		  <!-- A more flexible way (if you know your way around C#) is to configure the extra form controls in a custom usercontrol and add it -->
          <AddControl Parent="StackPanel106" Assembly="Custom.SM.PR.UserControlOverride, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" Type="Custom.SM.PR.UserControlOverride.Override" Left="0" Top="0" Right="0" Bottom="0" Row="0" Column="0" />
        </Customization>
      </Form>
    </Forms>
  </Presentation>
  <Resources>
  <!-- We need to specify our custom user control assembly here so it can be accessed by the console -->
    <Assembly ID="Custom.SM.PR.FormCustomizations.Assembly" Accessibility="Public" FileName="Custom.SM.PR.UserControlOverride.dll" QualifiedName="Custom.SM.PR.UserControlOverride, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  </Resources>
</ManagementPackFragment>

{%endhighlight%}

There are 2 ways to use this XML feature to setup the form customization:

* Specify the customizations directly in XML
    * This is the most easy way and also the method used by the [SCSM authoring tool](http://cireson.com/blog/adding-a-field-to-the-right-of-an-existing-field-scsm-2012-authoring-tool/) (which you should use if your not familiar with the XML syntax)
    * Not all modifications work, caused by (what I assume) are layout operations (executed by form-code) hapening after the customization is applied

* Add a custom user-control (through a DLL) and have it manipulate the form to add extra controls
    * This allows you the most control over what will happen as you can use all SDK features instead of the limited XML customizations
    * Not for the faint of heart, although this sample will help you to get the picture :)

I will describe the last option as it is the least documented one on the internet (and also my favortie option ;)

* Create a new WPF User Control Library in Visual Studio. All the info is in the old, but gold article from Anton Gritsenko [here](http://blog.scsmsolutions.com/2011/08/create-custom-user-control-for-scsm-2010/)
* Modify the user control so that it is invisible (we only need the code-behind, not the actual layout functions). Setup an event handler for the datacontextchanged-event. This will enable us to execute code once the SCSM form is loaded with data.
{% highlight XML %}
<UserControl x:Class="Custom.SM.PR.UserControlOverride.Override"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:Custom.SM.PR.UserControlOverride"
             mc:Ignorable="d" Width="0" Height="0" Visibility="Collapsed" DataContextChanged="UserControl_DataContextChanged">
</UserControl>
{%endhighlight%}
* Setup the code so that it will get the location of the form where the existing problem controls are present, and then add and bind our custom controls
{% highlight csharp %}
 public partial class Override : UserControl
    {
        private bool flag = false;
        public Override()
        {
            InitializeComponent();
        }
        private ListPicker lp = null;
        private Binding myBinding = null;

		// this is not my function.
		// It is a common function found on the Technet forums that will recursively search a SCSM form for a certain control-type
        private DependencyObject GetParentDependancyObject(DependencyObject child, string name)
        {
            try
            {
                //We need the logical tree to get our parent
                DependencyObject parent = LogicalTreeHelper.GetParent(child);
                DependencyObject lastparent = null;
                //Is the parent our specified control?
                if (name != "" && parent.GetType().ToString() == name) return parent;
                //No, process further
                while (parent != null)
                {
                    string s = parent.GetType().ToString();
                    if (s == name && name != "") return parent;
                    parent = LogicalTreeHelper.GetParent(parent);
                    if (parent != null) lastparent = parent;
                }
                //Return results
                if (name != "") return null;
                else return lastparent;
            }
            catch
            {
                return null;
            }
        }

        private void UserControl_DataContextChanged(object sender, DependencyPropertyChangedEventArgs e)
        {
            // Because a form can have its data-context (= workitem data that is linked to it) changed multiple times, we set a flag after the first execution to prevent our code from firing more than once.
            if (flag == false)
            {
				// Find the tab-control of our form (it is important when adding our custom usercontrol in the MP XML to use a parent that is in the general tabcontrol)
                DependencyObject doTabControl = GetParentDependancyObject(this,
                    "Microsoft.EnterpriseManagement.ServiceManager.Applications.ProblemManagement.Forms.GeneralTabControl");

                GeneralTabControl tabitem_general = (GeneralTabControl) doTabControl;
				// Get the existing assigned-to field, and get its parent container, which is a stackpanel. A stackpanel adds controls right after another, so is ideal for adding extra controls without worrying too much about layout.
                UserPicker userPicker = (UserPicker) tabitem_general.FindName("AssignedToUserPicker");
                System.Windows.Controls.StackPanel parent = (System.Windows.Controls.StackPanel) userPicker.Parent;
				// Define the extra controls and bind them to the data
                lp = new ListPicker(new Guid("e24ffbc7-78fd-a2da-e4b1-32312a8035ce"));
                myBinding = new Binding();
                myBinding.Source = this.DataContext as IDataItem;
                myBinding.Path = new PropertyPath("SupportGroup");
                myBinding.Mode = BindingMode.TwoWay;
                myBinding.UpdateSourceTrigger = UpdateSourceTrigger.PropertyChanged;
                lp.Name = "SupportGroupListPicker";
                lp.Width = Double.NaN; // AUTO
                lp.Height = Double.NaN; // AUTO
                lp.HorizontalAlignment = HorizontalAlignment.Stretch;
                lp.VerticalAlignment = VerticalAlignment.Center;
                Label lbl = new Label();
                lbl.HorizontalAlignment = HorizontalAlignment.Left;
                lbl.VerticalAlignment = VerticalAlignment.Center;
                lbl.Content = "Support Group:";
				// Add the controls to the stackpanel
                parent.Children.Add(lbl);
                parent.Children.Add(lp);

                flag = true;

            }
			// Update the binding when data is changed
            else
            {
                
                myBinding = new Binding();
                myBinding.Source = this.DataContext as IDataItem;
                myBinding.Path = new PropertyPath("SupportGroup");
                myBinding.Mode = BindingMode.TwoWay;
                myBinding.UpdateSourceTrigger = UpdateSourceTrigger.PropertyChanged;
                lp.SetBinding(ListPicker.SelectedItemProperty, myBinding);

            }
            
        }
    }
{%endhighlight%}
* Reference the assembly in the MP project and make sure that 'Package to bundle' is enabled in the reference properties
* Build the User Control project, and check the MP <Assembly>-tag to make sure that it recognizes the generated DLL (MP Authoring autocomplete does this)
* Build the MP and copy the generated MPB-file from the bin/debug folder into your management group
* The field should now be added to the problem form

We can now use this field in forms, views, security roles. The problem work-item type is ready for multi-tenant use!

<img src="{{ site.url }}/assets/scsm-multitenant/problemsupportgroupform.png" />


The full project can be found [here](https://github.com/JanVanMeirvenne/Custom.SM.PR.SupportGroup). I hope it helps you putting 1 and 1 together and setup your own customizations :)

Jan out






