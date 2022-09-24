---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"]
title: "Input Parameters"
linkTitle: "Inputs"
weight: 11
date: 2012-09-24
description: Module input parameter principles
---
<hr>

## BLA

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
- In sub-modules always input object properties and not whole objects, as that makes for_each loops much simpler in HCL.
- In wrapper-modules, you can input a whole object (preferrably outputted from one of the sub-modules used) and use the below technique to grab only what you need from it. There is a limitation, if the object output contains sensitive data, all output properties will inherit it too, even if it's not sensitive, and you can't use it for count and for_each operations in calling wrapper-modules. In this case you also need to output the required object properties as well.

### BLABALA
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
## BALABA
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