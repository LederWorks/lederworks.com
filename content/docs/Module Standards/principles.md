---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"] 
title: "Module Principles"
linkTitle: "Principles"
weight: 22
description: Module principles
---
<hr>

## Principles
As a general rule, keep it simple! If you write a module only put in what is needed for minimum functionality, all extras needs to be handled in other modules, so we can maintain a sustainable ecosystem.

### Input Variables
- Variables are using snake_case syntax.
- Every sub-module variable name should be prefixed with the last section of the module name eg. the _purpose_, delimited by underscores. Wrapper modules needs to have a separate variable_%sub-module name%.tf file and the same variable names should be used there, as in the called Bricks.
    ```hcl
    #Module Name: 
    terraform-azurerm-easy-wrapper-aks-alert

    #Variable Names:
    aks_alert_container_cpu_percentage
    aks_alert_container_memory_percentage
    aks_alert_node_cpu_percentage
    aks_alert_node_memory_percentage
    ```
- Always have a description for each of your variables, as terraform-docs will use this to generate the README.MD
- Create a validation block for your variables whenever possible. This will prevent user errors. Make sure that your error_message clearly states what is the problem.
- In sub-modules prefer inputting object properties over whole objects, as that makes for_each loops much simpler in HCL.
- In wrapper-modules, you can input a whole object (preferrably outputted by the Bricks as maps) and use the below technique to grab only what you need from it. There is a limitation, if the object output contains sensitive data, all output properties will inherit it too, even if it's not sensitive, and you can't use it for count and for_each operations in calling wrapper-modules. In this case you also need to output the required object properties 
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
    #variables-common.tf

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
    variable "resource_group_object" {
        description = "Resource Group Object"
        type        = any
    }

    #### Tags ####
    variable "tags" {
    description = "BYO Tags, preferrable from a local on your side :D"
    type        = map(string)
    }
    ```
- All the resource names should be feed from the top level, where the deployment called and propagated down to the Bricks. These are preferrably coming from a separate naming module such as https://github.com/Azure/terraform-azurerm-naming.

### Outputs
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
- When you are working with resource types, which having direct dependency on each other - for example a Virtual Machine and it's NIC, create and output complex maps. This way later you can have a direct binding between resources which having a _one-to-one_ or _one-to-many_ relationship.
    ```hcl
    output "windows_vm_list" {
        value = { for o in azurerm_windows_virtual_machine.windows_vm : o.name => {
            name : o.name,
            id: o.id,
            nic : { 
                name: azurerm_network_interface.interface[o.name].name,
                id: azurerm_network_interface.interface[o.name].id
            }
        }}
    }
    ```

### Locals
- Locals are using snake_case syntax.
- Build up tags as locals, and use these in your code instead of variables. If you tag each level of modules properly each created resource will inherit the tag of all the modules used in it's creation, just like a block-chain. It is fantastic as you can write exclusion policies for certain security use cases based on the resource tags. For example you do not enforce https_only on an `azurerm_storage_account`, which you use as an NFS file share, but everywhere else you can prevent deployments with a deny policy.
    ```hcl
    #locals.tf

    locals {
    #Tags
    tags = merge({
            creation_mode                                = "terraform",
            terraform-azurerm-easy-container-aks-cluster = "true"
           }, var.tags)
    }
    ```

### Providers
- Do not set providers inside a module, as that results in an error, when you remove module call from deployments. An example for exception could be, when you need to enforce certain deployments (Kured, Wiz Connector etc.) with a cluster deployment, than you can add a kubernetes or helm provider called deployment to your aks-cluster module. The main rule that it needs to share the lifecycle of the underlying resource.
- Set the minimum required terraform version. Currently we are using 1.3.0 across all modules, as that both supports [Custom Condition Checks](https://www.terraform.io/language/expressions/custom-conditions#preconditions-and-postconditions) and [Optional Object Type Attributes](https://www.terraform.io/language/expressions/type-constraints#optional-object-type-attributes) functionality.
- Set the minimum required provider source and version, in a terraform.tf file. Each resource should be set to the latest [azurerm provider version](https://github.com/hashicorp/terraform-provider-azurerm/blob/main/CHANGELOG.md), where there is an update or bugfix to the resource.
- If it is a Wrapper, then this will be inherited from the Bricks, and the highest common union will be required. If you set there you are breaking that inheritance chain, so don't do it.
    ```hcl
    #terraform.tf

    terraform {

      required_version = ">=1.1.0"

      required_providers {

          azurerm = {
            source = hashicorp/azurerm
            version = ">= 3.14.0"
          }

          azapi = {
            source = azure/azapi
            version = ">=0.5.0"
          }

          kubernetes = {
            source = hashicorp/kubernetes
            version = ">=2.11.0"
          }
      }
    }
    ```

### Lifecycle Block
- Typically you want to add ignore_changes block for every object property, which can be modified outside of terraform by Cloud Provider or automation, such as policies.
- You need to be very careful tough, as this could break the functionality to interact with resource properties via the modules through the lifecycle of the deployment. Unfortunately there is no way (yet), to dynamically set the lifecycle block.
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

### Timeouts
- Overriding timeouts can help us fail fast. While the default timeouts are very forgiving in both terraform and azurerm api, we do not want an AKS deployment to timeout after 90 minutes, when the average deployment time is 5-8 minutes. So we override that to 30 minutes. This needs to be explored independently for every resource, where we introduce it. As a rule we want to use it for resources, where creation time average is more than 5 minutes or failure rate justifies it.
    ```hcl
    #variables-aks_cluster.tf

    #### TimeOut ####
    variable "aks_cluster_timeout_create" {
        type        = string
        default     = "30m"
        description = "Specify timeout for create action. Defaults to 30 minutes."
    }
    variable "aks_cluster_timeout_update" {
        type        = string
        default     = "30m"
        description = "Specify timeout for update action. Defaults to 30 minutes."
    }
    variable "aks_cluster_timeout_read" {
        type        = string
        default     = "5m"
        description = "Specify timeout for read action. Defaults to 5 minutes."
    }
    variable "aks_cluster_timeout_delete" {
        type        = string
        default     = "30m"
        description = "Specify timeout for delete action. Defaults to 30 minutes."
    }

    #main.tf

    resource "azurerm_kubernetes_cluster" "aks_cluster" {
    timeouts {
        create = var.aks_cluster_timeout_create
        update = var.aks_cluster_timeout_update
        read   = var.aks_cluster_timeout_read
        delete = var.aks_cluster_timeout_delete
    }
    ```
- With this we can limit failure times, but also allow people to raise or lower it, if they feel they need to.

### Versioning
- Modules are versioned with git release tags.
- Use semantic versioning. Syntax is major_version.minor_version.patch_version, eg. 1.0.0
- patch_version needs to be raised, whenever we fixing some issues or adjusting terraform code to mirror latest provider changes
- minor_version needs to be raised, whenever we are implementing a new feature. We should support preview features as well, so teams can opt-in.
- major_version needs to be raised, whenever we are introducing a breaking change, which requires rebuild of a given resource
- Due to the limitation that module sourcing can't use variables, Wrappers needs to use the same logic. When we version any of the Bricks, the Wrappers consuming it should be updated as well.