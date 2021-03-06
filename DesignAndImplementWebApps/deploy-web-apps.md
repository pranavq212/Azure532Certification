# Deploy Web Apps

## Define deployment slots
  * Deploy staging deployment slot before swapping it with the production slot. Different version of app to different URLs and then swap between slots. (-dev, -staging, etc.)
  * Eliminates downtime(warm up), traffic redirection, can be __Auto Swapped__
  	- Auto Swap is good for continuous deploy once servers are warmed up.
  * When slot configurations are cloned, some settings get cloned (slot specefic such as app configurations, connection strings) others do not (certificates).  Can mark configurations as slot specific.
  * Make sure slot settings are correct before swapping with production!
  * Multi-phase swap: swap configuration to see what happens first, then code.
  ```powershell
  New-AzureRmWebAppSlot -ResourceGroupName [resource group name] -Name [web app name] -Slot [deployment slot name] -AppServicePlan [app service plan name]

  ```

  * Links
  	- [Set up staging environments for web apps in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-staged-publishing/)
  
## Roll back deployments
  * Any errors are identified in production after a slot swap, roll the slots back to their pre-swap states by swapping
  * Some apps may require custom warm-up actions. The `applicationInitialization` configuration element in `web.config` allows you to specify custom initialization actions to be performed before 
  * Delete a deployment slot

  ```powershell
  Remove-AzureRmResource -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots –Name [web app name]/[slot name] -ApiVersion 2015-07-01
  ```

  * Swap a deployment
  ```bash
  $ azure site swap webappslotstest
  ```

## Implement pre and post deployment actions
  * [Kudo](https://github.com/projectkudu/kudu/wiki) is the engine behind git deployments in Azure Web Apps
  * Add an App Setting of `POST_DEPLOYMENT_ACTION` or `PRE_DEPLOYMENT_ACTION` and set the value to the patch of the .cmd, .ps1 or batch file to run after deployment
  * Can have multiple scrips run by setting the `SCM_POST_DEPLOYMENT_ACTIONS_PATH` setting.  The default path is `site\deployments\tools\PostDeploymentActions`
  * Can override the default deployment actions with having a `.deployment` files in root directory
  * Can use custom deployment script generator/`.deployment` file by issueing `azure site deploymentscript [options]`

## Create, configure and deploy a package
  * **Quick Create** and **Custom Create** are two waysto create and deploy a cloud service package
  *  Quick Create, you can create a cloud service just by specifying its URL and the region where it will be physically hosted. If you choose Custom Create, you can immediately publish a cloud service by specifying a package (.cspkg file), a configuration (.cscfg) file, and a certificate.
  * A cloud service is created from three components, the service definition (.csdef), the service config (.cscfg), and a service package (.cspkg). Both the ServiceDefinition.csdef and ServiceConfig.cscfg files are XML-based and describe the structure of the cloud service and how it's configured; collectively called the model. The ServicePackage.cspkg is a zip file that is generated from the ServiceDefinition.csdef and among other things, contains all of the required binary-based dependencies. Azure creates a cloud service from both the ServicePackage.cspkg and the ServiceConfig.cscfg.
  * __Service Definition__ The cloud service definition file (.csdef) defines the service model, including the number of roles.
  * __Service Configuration__ The cloud service configuration file (.cscfg) provides configuration settings for the cloud service and individual roles, including the number of role instances. The configuration values for the cloud service can be changed while the cloud service is running.
  * __Service Package__ The service package (.cspkg) contains the application code and configurations and the service definition file
  * The service definition file defines the runtime settings for your application including what roles are required, endpoints, and virtual machine size. The service configuration file configures how many instances of a role are run and the values of the settings defined for a role.
  * May need to configure certificates, if role instances (VMs) allow remote deskop as well as monitoring with Azure Diagnostics.
  * To upload a package, create new Cloud Service -> then upload the `.cspkg` file and `.cscfg` file. If the package was signed, upload the certificates `.pfx` file and provide its password.
  * You can use the __CSPack__ command-line tool (installed with the Azure SDK) to create the package file as an alternative to Visual Studio
  * Links
  	- <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-how-to-create-deploy-portal>
  	- <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-model-and-package>
  	- <https://azure.microsoft.com/en-us/documentation/articles/vs-azure-tools-publishing-a-cloud-service>
  	- <https://azure.microsoft.com/en-us/documentation/articles/vs-azure-tools-azure-project-create>
  	- <https://azure.microsoft.com/en-us/documentation/articles/vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio>

## Create App Service plans
  * Options: Web Apps, Mobile Apps, Logic Apps or API Apps
  * Pricing teirs: __Free, Shared, Basic, Standard and Premium__
  * Both apps and plans are contained in a resource group.
  * The ability to have multiple App Service plans in a single resource group allows you to __allocate different apps to different physical resources__. i.e. dev, test, production, different regions.
  * For a new app, creating a new resource group, plan, and app is the right choice. For an existing application, add it to the existing resource group.
  * If this new app is going to be resource intensive and have different scaling factors than the other apps hosted in an existing plan, it is recommended to isolate it into its own app service plan.
  * If you want to create a new app in a different region, and that region doesn't have an existing plan, you will have to __create a new plan in that region to be able to host your app there__.
  * App Service Plan, click + Create New, type the App Service plan name and select an appropriate __Location__. Click __Pricing__ tier and select an appropriate pricing tier for the service
  * __The app service plan will determine the location, features, cost and compute resource for your app__. It is the container for your app.
  * Links
  	- <https://azure.microsoft.com/en-us/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/>

## Migrate Web Apps between App Service plans
  * Having the capacity to move apps across plans also allows you to change the way resources are allocated across the bigger application.
  * Apps can be moved between plans as long as the plans are in the __same resource group and geographical region.__
  * To move an app to another plan, navigate to the app you want to move. In the settings menu look for __Change App Service Plan__.
  * Can move or clone apps between plans. __Clone if you want to move the app to a different region.__  Clone App option is in the tools blade.
  * Cloning and app has [limitations](https://azure.microsoft.com/en-us/documentation/articles/app-service-web-app-cloning-portal). Some are Traffic Manager settings, auto scale settings, App Insights and more. Cloning is only supported in __premium__ tier app service plans.  
  * Content and certificates are cloned.  Some options such as app settings, connection strings, domains can be toggled to be cloned as well.

## Create a Web App within an App Service plan
  * Use the UI. Select a subscription, resource group, app service plan... not much more to it.
  * App Service plans represent a set of features and capacity that you can share across your apps. App Service plans give you the flexibility to allocate specific apps to a given set of resources and further optimize you Azure resource utilization. This way, if you want to save money on your testing environment you can share a plan across multiple apps. You can also maximize throughput for your production environment by scaling it across multiple regions and plans.
  * An App Service Environment is a Premium service plan option of Azure App Service that provides a fully isolated and dedicated environment. ASE's can have many app service plans. The URL of a web app in an ASE is: [sitename].[name of your App Service Environment].p.azurewebsites.net
  * When you select pricing, the __price charged is applied to the App Service plan__ rather than to the individual apps
  * Links
  	- <https://azure.microsoft.com/en-us/documentation/articles/app-service-app-service-environment-intro>