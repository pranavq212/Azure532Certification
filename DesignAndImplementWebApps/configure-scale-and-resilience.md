#Configure Web Apps for Scale and Resilience
  * Use the Azure Portal to scale your App Service plan from Free mode to Shared, Basic, Standard, or Premium mode then configuring certain settings. Should first remove the spending caps in place or risk your web app becoming unavailable if you reach your caps before the billing period ends.
  * Does not require your code to be changed or your applications to be redeployed. What are all the components of your application? Where are the bottle necks in the application? When load is applied to your app, what will break first? Number of users, location of users, peak times.
  * Scaling considerations: Is your Application Statefull? Stateless?
  * Links
      - [Scale a web app in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-scale/)

##Configure auto-scale using built-in and custom schedules
  * In the Portal, can _configure_ scale by choosing __Add Rule__ 'schedule and performance rules' such as 'CPU percentage > 80 (increase count by 1)' and 'CPU Percentage < 60 (decrease instance size by 1)'.
  * Can move the two sliders to define the minimum and maximum number of instances to scale automatically.
  * If you have one or more __linked SQL Server databases__ linked to your web app (regardless of App Service plan mode), you can quickly scale them based on your needs.
      - Setup __geo-replication__ to increase the high availability and disaster recovery capabilities in the Geo Replication blade.
  * __Bitness__ Basic, Standard, and Premium modes support 64-bit and 32-bit applications. The Free and Shared plan modes support 32-bit applications only.

##Configure by metric
  * Can create scale rules based on resource, metric name (__CPU, memory, disk queue, HTTP queue__), operator (greater than), threshold (80%), duration (10 minutes), Time aggregation (average), __Action__ (increase count by), Value (1), Cool down (10 minutes.)
  * Examples: Scale up by 1 instance if CPU is above 80% in the last 10 minutes, Scale down by 1 instance if CPU is below 50% in the last 30 minutes.

##Change the size of an instance
  * Choose Web App > Settings > Scale Up > choose priceing tier.
  * Scale out > Scable by: 'Manual' Then increase slider to number of instances (based on app service plan).
  * __D-series__ VM instances are designed to run applications that demand higher compute power and temporary disk performance.
  * __Dv2-series__, a follow-on to the original D-series, features a __more powerful CPU__.
  * The __A8/A10__ and __A9/A11__ virtual machine sizes have the __same capacities__. 
    - The A8 and A9 virtual machine instances include an __additional network adapter__ that is connected to a remote direct memory access (RDMA) network for fast __communication between virtual machines__. 
    - A10 and A11 virtual machine instances do not include the additional network adapter, do not require constant and low-latency communication between nodes.
  * General purpose __XS-XL__ scaled based on CPU, Memory, RAM
  * Memory intesive applications should use __A5 - A7__
  * Netowork heavy: __A8/A9__
  * CPU heavy: __A10/A11__
  * D-Series: Optimized computer, faster processors, feature solid state drives. __D1-D4__ General purpose, __D11-D14__ are memory intensive. __Dv2__ have more powerful CPUs
  * Can specify the Virtual Machine size of a role instance as part of the service model described by the service definition file:
  ```xml
  <WebRole name="WebRole1" vmsize="Standard_D2">
  ...
  </WebRole>
  ```

  * All machine sizes provide an application disk that stores all the files from your cloud service package; it is around 1.5 GB in size.
  * Links
    - [Cloud Services Sizes](https://azure.microsoft.com/en-us/documentation/articles/cloud-services-sizes-specs)

##Configure Traffic Manager
  * Traffic Manager is the __distribution of user traffic__. Increases apps __responsiveness/availability, cloud migration/hybrid on-premises, A/B testing, distribute traffic based on weighted values (based on geolocation)__. Traffic Manager uses profiles that are based on __monitoring and routing__.
  * Traffic Manager is a popular option for on-premises scenarios including burst-to-cloud, migrate-to-cloud, and failover-to-cloud. Applies intelligent policy engine to Domain Name System (DNS) queries for the domain names of your Internet resources such as directing end-users to the endpoint with the lowest network latency from the client. Traffic Manager receives an incoming __request from a client and locates the Traffic Manager profile__.
  * Zero downtime: wait for the endpoint to complete the servicing of existing connections. When there is no more traffic to the endpoint, you update the service on that endpoint and test it, then re-enable it.
  * Configure to determine which endpoint should service the request based on a DNS query. The DNS resource record for the __company domain points to a Traffic Manager domain name__ maintained in Azure Traffic Manager. This is achieved by using a CNAME resource record that maps the company domain name to the Traffic Manager domain name. i.e. __contoso.com IN CNAME contoso.trafficmanager.net__.
  * Traffic Manager uses the specified traffic __routing method and monitoring status to determine which endpoint__ should service the request. Then  returns a CNAME record that maps the Traffic Manager domain name to the domain name of the endpoint. The user's DNS server resolves the endpoint domain name to its IP address and sends it to the user.
  * Weighting could be used to distribute a small percentage of traffic to a new or trial deployment for testing or customer feedback.
  * User continues to interact with the chosen endpoint until its __local DNS cache entry expires for the duration of their Time-to-Live (TTL)__. TTL value controls how often the client’s local caching name server will query the Azure Traffic Manager DNS system for updated DNS entries. Default value of 300 seconds (5 minutes).
  * When testing, you must either disable client-side DNS caching or clear the DNS cache between each attempt to ensure that a new DNS name query gets sent.
  * Seven steps to configure Traffic Manager
    1. __Deploy Endpoints__: Cloud services or websites
    2. __Choose DNS name/prefix for Traffic Manager Profile__
    3. __Choose the monitoring conifguration__
      + Wont route to endpoints that aren't available.
    4. __Choose load balancing method__
      + Failover, performance, round robin.
    5. __Create Traffic Manager profile__
    6. __Test profile__
    7. __Point the domain name to the Traffic Manager Profile__
  * Three routing method types. Each Traffic Manager profile can __only use one routing method at a time__.
    - __Failover__: Automatically redirect traffic when __failure detected__ at primary endpoint(s), revert to secondary, go down the __ordered list__ of endpoints until one works.
    - __Performance__: when have endpoints in different geographic regions, used the "closest" endpoint in terms of a __latency__. Traffic Manager locates the row in the _Internet Latency Table_ for the IP address of the incoming DNS request, resolves domain name to IP address of fastest route to the client.
    - __Round Robin__: based on _weighted metrics or random_, __distribute load__ between geographic locations/regions.
      + Gradual application upgrade: Allocate a percentage of traffic to route to a new endpoint, and gradually increase the traffic over time to 100%.
      + Application migration to Azure: Create a profile with both Azure and external endpoints, and specify the weight of traffic that is routed to each endpoint.
      + Cloud-bursting for additional capacity: Quickly expand an on-premises deployment into the cloud when you need extra capacity.
  * Can configure Traffic Manager settings using the Azure classic portal, with REST APIs, and with Windows PowerShell cmdlets. Some settings are not available using just one method. i.e. configuring external endpoints (type = ‘Any’) for Round Robin routing.
  * Can use __Quick Create__ in Azure Portal to create Traffic Manager profile.
  * You can use periods to separate parts of your DNS prefix such as web.contoso.trafficmanager.net, bill.contoso.trafficmanager.net. Bad naming can make finding the correct endpoint, managing the profile difficult.
  * All endpoints should be in the __same subscription__ where you are creating the profile. You can add endpoints from different subscriptions to a profile as __external endpoints__. External endpoints need to be manually removed.
  * In order for monitoring to work correctly, you must set it up the same way for every endpoint that you specify in your Traffic Manager profile
  * Endpoints in a __production environment only__ are available. You cannot direct to endpoints running in a staging environment.
  * __Disable endpoints or profile for temporary changes__, rather than changing your configuration. Useful for temporarily removing an endpoint that is in maintenance mode or being redeployed.
  * Can specify the name of another Traffic Manager profile as an endpoint, a practice known as __nested profiles__, child/parent relationship. The Traffic Manager profile name is its DNS name, such as contoso-usa.trafficmanager.net. i.e. top tier routing based on domain uses a the performance routing method which points to the best data center nested profile. The data center nested profile may then use the round robin traffic routing method and then return the appropirate DNS record.
  * To disable a profile, change the DNS resource record on your Internet DNS server so that it no longer uses a CNAME resource record that points to the domain name of your Traffic Manager profile. Then disable the Traffic Manager profile in Azure.
  * Before you delete a Traffic Manager profile, make sure a CNAME record isn't still pointing to it.
  * Can view change history for profile. Management Services > Operation Logs.
  * Taffic Manager CmdLets:  
  `(Add|Disable|Enable|Get|New|Remove|Set)-AzureTrafficManagerEndpoint`, `(Set|Remove)-AzureTrafficManagerEndpoint)`, `Test-AzureTrafficManagerDomainName`
  * Can run commands to get help:  
  `Get-Help <cmdlet name> [-Detailed] [-Examples] [-Full]`
  * Links
    - [Traffic Manager](https://azure.microsoft.com/en-us/services/traffic-manager)
    - [What is Traffic Manager](https://azure.microsoft.com/en-us/documentation/articles/traffic-manager-overview)
    - [Nested Profiles](https://azure.microsoft.com/en-us/blog/new-azure-traffic-manager-nested-profiles)
    - [Manage an Azure Traffic Manager profile](https://azure.microsoft.com/en-us/documentation/articles/traffic-manager-manage-profiles)
    - [Taffic Manager Routing Methods](https://azure.microsoft.com/en-us/documentation/articles/traffic-manager-routing-methods)
