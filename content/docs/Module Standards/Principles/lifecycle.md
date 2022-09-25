---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"]
title: "Terraform Lifecycle"
linkTitle: "Lifecycle"
weight: 15
date: 2012-09-24
description: Module lifecycle principles
---
<hr>

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