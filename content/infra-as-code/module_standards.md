---
title: "Module_standards"
date: 2022-04-19T12:54:39+02:00
draft: false
---

# Authors
  - [Bence Bánó](mailto:bence.bano@lederworks.com)
<hr>

[[_TOC_]]
# Background
Infrastructure as Code module standards are available here. 

The driver behind the changes are the followings:
- The increasing complexity of the IaC modules are making updating and testing the code more and more difficult and time consuming.
- There are need for common standards how these modules are developed, in order to make integration of these parts into each other seamless and _EASY_.
<hr>

# Goals
- Develop small sub-modules, and build up complex deployments with so called wrapper-modules.
- Develop reusable terratests, as a go module, based on [Gruntworks Terratest suite](https://terratest.gruntwork.io/)
- Develop automated README.MD generation with [terraform-docs](https://terraform-docs.io/)
- Develop automated CHANGELOG.MD generation - tool under discussion.
- Follow DevSecOps best practices as a default deployment model, so consumers having _secure by default_ deployments, which meets the requirements of Defender for Cloud security score and Industry Best Practices.
<hr>

# Module Standards

## Naming
We are following the default naming conventions enforced on the [Terraform Registry](https://registry.terraform.io/) for terraform code.
Each sub- and wrapper-module needs its own repository in the [LederWorks Organization](https://github.com/LederWorks) based on the following rules:
- Every word is delimited by hyphens, eg. _-_, except the _purpose_, which is a suffix for the repository name and using snake_case
- The first word of every module is the language the majority of the code is written, eg. _terraform_, _bicep_, _ansible_, _golang_ etc.
<hr>

### Terraform Sub-Modules
- The second word is the provider the majority of the code is written in, eg. _azurerm_, _azuread_, _kubernetes_, _helm_ etc.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The fourth word is the category the resource under on the [Terraform Registry](https://registry.terraform.io/) documentation, eg. _compute_, _container_, _storage_ etc. If that is more than 2 word eg. _Logic App_, the do not use any delimiters, write in a single word, eg. _logicapp_.
- Any subsequent words should clearly state, what that module is used for and delimited by _. If it is deploying a Linux VM, then it should be linux_vm.

Example: terraform-azurerm-easy-container-aks_cluster

### Terraform Wrapper-Modules
- The second word is the provider the majority of the code is written in, eg. _azurerm_, _azuread_, _kubernetes_, _helm_ etc.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The fourth word is _wrapper_.
- Any subsequent words should clearly state, what that module is used for and delimited by _. If it is deploying a Private AKS Cluster, then it should be private_aks_cluster.

Example: terraform-azurerm-easy-wrapper-public_aks_cluster
<hr>

### Bicep Sub-Modules
- The second word is the second part of the [ARM API Documentations](https://docs.microsoft.com/en-us/azure/templates/) resource, eg. for Microsoft.Network/ it is going to be _network_.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- Any subsequent words should clearly state, what that module is used for and delimited by _. If it is deploying a DataBricks workspace it is going to be databricks_workspace.

Example: bicep-managedidentity-easy-user_managed_identity

### Bicep Wrapper-Modules
- The second word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The third word is _wrapper_.
- Any subsequent words should clearly state, what that module is used for and delimited by _. If it is deploying a DataBricks deployment in an island vnet, then it is going to be databricks_isolated_vnet.
  
Example: bicep-easy-wrapper-databricks_isolated_vnet
<hr>

### GoLang Modules
As Go Modules in general importing several packages, I do not think that we need to include this information in the name of the respective repository.
- The second word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- Any subsequent words should clearly state, what that module is used for. If it is used for terratests, then it should be _terratests_.

Example: golang-easy-terratests
<hr>

## Principles

### Terraform
As a general rule, keep it simple! If you write a module only put in what is needed for minimum functionality, all extras needs to be handled in other modules, so we can maintain a sustainable ecosystem.

#### Input Variables
- Variables are using snake_case syntax.
- Every sub-module variable name should be prefixes with the last section of the module name eg. the _purpose_, delimited by underscores. Wrapper-modules needs to have a separate variable_%sub-module name%.tf file and the same variable names should be used there.
    ```hcl
    #Module Name: 
    terraform-azurerm-easy-wrapper-aks_alert

    #Variable Names:
    aks_alert_container_cpu_percentage
    aks_alert_container_memory_percentage
    aks_alert_node_cpu_percentage
    aks_alert_node_memory_percentage
    ```
- Always have a description for each of your variables, as terraform-docs will use this to generate the README.MD
- Create a validation block for your variables whenever possible. This will prevent user errors. Make sure that your error_message clearly states what is the problem.
- In sub-modules always input object properties and not whole objects, as that renders a for_each loop on the module unusable due to limitations in terraform language.
- In wrapper-modules, you can input a whole object (preferrably outputted from one of the sub-modules used) and use the below technique to grab only what you need from it. There is a limitation, if the object output contains sensitive data, all output properties will inherit it too, even if it's not sensitive, and you can't use it for count and for_each operations in calling wrapper-modules. In this case you also need to output the required object properties as well.
    ```hcl
    #Sensitive value can't be used in subsequent wrapper modules in terraform functions:
    output "aks" {
        value = azurerm_kubernetes_cluster.aks_cluster
        sensitive = true
    }

    #So We output the used object properties as well:
    output "name" {
        value = azurerm_kubernetes_cluster.aks_cluster.name
    }
    output "id" {
        value = azurerm_kubernetes_cluster.aks_cluster.id
    }
    output "fqdn" {
        value = azurerm_kubernetes_cluster.aks_cluster.fqdn
    }
    output "identity" {
        value = azurerm_kubernetes_cluster.aks_cluster.identity[0]
    }
    output "kubelet_identity" {
        value = azurerm_kubernetes_cluster.aks_cluster.kubelet_identity[0]
    }

    ```
- Use common variables to get all the specific details for a given deployment such as, where, who, why and what. Also for a Tag input, with team specific tagging. Common vars does not needs to be prefixed with the "purpose" os the sub-modules.
    ```hcl
    #variables.tf

    #### Subscription ####
    variable "subscription_id" {
        description = "ID of the Subscription"
        type        = string
        validation {
            condition     = can(regex("\\b[0-9a-f]{8}\\b-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-\\b[0-9a-f]{12}\\b", var.SubscriptionID))
            error_message = "Must be a valid subscription id. Ex: 9e4e50cf-5a4a-4deb-a466-9086cd9e365b."
        }
    }

    #### Resource Group ####
    variable "resource_group_name" {
        description = "Resource Group Name"
        type        = string
    }

    #Use a data source to get the location and use that for every other resource deployment
    data "azurerm_resource_group" "resource_group" {
        name = var.resource_group_name
    }

    resource "azurerm_example_resource" "example" {
        name                          = "example"
        location                      = data.azurerm_resource_group.resource_group.location
        resource_group_name           = data.azurerm_resource_group.resource_group.name
    }

    #### Tags ####
    variable "tags" {
    description = "BYO Tags, preferrable from a local on your side :D"
    type        = map(string)
    }
    ```
- All the module names should be feed from the top level, where the deployment called and propagated down to the lowest level of the submodules. These are preferrably coming from a separate naming module such as https://github.com/Azure/terraform-azurerm-naming.
<hr>

#### Outputs
- Always use sensitive=true if you need to. Terraform 0.15.x and above will fail your plan, but on lower versions sensitive data is outputted in clear text.
- Output complete objects, and it's properties only if it is _sensitive_ and going to be called with a terraform function on the wrapper level. It is limiting the number of objects in the statefile, also can be better consumed in wrapper-modules.
    ```hcl
    #outputs.tf

    output "aks" {
        value     = azurerm_kubernetes_cluster.aks_cluster
        sensitive = true
    }

    output "udr" {
        value = azurerm_route_table.udr
    }
    ```
<hr>

#### Locals
- Locals are using snake_case syntax.
- Build up tags as locals, and use these in your code instead of variables. If you tag each level of modules properly each created resource will inherit the tag of all the modules used in it's creation, just like a block-chain. It is fantastic as you can write exclusion policies for certain security use cases based on the resource tags. For example you do not enforce https_only on an azure storage account, which you use as an NFS file share, but everywhere else you can prevent deployments with a deny policy.
    ```hcl
    #locals.tf

    locals {
    #Tags
    tags = merge({
        creation_mode          = "terraform",
        terraform-azurerm-easy-container-aks_cluster = "True"
        update_timestamp       = timestamp(),
        environment            = var.environment_ type
    }, var.Tags)
    }
    ```
- 
<hr>

#### Providers
- Do not set providers inside a module, as that results in an error, when you remove module call from deployments.
- Set the minimum required provider versions, in a providers.tf file. Each resource should be set to the latest [azurerm provider version](https://github.com/hashicorp/terraform-provider-azurerm/blob/main/CHANGELOG.md), where there is a change to the resource. If it is a wrapper-module, then this needs to be set to the highest of the consumed sub-modules.
    ```hcl
    #providers.tf

    terraform {
    required_providers {
        azurerm = {
        version = "= 2.99.0"
        }
    }
    }
    ```
- As of now we it is still under debate when the support fort azurerm 3.x.x should start, so we pin it to 2.99.0.
<hr>

#### Lifecycle Block
- Typically you want to add ignore_changes block for every object property, which can be modified outside of terraform either by users or Azure.
    ```hcl
    #main.tf

    resource "azurerm_kubernetes_cluster" "aks_cluster" {
    lifecycle {
        ignore_changes = [
        kubernetes_version,
        default_node_pool[0].orchestrator_version,
        default_node_pool[0].max_count,
        ]
    }
    ```
<hr>

#### Timeouts
- Overriding timeouts can help us fail fast. While the default timeouts are very forgiving in both terraform and azurerm api, we do not want an AKS deployment to timeout after 90 minutes, when the average deployment time is 5-8 minutes. So we override that to 30 minutes. This needs to be explored independently for every resource we introduce it. As a rule we want to use it for resources, where creation time average is more than 5 minutes or failure rate justifies it.
    ```hcl
    #variables.tf

    #### TimeOut ####
    variable "aks_timeout_create" {
        type        = string
        default     = "30m"
        description = "Specify timeout for create action. Defaults to 30 minutes."
    }
    variable "aks_timeout_update" {
        type        = string
        default     = "30m"
        description = "Specify timeout for update action. Defaults to 30 minutes."
    }
    variable "aks_timeout_read" {
        type        = string
        default     = "5m"
        description = "Specify timeout for read action. Defaults to 5 minutes."
    }
    variable "aks_timeout_delete" {
        type        = string
        default     = "30m"
        description = "Specify timeout for delete action. Defaults to 30 minutes."
    }

    #main.tf

    resource "azurerm_kubernetes_cluster" "aks_cluster" {
    timeouts {
        create = var.aks_timeout_create
        update = var.aks_timeout_update
        read   = var.aks_timeout_read
        delete = var.aks_timeout_delete
    }
    ```
- With this we can limit failure times, but also allow people to raise or lower it, if they feel they need to.
<hr>

## Versioning
- Modules are versioned with git release tags.
- Use semantic versioning. Syntax is major_version.minor_version.patch_version, eg. 1.0.0
- patch_version needs to be raised, whenever we fixing some issues or adjusting terraform code to mirror latest provider changes
- minor_version needs to be raised, whenever we are implementing a new feature. We should support preview features as well, so teams can opt-in.
- major_version needs to be raised, whenever we are introducing a breaking change, which requires rebuild of a given resource
- Due to the limitation that module sourcing can't use variables, wrapper-modules needs to use the same logic. When we version any of the sub-modules, the wrappers consuming it should be updated as well.
<hr>