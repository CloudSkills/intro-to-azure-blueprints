# Introduction to Azure Blueprints

Let me ask you a question. Would you consider building a house without knowing what it would look like when you're done? The supplies needed? An idea of what is allowed by building and safety codes? I assume the answer to that question is an emphatic, NO. So why do you build your Azure environments that way?

As a consultant for a managed service provider, producing a consistent, supportable and reproducible environment is a part of my design philosophy. Deviating my company's design principles and best practices increase support costs and lead to unhappy customers. That's why I'm very interested in technologies that can save me time and effort.

There are many techniques to control the deployment of an environment. Techniques range from the low tech spreadsheet with a checklist of tasks, to the modern infrastructure-as-code technologies like Terraform or Ansible. In my view, checklists are too slow and error-prone and infrastructure-as-code technologies need too much ramp-up time to be effective for the small Azure environments our customers need. Microsoft might have hit the sweet spot between simplicity and power with Azure Blueprints.

Azure Blueprints is a new service, currently in preview, that helps you define your environment's foundation before you start building your Azure "house". Azure Blueprints use existing Azure services like Azure Policies, permissions and ARM templates to give you control of the rollout of your environment. Let's take a look a the components of an Azure Blueprint.

## Blueprint Components

Azure Blueprints are made up of building blocks called **artifacts**. Each Artifact is a current Azure services so you are likely already familiar with them. 

## Artifacts

+ **ARM Templates** - Use your own ARM templates or any of the Azure Quick Start templates 
+ **Resource Groups** - Specify the resource groups you want to create
+ **Roles Assignments** - Apply IAM roles to the deployed resource groups to apply permissions to users and/or groups
+ **Policies Assignments** - Apply Azure Policies and/or Initiatives to your environments

## Blueprint creation workflow
What does creating a new Blueprint look like? The creation of a Blueprint involves three simple steps:

1. **Create a Draft of your Blueprint** -  Add Artifacts (ARM templates, Policies, Resource Groups and Roles) into a hierarchy to define your environment. When you add an ARM Template to your design you will have the option to define the parameters when you build the Blueprint or allow the parameters to be defined when the Blueprint is assigned.
2. **Publish your Blueprint** - After your Blueprint is ready to go, publish it by giving it a version number and description. It is now ready to be assigned.
3. **Assign your Blueprint** - Assign your published Blueprint to your subscriptions or management groups. After assignment your environment will be created, resources deployed and polices applied.

At the time of assignment you can choose to use Lock Assignments to protect your design. The Lock Assignment allow you protect the artifacts you've defined as read-only or to prevent them from being deleted, even by subscription owners. 

### Lock Assignment settings:

* **Don't Lock** - The assignment is not locked. Users, groups, and service principals with the appropriate permissions can modify and delete deployed resources.
* **Do Not Lock** - The assignment is locked. Deployed resources can't be deleted - even by subscription owners. Not all resource types support locking. Due to caching, locks may take up to 30 minutes to become enforced.
* **Read Only** - The assignment is locked. Deployed resources can't be modified or deleted - even by subscription owners. Not all resource types support locking. Due to caching, locks may take up to 30 minutes to become enforced.

## Blueprint Versioning
Now that we have created a Blueprint, what do we do when we want to make changes? The process is very similar to the initial Blueprint creation.

1. **Create a new Draft** - Select the Blueprint you want to create a new version and edit it to make the necessary changes
2. **Publish the Blueprint** - Publish the Blueprint with a new description and version number
3. **Assign the Blueprint** - Assign the new published Blueprint to a subscription or management groups

## Updating Assigned Blueprints

If you need to make changes to a Blueprint after it has been assigned, you can update the settings of the version but not add or remove Artifacts. To accomplish this you would need to create a new version of the Blueprint.

Changes we can make when updating an assignment are:

1. Adding or removing lock assignments or changing the lock assignment type
2. Changing the values of dynamic parameters
3. Changing the assigned Blueprint to a newer version

## Scenario: 

Now that we understand the basics of Azure Blueprints, let's consider the following scenario. You have been assigned a project to assist a team of developers to deploy small Azure environments for an application they're writing. I have listed the design requirements below. Let's walk through how we could do this with Azure Blueprints.

### Requirements:
* VNet configured with NSG
* Resource group for network resources
* Resource group for VM
* Contributor role for "Cloud Admins" group on the subscription
* Virtual machine contributor role for "Helpdesk Team" on the Virtual Machine Resource Group
* Ensure VM are backed up
* Assign Usage tag to all resources
* Make the environment reproducible for other subscriptions
* Disallow environment user from deleting the defined resources

### Walk through of creating a Blueprint
1. Open the Azure Portal and launch the Azure Blueprints service

![OpenBlueprint](/images/step-1-open-blueprint.png "Open Blueprint")

2. In the Create a blueprint section, click **Create**

![CreateNewBlueprint](/images/step-2-create-new.png "Create New Blueprint")

3. Select the **Basic Networking (VNET) template**

![SelectBlueprintTemplate](/images/step-3-select-template.png "Select Blueprint Template")

4. Enter Blueprint name, Blueprint description and Definition location, then click Next: **Artifacts**

![BlueprintBasicInfo](/images/step-4-create-blueprint-basics.png "Blueprint Basic Info")

> The location can be a subscription or a management group

5. At the subscription level

   * Add artifact > Policy assignment

![AddArtifacts](/images/step-5-1-Add-Artifacts.png "Add Artifacts")

   * Add artifact > Policy assignment

![AddPolicy](/images/step-5-2-Add-Policy.png "Add Policy")

   * Type "Azure Backup" in the search field and select **Azure backup should be enabled for Virtual Machines** > Click Add

![PolicyAzureBackup](/images/step-5-3-Add-Policy-Backup.png "Policy - Azure Backup")    

   * Type "add a tag" in the search field and select **Add a tag to resource groups** > Click Add

![PolicyAzureTags](/images/step-5-4-Add-Policy-Tags.png "Policy - Azure Tags")

   * From the Create Blueprint hierarchy > select the **Add a tag to resource groups** policy and enter the tag: **Usage** and tag value: **Production**

![PolicyAzureTags](/images/step-5-4-1-Add-Policy-Tags.png "Policy - Azure Tags")

   * Add artifact > Resource group named **Virtual Machine Resource Group**

![AddResourceGroup](/images/step-5-5-Add-ResourceGroup.png "Add Resource Group")

   * Add artifact > Role assignment > Select **Contributor** and choose the **Cloud Admins** group (this is a preexisting Azure AD group)

![AddRoleAssignment](/images/step-5-6-Add-RoleAssignment.png "Add Role Assignment")

6. At the VNET Resource Group level

   * Add artifact > Azure Policy > select the **Inherit a tag from the resource group** policy

![AddPolicyInheritTag](/images/step-6-Add-PolicyInheritTag.png "Add Policy Inherit Tag")

7. At the Virtual Machine Resource Group level

   * Add artifact > Azure Policy > select the **Inherit a tag from the resource group** policy

![AddPolicyInheritTag](/images/step-7-Add-PolicyInheritTag.png "Add Policy Inherit Tag")

   * Add artifact > Role assignment > Select **Virtual Machine Contributor** choose the **Helpdesk Team** group (this is a preexisting Azure AD group)

![AddRoleAssignmentVM](/images/step-7-1-Add-RoleAssignmentVM.png "Add Role Assignment VM")

8. Save Draft

![SaveDraft](/images/step-8-Save-Draft.png "Save Draft")

9. Publish Blueprint

    * Click on Blueprint definitions > and from there you will see a list of Blueprints 

![BlueprintView1](/images/step-9-BlueprintView1.png "Blueprint View 1")

> Notice the different icons, the lighter icon denotes draft Blueprints and the darker ones are published

   * Open Blueprint definitions > Click on Blueprint-01 > Publish Blueprint

![PublishBlueprint](/images/step-9-1-PublishBlueprint.png "Publish Blueprint")

   * Enter version number and description

!["PublishBlueprintInfo"](/images/step-9-2-PublishBlueprintInfo.png "Publish Blueprint Info")

10. Assign Blueprint

   * Click on Blueprint-01 > Assign blueprint

![AssignBlueprint](/images/step-10-AssignBlueprint.png "Assign Blueprint")

   * Enter Assignment name, Location, definition version
   * Choose Lock Assignment value **Do Not Delete** to prevent artifacts defined by blueprint from being deleted

![AssignBlueprintSettings0](/images/step-10-1-AssignBlueprintInfo0.png "Assign Blueprint Settings 0")

   * Assign values to the available template parameters

> Blueprint parameters can be defined during its creation or they can be left to be entered when the Blueprint is assigned.  

![AssignBlueprintSettings1](/images/step-10-1-AssignBlueprintInfo1.png "Assign Blueprint Settings 1")
 
![AssignBlueprintSettings2](/images/step-10-1-AssignBlueprintInfo2.png "Assign Blueprint Settings 2")
 
![AssignBlueprintSettings3](/images/step-10-1-AssignBlueprintInfo3.png "Assign Blueprint Settings 3")

   * Click **Assign**

11. Verify resources deployed correctly

![VerifyDeployment](/images/step-11-VerifyDeployment.png "Verify Deployment")

12. Connect to Azure tenant

   * Connect to to your Azure tenant via PowerShell

13. Export Blueprint for redistribution

   * We can use the Az.Blueprint PowerShell module to export the Blueprint with the following commands:

```azurepowershell-interactive
Connect-AzAccount
$bpDefinition = Get-AzBlueprint -Name 'Blueprint-01' -Version '1.0'
Export-AzBlueprintWithArtifact -Blueprint $bpDefinition -OutputPath 'C:\Images\Blueprints'
```
   * The result of running those commands looks like this:

![ExportBlueprint](/images/step-13-ExportBlueprint.png "Export Blueprint")

   * If you browse to the specified directory you will find a folder that matches the name of your Blueprint that contains a JSON file and a subfolder called Artifacts that contains your artifacts.

![ExportedFiles](/images/step-13-1-exportedfiles.png "Exported Files")

14. Now we have exported the Blueprint named Blueprint-01 for use with other projects. We can save it locally or share it with Git or Azure DevOps.

## How we met our requirements

Let's take a quick look at which settings and artifacts we used to meet our requirements in this scenario:

* **VNet configured with NSG** - Blueprint template that included ARM templates
* **Resource group for network resources** - Resource group artifact
* **Resource group for VM** - Artifact - Resource group artifact
* **Contributor role for "Cloud Admins"** group on the subscription - Policy assignment artifact
* **Virtual machine contributor role for "Helpdesk Team"** - Role assignment artifact
* **Ensure VM is backed up** - Policy assignment artifact
* **Assign Usage tag to all resources** - Policy assignment artifact
* **Make the environment reproducible for other subscriptions** - Az.Blueprint PowerShell module 
* **Disallow environment user from deleting the defined resources** - Lock assignment 

## Summary

I am excited about how easy the Azure team has made creating new deployments using Azure Blueprints. As someone who is not familiar with tools like Ansible or Terraform, Azure Blueprints are a good building block into the infrastructure as code mindset. I'm looking forward to testing it further and using it in my customer deployments.

## Further reading

Check out this sites for more information:
* [Azure Blueprints Documentation](https://www.screentogif.com/) Official Microsoft documentation
* [Azure Tips and Tricks: Working with Azure Blueprints](https://microsoft.github.io/AzureTipsAndTricks/blog/tip210.html) 
* [An overview of Azure Blueprints | Azure Friday](https://www.youtube.com/watch?v=cQ9D-d6KkMY&t=311s) Video
* [Azure Blueprints & Policy to get DevOps right | Best of Microsoft Ignite 2018](https://www.youtube.com/watch?v=OiOXlgFNgDo) Video 