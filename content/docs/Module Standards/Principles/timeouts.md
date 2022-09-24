---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"]
title: "Timeouts"
linkTitle: "Timeouts"
weight: 16
date: 2012-09-24
description: Module timeout principles
---
<hr>

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