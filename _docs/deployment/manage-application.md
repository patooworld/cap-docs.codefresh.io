---
title: "Managing applications"
description: ""
group: deployment
toc: true
---

Application creation and deployment is one part of the continuous deployment/delivery process. An equally important part is optimizing deployed applications when needed. Codefresh gives you the options to manage your applications using GitOps principles

* Edit applications  
  Optimize deployed applications by changing application definitions when needed.

* Synchronize applications  
  Sync applications on-demand by manually applying sync options or selecting the reosurces to sync.

* Delete applications  
  Delete unused or legacy applications to avoid clutter and remove unnecessary resources.



### Edit application definitions 
Update General or Advanced configuration settings for a deployed application through the Configuration tab. Once the application is deployed to the cluster, the Configuration tab is available on selecting the application in the Applications dashboard. 

> You cannot change application definitions (the application name and the selected runtime), and the Git Source selected for the application.

**How to**  

1. In the Codefresh UI, go to the [Applications dashboard](https://g.codefresh.io/2.0/applications-dashboard){:target="\_blank"}.
1. Do one of the following: 
  * Select the application to update, and then from the context menu on the right, select **Edit**. 
  
  * Click the application and then select the **Configuration** tab.

{% include 
   image.html 
   lightbox="true" 
   file="/images/applications/edit-app-configuration-tab.png" 
   url="/images/applications/edit-app-configuration-tab.png" 
   alt="Configuration tab with application settings" 
   caption="Configuration tab with application settings"
   max-width="70%" 
   %} 

{:start="3"}
1. Update the **General** or **Advanced** configuration settings as needed:  
  [General configuration]({{site.baseurl}}docs/deployment/create-application/#application-general-configuration-settings)  
  [Advanced configuration]({{site.baseurl}}docs/deployment/create-application/#advanced-configuration-settings)  
  When you change a setting, the Commit and Discard Changes buttons are displayed.

  {% include 
   image.html 
   lightbox="true" 
   file="/images/applications/edit-app-change-setting.png" 
   url="/images/applications/edit-app-change-setting.png" 
   alt="Edit application settings" 
   caption="Edit application settings"
   max-width="70%" 
   %} 

{:start="4"}
1. Do one of the following:
   * To _commit all changes_, click **Commit**. This final commit screen is displayed with a diff view of the changes.  
   * To _undo all changes_ and return to the previous settings, click **Discard Changes**. This action removes all the changes you have made so far and returns you to the Applications dashboard.

   >If you change settings and then restore existing values for the same, Codefresh automatically removes the Commit and Discard Changes buttons as there are no new changes to commit or discard. 

  {% include 
   image.html 
   lightbox="true" 
   file="/images/applications/edit-app-diff-view.png" 
   url="/images/applications/edit-app-diff-view.png" 
   alt="Commit changes with diff view" 
   caption="Commit changes with diff view"
   max-width="70%" 
   %} 

{:start="5"}
1. To confirm all changes, at the bottom-left, click **Commit**.
  The changes are committed to Git, and in a few moments also synced to the cluster. 

{::nomarkdown}
<br><br>
{:/}

### Manually synchronize an application
Manually synchronize an application to expedite Git-to-cluster sync.  The sync options selected for manual sync override the sync options defined for the application. 
T
The sync options, grouped into Revision and Additional Settings, are identical to the Sync options in the General settings when you created the application. 

**Before you begin**

**How to**  
1. In the Codefresh UI, go to the [Applications dashboard](https://g.codefresh.io/2.0/applications-dashboard){:target="\_blank"}.
1. Select the application to sync, and do one of the following: 
  * From the context menu on the right, select **Synchronize**. 
  * On the top-right, click **Synchronize**. 
1. Select the **Revision** and **Additional Options** for the manual sync.  
  Review 
1. Click **Next**.
1. To syncIn the Synchronize Resources form, select the scope of the manual sync:
  * To sync only specific resources, search for the resources by any part of their names, or define a Regex to filter by the required resources.  
  * **All**: Sync all resources regardless of their sync state.
  * **Out of sync**: Sync _only_ resources that are `Out of sync`.  
    If you have already selected **Apply out of sync only** in Additional Options, this 
  * **None**: ???.

#### Revision settings for application sync
Revision settings determine the behavior for the branch you select.  

**Revision** 
The branch in Git to synchronize with the cluster.

**Prune**
When selected, removes legacy resources from the cluster that do not exist currently in the Git branch. 

**Apply only**
When selected, syncs only those resources in the application that have been changed and are `OutOfSync`, instead of syncing every resource regardless of their state. This option is useful to reduce load and save time when you have thousands of resources in an application. See [Selective Sync](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/#selective-sync){:target="\_blank"}.

**Dry run**
When selected, allows you to preview the application before changes are made to the cluster. 

**Force**  
When selected, orphans (deletes??) the dependents of a deleted resource during the sync operation. This option is useful to prevent 

#### Additional Options for application sync

##### Sync Options

* **Skip schema validation**  
  When selected, bypasses validating the YAML schema.  
* **Auto-create namespace**   
  When selected, automatically creates the namespace if the specified namespace does not exist in the cluster. If the namespace already exists, this setting is ignored.
* **Prune last**  
  When selected, removes those resources that do not exist in the currently deployed version during the final wave of the sync operation. See [Prune last](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/#prune-last){:target="\_blank").  
* **Apply out of sync only**
  When selected, syncs only those resources in the application that have been changed and are `OutOfSync`, instead of syncing every resource regardless of their state. This option is useful to reduce load and save time when you have thousands of resources in an application. See <a href="" >[Selective Sync](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/#selective-sync){:target="\_blank"}.  
* **Respect ignore differences**  
  When selected, Argo CD omits resources defined for the `spec.ignoreDifferences` attribute from the sync. Otherwise, Argo CD implements the desired state ad-hoc during the sync operation. See [Respect ignore difference configs](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/#respect-ignore-difference-configs){:target="\_blank"}.

##### Prune propagation policy
{::nomarkdown}Defines how resources are pruned, applying Kubernetes cascading deletion prune policies. 
Read more at <a href="https://kubernetes.io/docs/concepts/architecture/garbage-collection/#cascading-deletion" target="_blank">Kubernetes - Cascading deletion</a>.<ul><li><b>Foreground</b>: The default prune propagation policy used by Argo CD. With this policy, Kubernetes changes the state of the owner resource to `deletion in progress`, until the controller deletes the dependent resources and finally the owner resource itself. </li><li><b>Background</b>: When selected, Kubernetes deletes the owner resource immediately, and then deletes the dependent resources in the background.</li><li><b>Orphan</b>: When selected, Kubernetes deletes the dependent resources that remain orphaned after the owner resource is deleted.</li></ul> </br>{:/}
All Prune propagation policies can be used with:  

**Replace**: When selected, Argo CD executes `kubectl replace` or `kubectl create`, instead of the default `kubectl apply` to enforce the changes in Git. This action will potentially recreate resources and should be used with care. See [Replace Resource Instead Of Applying Change](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/#replace-resource-instead-of-applying-changes){:target="\_blank"}.   
  

**Retry**: When selected, retries a failed sync operation, based on the retry settings configured:   
* Maximum number of sync retries (**Limit**)  
* Duration of each retry attempt in seconds, minutes, or hours (**Duration**)  
* Maximum duration permitted for each retry (**Max Duration**)  
* Factor by which to multiply the Duration in the event of a failed retry (**Factor**). A factor of 2 for example, attempts the second retry in 2 X 2 seconds, where 2 seconds is the Duration.

#### Synchronize resources
Synchonize Resource options allow you to selectively sync application resources. You can bypass sync settings at the application level, and directly select the resources you want to sync, by state or otherwise.  
* All resources regardless of their sync states
* Only out-of-sync resources
* Only selected resources

By default, Synchronize Resources displays and selects all resources in the application. 

   {% include 
   image.html 
   lightbox="true" 
   file="/images/applications/sync-manual-resources-form.png" 
   url="/images/applications/sync-manual-resources-form.png" 
   alt="Default settings for Synchronize Resources" 
   caption="Default settings for Synchronize Resources"
   max-width="70%" 
   %} 

You can search/filter resources using part of the resource names or regex strings, and then select the resources you want to sync.
For example, if you made changes to `api` resources or `audit` resources, type `api` or `audit` to locate the resources and then selectively sync those resources.

   {% include 
   image.html 
   lightbox="true" 
   file="/images/applications/sync-manual-resource-search.png" 
   url="/images/applications/sync-manual-resource-search.png" 
   alt="Selective sync in Synchronize Resources" 
   caption="Selective sync in Synchronize Resources"
   max-width="70%" 
   %} 

### Delete an application
Delete an application from Codefresh. Deleting an application deletes the manifest from the Git repository, and then from the cluster where it is deployed. When deleted from the cluster, the application is removed from the Applications dashboard in Codefresh.
 
>**Prune resources** in the application's General settings determines the scope of the delete action.  
When selected, both the application and its resources are deleted. When cleared, only the application is deleted. For more information, review [Sync settings]({{site.baseurl}}/docs/deployment/create-application/#sync-settings).  
Codefresh warns you of the implication of deleting the selected application in the Delete form. 

1. In the Codefresh UI, go to the [Applications dashboard](https://g.codefresh.io/2.0/applications-dashboard){:target="\_blank"}.
1. Select the application to delete.
1. Click the three dots for additional actions, and select **Delete**.
  
  {% include 
   image.html 
   lightbox="true" 
   file="/images/applications/app-delete-option.png" 
   url="/images/applications/app-delete-option.png" 
   alt="Delete application" 
   caption="Delete application"
   max-width="80%" 
   %} 

  Pay attention to the _impact of the delete action_ for the selected application that Codefresh displays.

   {% include 
   image.html 
   lightbox="true" 
   file="/images/applications/delete-app-prune-affects.png" 
   url="/images/applications/delete-app-prune-affects.png" 
   alt="Prune setting impact on deleting application" 
   caption="Prune setting impact on deleting application"
   max-width="70%" 
   %} 

{:start="4"}
1. To confirm, click **Commit & Delete**.

{::nomarkdown}
<br><br>
{:/}




