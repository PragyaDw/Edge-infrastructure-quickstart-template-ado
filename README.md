# Edge Infrastructure QuickStart Template For Scaling (Preview) 
  ----- Accelerate, Automate, and Simplify the IaC setup using your familiar tools
## Overview

This Quick-Start template simplifies your Infrastructure as Code journey with Azure edge products throughout their lifecycle. It includes a few Terraform modules (**AKS Arc**, **Azure Stack HCI**, **Arc Site Manager** and **Arc extensions**), a scalable hands-on repository structure and the automation tools to streamline the setup of the infrastructure configurations for production scaling.

### What You'll Get

By using this template, you can get all of the followings inside a single PR under your GitHub account

* a scalable and extendible repository structure following the DevOps best practice

    <details>

    <summary><b>Repo Structure</b></summary>

    <img src="doc/img/repoStructure.png" alt="repoStructure" width="800"/>

    ```PROJECT_ROOT
    │
    ├───.azure
    │   │   backendTemplate.tf              // Backend storage account config file
    │   │
    │   └───hooks
    │           pre-commit                  // Git hook to generate deployment workflow and set backend
    │
    ├───.github
    │   └───workflows
    │           site-cd-workflow.yml        // Set up CD pipeline
    |           terraform-plan.yml
    │
    ├───dev                                 // The first stage to deploy
    │   └───sample
    │           backend.tf
    │           main.tf                     // Main configuration file for the site
    │           provider.tf
    │           terraform.tf
    │           variables.tf
    │
    ├───modules
    │   ├───base                            // Base module of all sites
    │   │       main.tf
    │   │       variables.tf
    │   │
    │   ├───hci                             // Module to manage HCI clusters
    │   │
    │   ├───hci-extensions                  // Module to manage HCI extensions                                                                     
    │   ├───hci-provisioners                // Module to connect servers to Arc
    │   │───aks-arc                         // Module to manage AKS Arc clusters
    │   └───hci-vm                          // Module to manage HCI VMs
    │   └───site-manager                    // Module to manage site-manager
    │
    ├───prod                                // prod stage sites are deployed after qa stage
    │   │
    │   └───prod1
    │
    └───qa                                  // qa stage sites are deployed after dev stage
        │
        └───qa1
    ```

    Base module contains the global variables across all sites. Each stage and each site folder contains the local variables specific to the stage/site. Local settings can override the global settings.

    </details>

* organized variables with the prefilled values to boost your initial setup
    <details>

    <summary><b>Variables Structure</b></summary>

    | Variable Type           | Description                                                                                                     | Example             | Where to set value                                                                                       | Override Priority |
    | ----------------------- | --------------------------------------------------------------------------------------------------------------- | ------------------- | -------------------------------------------------------------------------------------------------------- | :---------------: |
    | Global Variables        | The values of the global variables typically are consistent across the whole fleet but specific for one product | `domain_fqdn` in HCI | Set in `modules/base/<product>.global.tf`. Add default value for variables.                          |        low        |
    | Site specific variables | The values of these variables are unique in each site                                                           | `starting_address`            | These variables must be set in the site `main.tf` file under each site folder                            |       high        |
    | Pass through variables  | The values of these variables are inherited from GitHub secrets                                                 | `subscription_id`    | `modules/base/<product>.misc.tf`                                                                     |                   |
    | Reference variables     | These variables are shared by 2 or more products                                                                | `location`          | Its definition can be found in `variables.<product>.*.tf` if its link is `ref/<product>/<variable_name>` |                   |

    </details>

* a customizable CD pipeline with the automations.
    <details>

    <summary><b>CI/CD Pipeline</b></summary>

    <img src="doc/img/CDPipeline.png" alt="CDPipeline"/>

    </details>

### Is This Right for You?

**Yes** if you want to:

* Create an initial site containing AKS Arc, HCI23H2 along with Arc extensions using Terraform
* If you have manually created a PoC site and wish to convert the PoC site settings into Terraform code.
* Replicate the settings from the above site multiple times
* Integrate the above settings with CI/CD pipeline using GitHub Actions
* Automate all of the above scenarios

**No** if you want to:

* Create single AKS Arc or HCI instance using Terraform. Although this template contains the Terraform module for each of them, we are still waiting to officially publish them into public Terraform registry. You are welcome to use this repository for testing and exploration. For production usage, please contact arcIaCSupport@microsoft.com for each module's status.
* Use any other IaC tool such as Bicep or ARM to provision your Azure resources. We are working on our roadmap. Please stay tuned for future releases.

### Supported scenarios

```mermaid
flowchart LR;
    A[Fork QuickStart Repo] --> B["`Finish setup 
    in Getting-Started`"];
    B --> F[Uncomment sample code];
    F --> G["`Input values 
    for your first site`"];
    G --> J["`Move shared 
    paramteres to global`"];
    J --> K(["`Scale more sites
    by sample module`"]);
    K --> L[Get scaled sites];
```

### Supported Azure edge resource types

<details>

<summary><b> Supported Azure edge resource types</b></summary>

* [Azure Stack HCI, version 23H2](https://learn.microsoft.com/en-us/azure-stack/hci/whats-new)
* [Azure Stack HCI extensions](https://learn.microsoft.com/en-us/azure-stack/hci/manage/arc-extension-management?tabs=azureportal)
* [Azure Kubernetes Service (AKS) enabled by Azure Arc](https://learn.microsoft.com/en-us/azure/aks/hybrid/)
* [Arc Site Manager](https://review.learn.microsoft.com/en-us/azure/azure-arc/site-manager/overview?branch=release-preview-site-manager)
* [Azure resource group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview)

</details>

## Getting started

This repository implements AD preparation and Arc connection. Follow the instructions below to set up the rest of the components.

**Steps**: [Getting-Started](./doc/Getting-Started.md)

## Scenario 1: Create your first site from scratch using quick-start template, then scale more sites

### Create your first site from scratch using quick-start template

**Overview**: Ready to deploy your first with AKS Arc on HCI23H2 along with Arc extensions? It's the right place for you.
This scenario provides a quick and efficient way to establish a new site with edge resources with a predefined infrastructure template.

**Steps**: [Create your first site](./doc/Add-The-First-Site.md)

**Expected outcome**:

* A PR containing Terraform code set up for AKS Arc, HCI, Arc extensions under a single resource group
* A PR containing a pre-defined CI/CD pipeline with the 3 stages: Dev, QA, Prod
* Provisioning action will happen in your side (*merge the PR to `main`*)

### Scale more sites

**Overview**: Configure scaling settings based on the parameters defined in the previous steps.

**Steps**:

1. Create site secrets in the key vault.
   - `<site>-localAdminUser`: The admin user name of HCI hosts.
   - `<site>-localAdminPassword`: The admin user password of HCI hosts.
   - `<site>-deploymentUserPassword`: The password of deployment user which will be created during HCI deployment.
2. Copy and paste the first site to the new site.
3. Edit parameters for the new site in `main.tf`

## Scenario 2: Import your 22H2 cluster into IaC code, then upgrade to 23H2

**Overview**: If you already have a 22H2 cluster modeled within a resource group, this scenario will import them into Terraform modules, then upgrade the cluster to 23H2.

**Steps**:

* Prepare nodes, and validate solution upgrade [readiness](https://learn.microsoft.com/en-us/azure/azure-local/upgrade/about-upgrades-23h2)
    * Sometimes, it may be necessary to [remediate obsolete LCM packages](https://aka.ms/RemediateLCMPreviousZip)
* Create a branch, and modify main.tf, where `"ClusterUpgrade"` should be specified for `operation_type`.
    * `"InfraOnly"` is currently the only supported `configuration_mode`
    * `enable_provisioners` is currently not supported for `"ClusterUpgrade"`; verify the following roles have already been assigned to the nodes (Machine - Azure Arc)
        - KVSU   = "Key Vault Secrets User"
        - ACMRM  = "Azure Connected Machine Resource Manager"
        - ASHDMR = "Azure Stack HCI Device Management Role"
        - Reader = "Reader"
```
    operation_type = "ClusterUpgrade"
    configuration_mode = "InfraOnly"
    enable_provisioners = false
```
* Uncomment imports.tf
    * If the naming convention does not suit your needs, modify `naming.tf` under `modules/base`

> [!IMPORTANT]
> If the naming is incorrect, the resources imported will be destroyed and replaced by Terraform. Please double confirm the naming.

* Appply the changes using automation in the same way.

**Expected outcome**:

* The cluster is upgraded to 23H2.
* Arc bridge and Custom location are available in the resource group.

## Enable opt-in features for all sites

Any change merged into `main` branch will trigger the update pipeline. If the change fails in early stages, the deployment will be blocked so that this failure will not affect the production sites.

Following tutorials help you to turn on opt-in features:

* [Add HCI Insights](./doc/Add-HCI-Insights.md)
* [Add New Sites with Arc Site Manager](./doc/Add-Site-Manager.md)
* [Add HCI VM by Marketplace Windows Server Image](./doc/Add-HCI-VM.md)

## Advanced topics

- [Customize Stages](./doc/Customize-Stages.md)
- [Update global parameters](./doc/Edit-Global-Parameters.md)
- [Disable Telemetry](./doc/Disable-Telemetry.md)
- [Untrack Resources from The Repository](./doc/Untrack-Resources.md)
- [View your CI/CD pipeline running status](./doc/View-pipeline.md)
- [TroubleShoot](./doc/TroubleShooting.md)

## Ask for support

[Open issue](https://github.com/Azure/Edge-infrastructure-quickstart-template/issues/new) or contact arcIaCSupport@microsoft.com for any issue or support

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more information. This repository includes a script that will download binary files into your environment, which remain subject to your relevant customer agreement with Microsoft.

## Disclaimer  

'Preview Terms'. This repository (the "Preview") is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/). Unless otherwise noted, Customer should not use the Preview to process Personal Data or other Data that is subject to legal or regulatory compliance requirements.

'Confidentiality'.The Preview and any associated materials or documentation are confidential information and subject to obligations in your Non-Disclosure Agreement with Microsoft.

This repository is provided "as-is" without any warranties or support. Use at your own risk. Always test in a non-production environment before deploying to production.  

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit [Microsoft opensource](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
