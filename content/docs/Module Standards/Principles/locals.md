---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"]
title: "Terraform Locals"
linkTitle: "Locals"
weight: 13
date: 2012-09-24
description: Module locals principles
---
<hr>

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