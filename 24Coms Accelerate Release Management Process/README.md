# DevOps Hackfest with 24Coms #

## The hack team ##

The team for this engagement consisted of following key profiles from 24Coms and Microsoft evangelists. 

**24Coms:**

- Leo Winder – Owner/CTO
- Michael Vlaar –  Development Engineer
- Leone Keijzer – Development Engineer
- Ronald Koster – Development Engineer 

**Microsoft:**

- Valery Jacobs (@valeryjacobs) – Senior Technical Evangelist 
- Aleksandar Dordevic (@Alex_ZZ) – Principal Technical Evangelist

## Customer Profile ##

24COMS empowers enterprises, SOHO (Small Office Home Offices) customers and end-users to self-organize the dynamics of their personal and business communications and information sharing. Targeted communication from a Control Center to the apps on mobile devices to manage efficient information and communication sharing. Reduce waste of time on nonsense messaging, create efficiency and end2end value in projects and collaborations between groups, teams, locations and individuals. 

24Coms are focusing on creating software solutions for Digital Social Communication, Geo-fencing, Phone as a Sensor, Location Based Innovations and etc., both for consumer and enterprise customers. Their unique solution provides new possibilities, in modern communications, by adding Geo-tracking dimension to convention communication mechanisms, information exchange and alerting. This solution is called 24Coms, and it is growing number of customers and companies.

More at [https://24coms.com ](https://24coms.com/ )

Our journey started with a meeting between 24Coms and Microsoft where we talked about how to practice more agile software development by applying DevOps practices. During this meeting, we narrowed down the options to executable and doable steps that would be inline and helpful to their DevOps journey.  The decision was made to deliver a Value Stream Map (VSM) workshop followed by a DevOps hackfest. First we did a 1 day VSM workshop at mid-August 2016 followed by a 3-day hackfest in Mid-September 2016.

 
## Problem Statement ##

24Coms teamed-up with Microsoft to optimize and accelerate their overall delivery pipeline and capacity to provision new customers to their platform. To discover areas of improvement and to narrow down the focus areas for the hackfest the team meet and started their DevOps journey with a Value Stream Map workshop. 

Before starting the Value Stream Map process an agreement was made on what the focus of the workshop should be regarding the solution and its components. The 24Coms team wanted to focus on their recently refactored backend service called “Follow me”. Previously this was a big monolithic application and they turned this into a micro service based application. 

24Coms designed this solution as an Azure Service Fabric implementation and already they are running their production environment on this new architecture. The focus of the conversation with 24Coms was on the operations pipeline and we discovered that most of the impact during the hackfest could be at this backend service. The next step was the Value Stream Map workshop.

In the VSM workshop we captured the high-level conceptual architecture of the solution, so everyone could understand what the solution would look like and what the characteristics of a micro service implementation are. 

![alt tag](Images/1.jpg)

Originally the architecture was based on Cloud Services and a big dependency on Azure Service Bus. The performance limitations of this approach were severe and an alternative had to be found to improve scalability and ease of deployment.

With the original approach their performance limit was 250 msg/s which meant they had to scale on a per Service Bus namespace basis which would have caused a lot of management overhead and even would max out the subscription quota for the maximum amount of Service Bus namespaces as their throughput requirements are way beyond what is offered currently with this service. They target servicing up to 100.000+ devices which can potentially submit messages frequently depending on circumstances (alert situations, big events, etc.)

During earlier conversation with 24Coms Service Fabric was mentioned as it could offer the opportunity to build a solution on top of a framework that had reliability, scalability and upgrade flexibility build in from the start. 

The result of the migration to micro services were 4 different services.

- Client Service
- Tracker Service
- Geofence Service
- Notification Service

Client to these services are specialized tracking devices or smartphones running iOS, Android or Windows. Transport security and message level security are based on encryption and access tokens. All services are running within a Service Fabric cluster with a couple of dependencies on external services.

After drawing up the solution the discussion continued on what areas of improvement were the most important for 24Coms’ business.


![alt tag](Images/2.png)

The Value Stream Mapping workshop started with their CTO and development lead. We determined all the steps that were involved in their existing life-cycle management procedure. This gave us insights in the effort it took the company to get from a great idea to a successful deployment in production and how such a deployment would evolve over time as a result of measuring customer usage and feedback.

The outcome of the investigation were the 6 core steps they employ in their development practices and these would the subject of investigation for optimizing and automating the overall process.

![alt tag](Images/3.png)

Based on the VSM we identified areas of improvement. The process of development and releasing showed to most potential for improvement. Basically they were facing the challenge of a startup that needs to develop at a high velocity while at the same time handled a lot of changes to existing parts of the system as well as parts that were under construction. It was clear to a team that, if the aim was to have a more predictable, manageable and sustainable delivery process that most of the manual work needs to be converted into automated tasks. 

It should not be left unnoted that this process by itself was very fruitful for the team. They took the opportunity to look back and get a good grip on what they wanted and needed to change. 

![alt tag](Images/4.jpg)

The workshop resulted in a set of goals for the rest of the Ascend+ engagement:



- **Implement Continuous Integration and Continuous Deployment (CI/CD)**
	- Implement automated Builds of Development output
- **Implement Infrastructure as Code (IaC) and Configuration Management**
	- Declare Environments via ARM to achieve a better level of automatization and predictability. In addition, apply Infrastructure as Code and Release Management practices 
- **Implement Release Management (RM)**
	- Moving and Deploying the application to different environments during its lifecycle
- **Innovation of development process and people skill development**
	- Leverage innovation and accelerate future feature development by planning and scheduling development of new skills, practices & tools as a standard way of working and planning.
	- Creating smaller deployment batches, drive development towards continuous innovation and, as a result, let their business be more agile.

## Solution, steps and delivery  ##

The next step was a 3-day hackfest at the Microsoft office in Schiphol-Rijk. We did a couple of preparation call prior to the event so everybody could start well informed and fully equipped. The goals set during the VSM workshop were translated into concrete activities for the hackfest.


**High level goals:**
 
1.	Create a secure Azure Service Fabric cluster    
	- Certificate to secure the cluster
	- Azure Key Vault holding the certificate
	- ARM template to setup the full cluster (IaC)
2.	Connect VSTS & Azure Service Fabric
	- New VSTS project
	- Push code into the repositories
	- Create Builds definitions for multiple projects (CI)
	- Create Release definitions for multiple environments (CD/RM)
3.	Build, Ship & Test
	- Apply software changes and test the overall delivery pipeline
	- Check the versioning of the application and how it gets upgraded in Service Fabric
	- Test the platform including load testing using VSTS web test and virtual user resources

The rest of this chapter will describe the journey and learnings we encountered executing these tasks.

## Creating secure Azure Service Fabric ##
According to the plan the first step was creating an ARM template to provision a secure Azure Service Fabric cluster. At that point 24Coms were provisioning their clusters for test, staging and production manually using the Azure portal. To be more agile and in line with DevOps practices the template would be used as the base for the cluster that would be used during the hackfest.

Our starting point [was the Azure QuickStart templates at GitHub](https://github.com/Azure/azure-quickstart-templates), where we found an adequate template that was tuned to their specific needs. The template used was the [5 Node secure Service Fabric Cluster with WAD enabled](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype-wad).


> Copying and pasting ARM templates from Github can result in invalid json files. To prevent this any advanced text editor could be used to make sure there are no hidden characters present in the file. We have been using Visual Studio and Visual Studio Code at this hackfest when creating IaC.


![alt tag](Images/6.png)
*figure 3: arm template for the service farbric  cluster*

> Before deploying an ARM template, check the Core quota of the subscription. In case there are not enough cores available the error message notifying this might not be instantly visible and this could lead to confusion. 

To setup the security assets we used a very helpful PowerShell module on GitHub that helped with securing and populating an Azure Key Vault to secure communication with the Azure Service Fabric management endpoints. [https://github.com/ChackDan/Service-Fabric#microsoft-azure-service-fabric-helper-powershell-module ](https://github.com/ChackDan/Service-Fabric#microsoft-azure-service-fabric-helper-powershell-module)

The majority of the template was straight forward with one notable addition for linking the cluster to the certificate in the Azure Key Vault.

We discussed the options for the certificate to be used for securing the cluster. Since they need to hook multiple domain names for white-labeling their service to customers a wild card certificate would have made sense but 24Coms preferred not to use this type of certificate due to speculated security degradation involved. In the end we opted for the approach where the certificate would use add CN names. A 2K key was used instead of a 4k key because some tracking devices have too limited resources to handle these large keys.

Here a snippet from the ARM parameters file indicating the setup of the KeyVault based certificate:

        "certificateThumbprint": {
      "value": "99912345o458045827c723047234"
    },
    "sourceVaultvalue": {
      "value": "/subscriptions/<Sub ID>/resourceGroups/<Resource group name>/providers/Microsoft.KeyVault/vaults/<vault name>"
    },
    "certificateUrlvalue": {
      "value": "https://<name of the vault>.vault.azure.net:443/secrets/<exact location>"
    },

A more detailed description of these steps were published on Github and can be found [here](https://github.com/djzeka/VSTS-AzureServiceFabric/blob/master/docs/Creating%20secure%20Azure%20Service%20Fabric%20Cluster.md).

![alt tag](Images/7.png)
*figure 4: a secured service fabric cluster*

This concluded the first step where we had a complete template to be able to generate clusters for specific environments automatically.

## Connecting VSTS & Azure Service Fabric ##
Next up was the step to add a Visual Studio Team Services project that would hold all the sources, scripts and solution assets. 24Coms had separate repositories for the back-end and front-end applications and we went ahead by combining them within the same VSTS project to get a single unit of deployment and life-cycle management.

The opportunity was taken to restructure and clean up the solution to only include the necessary code base for the services. This should speed up the build time since all project in the solution will be built during CI/CD steps. 

>To be able to use the solution for the latest cluster version it needs to be updated to the latest SDK. The version can be looked up in de .sfproj file under the ‘ProjectVersion’ element. Visual Studio is capable of updating the project after the latest SDK is installed. Potential build error can be fixed by updating the service fabric NuGet packages in the solution.

Not all team members use Windows as their workplace operating system and we wanted to include the whole team to get familiar with Microsoft’s tooling. We opted to use Azure Dev Test labs to provide all team members with a pre-configured workspace that included Visual Studio 2015 and common tools used by the team. This step only took a brief moment because they only needed to select the ‘formulas’ they needed and 15 minutes later they were all up and running. These machines could be scheduled to turn off outside of office hours to save costs and waste of resources.

The ARM template was added to the solution as well to get an integrated repository that included Infrastructure as Code. This way the full history of every topology chance and the option to revert incorrect changes would be available.

![alt tag](Images/8.png)

## Creating build definitions ##

**Backend services:**

The next task was to setup a build definition for the micro service based back-end. Luckily a template is already available in VSTS so only the configuration had to be changed. The ‘Update Service Fabric App Versions’ step update the application and service versions for all projects that have been updated. This way an automated build and deploy will only touch things that have to be updated.

> To be able to filter out services that don’t have any code changes the solution projects have to be enabled for deterministic builds. This can be achieved updating the project files to include a *<Deterministic>True</Deterministic>* element in a *<PropertyGroup>* element. This makes sure the build output is binary identical so it doesn’t get picked up for an automated version update.

![alt tag](Images/9.png)

> Testing the build resulted in an error message. This turned out to be the result of a misconfigured build definition in the solution file. All projects need to output x64 binaries else there is a mismatch with the SDK’s assemblies which are built only for 64 bit environments.

**iOS Client application:**
Next up was setting up de CI/CD for the iOS mobile application. An extra step had to be added for a successfully automated build to manage the dependencies of the Objective-C Cacoa project (UI framework for iOS). The CacoaPod pod install command arranges the package management and initialization step.

![alt tag](Images/10.png)

Apple doesn’t allow builds of iOS of OSX application to happen on other platforms than OSX. For this reason, builds cannot run in hosted mode. Luckily there is a handy agent available that 24Coms used to setup a local machine to execute the builds. VSTS will orchestrate the process according to the build definition. The agent retrieves all assets needed to do the build and the output is send back for further release steps.

> There is a paid alternative that can be considered called MacinCloud. This service hosts genuine Mac hardware and offer build agents as a service. More info can be found [here](https://www.macincloud.com/pricing/build-agent-plans/vso-build-agent-plan).

![alt tag](Images/11.jpg)
*figure 5: the 24coms ios app communicating with the followme back-end*

**Android Application:**
Last up was the Android version of the 24Coms mobile application. Key points here were the addition of a Gradle step which triggers the build process and .PKG creation process. Gradle needs a build definition file to know how to build. 

After updating this file the builds still continued to fail. The reason for this was that the hosted build agent did not have the correct Agent version installed to support the Android project version (1.98.1). The alternative was to use the local build agent that already was setup for the iOS build. Custom build agents (local or cloud hosted) offer more flexibility in preinstalling software for builds. There are agent version for multiple platforms as can be found [here](https://github.com/Microsoft/vsts-agent).

![alt tag](Images/13.png)
![alt tag](Images/14.png)

## Creating release definitions ##
The template offered by VSTS for a service fabric release definition contains a single task that publishes an application to the cluster based on the selected publishing profile.  This publishing profile was matched with the corresponding environment the build definition was setup for. 

In preparation of this step a service endpoint item was created linking the deployment to the secured cluster management endpoints using the certificate in the Azure Key Vault.

![alt tag](Images/15.png)

> When adding new Azure Service Fabric “Service” at VSTS make sure you add https URL to your ASF cluster followed by the port e.g. *https://qwerty.westeurope.cloudapp.azure.com:19000/* to avoid errors and confusion.

![alt tag](Images/16.png)

**Back-end testing & discovery of external Clients:**
 
While part of the te
am was working on iOS, Android and GPS trackers and building and pushing new app versions and backend services. We also did a live test and checked if data flow from frontends is working.

We were using the dark websocket terminal tool (Chrome extension) to connect to the backend and pickup the “signals” from front end devices. The picture below shows what was captured – all 3 expected devices were successfully sending GPS data to newly provisioned test cluster.

![alt tag](Images/17.png)
*figure 6:using a socket tool to check incoming signals*

With this we reached the milestone where a newly updated & provisioned system was working as expected. 

**Beta testing:**

24Coms was using Apple’s TestFlight for beta testing and realized they had to implement and manage multiple tools for the various platform they support. 
![alt tag](Images/18.png)
*Figure 5: testflight beta/test app releases, ios only*

To include beta releasing as part the on-ramp to the DevOps practice feature flagging *HockeyApp* was introduced into the flow. The full migration to *HockeyApp* was out of scope for the 3-day hackfest but for 24Coms *HockeyApp* looked like a promising option and an improvement that would simplify their DevOps practice even further.

![alt tag](Images/19.jpg)
*figure 6: hockeyapp offering multiplatform app beta-testing and monitoring*

**Load Testing:**

Load testing often is an important step in making sure a scalable system actually has the characteristics we like to see. Although not part of the core priorities for the hackfest (due to the time investments needed to set this up) we did have a discussion on a recommended approach. VSTS offers cloud based load testing ranging from a simple HTTP call that can be configured and run multiple times to recorded and coded tests that can simulates large amounts of clients and devices. During the engagement we shared some best practices for writing a code based load test that can hit the Service Fabric cluster endpoints from a large pool of virtual users. With a simple click of a button in the VSTS portal a large scale load test can be triggered and the result are instantly available online. Using Application Insight to monitor the environment under test makes a lot of sense and for Service Fabric there is guidance on how to set it up (see resources).

## Conclusion ##
Before wrapping up the event a roundtable discussion held with the 24Coms team to get feedback on what we were able to achieve and how they experienced spending their time with Microsoft. The overall feedback was really positive. 

## Hackfest outcome ##
At the end of the Hackfest 24Coms had the core DevOps practices embedded in their company DNA. Changes in their market, that lead to new insights at the level of 24Coms’s management now can be turned in to new features quicker than ever, due to the following improvements:

- Code changes will lead to automatically updated and validated test, staging and eventually production environments. 
![alt tag](Images/20.png)
- The build steps for mobile applications are no longer isolated steps but will be part of a consistent and highly automated process.
- Deployment management can be done centrally using a single orchestrator.
- Updates to back-end services can be rolled out without downtime. Service Fabric handles the deployment and message/request routing.
- At the application level there is a resilient runtime that monitors services, acts on failures automatically, often by fixing them without human intervention.
- Resources running in the cloud are used effectively. By running multiple service in a coherent cluster each CPU payed for can be used to capacity.
- Azure Service Fabric allows us to push, mitigate and deliver new code and bug fixes fast
![alt tag](Images/21.jpg)
*figure 8: pushing bug fix with the ease and speed of azure service fabric*

Last learning and perhaps most important one. Just 1 day after we have finished our hack with 24Coms team they informed us about improvements they have made – they have applied learned and utilized full CI/CD/RM on their production and it looks like this:

![alt tag](Images/22.png)
Figure 9: Automated updates from VSTS to Service Fabric

## Appreciation ##
> It is exciting to see how quickly changes can be made if we tune ourselves in to the discovery and hacking mindset. Many thanks to the team for picking up the challenge, your partnership and will to drive this hackfest to a success – 24Coms, Thank you!

## Resources ##

Resource Manager, KeyVault and Azure Service Fabric useful links:

- [https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/)  
- [https://azure.microsoft.com/en-us/documentation/articles/resource-manager-keyvault-parameter/](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-keyvault-parameter/)
- [https://github.com/Azure/azure-quickstart-templates/ ](https://github.com/Azure/azure-quickstart-templates/ )
- [https://azure.microsoft.com/en-us/documentation/services/service-fabric/](https://azure.microsoft.com/en-us/documentation/services/service-fabric/)
  
Azure Service Fabric HOL’s and additional materials:

- [https://github.com/djzeka/VSTS-AzureServiceFabric ](https://github.com/djzeka/VSTS-AzureServiceFabric )
- [https://github.com/ChackDan/Service-Fabric#microsoft-azure-service-fabric-helper-powershell-module ](https://github.com/ChackDan/Service-Fabric#microsoft-azure-service-fabric-helper-powershell-module )

HockeyApp, Application Insights:

- [http://www.medic-consulting.com/2016/08/12/Application-Insights-and-Semantic-Logging-for-Service-Fabric-Microservices/](http://www.medic-consulting.com/2016/08/12/Application-Insights-and-Semantic-Logging-for-Service-Fabric-Microservices/)
- [https://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs](https://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs)

Build agents, XCode setup:

- [https://github.com/Microsoft/vsts-agent/blob/master/README.md](https://github.com/Microsoft/vsts-agent/blob/master/README.md)
- [https://www.visualstudio.com/en-us/docs/build/admin/agents/v2-osx](https://www.visualstudio.com/en-us/docs/build/admin/agents/v2-osx)
- [https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate)
- [https://www.visualstudio.com/docs/build/apps/mobile/xcode-ios](https://www.visualstudio.com/docs/build/apps/mobile/xcode-ios) 


