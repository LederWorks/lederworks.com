---
categories: ["Modules", "Standards"]
tags: ["docs","standards","naming"] 
title: "Module Naming"
linkTitle: "Naming"
weight: 1
description: >
  Module naming standards
---
<hr>

## Naming
We are following the default [naming conventions](https://www.terraform.io/registry/modules/publish) enforced on the [Terraform Registry](https://registry.terraform.io/) for terraform modules.

Each Brick and Wrapper needs its own repository in the [LederWorks Organization](https://github.com/LederWorks) based on the following rules:

`language-provider-easy-category-purpose`

- Every word is delimited by hyphens.
- The first word of every module is the _language_ the majority of the code is written, eg. _terraform_, _bicep_, _ansible_, _golang_ etc.
- The second word is either a terraform provider or an API class.
- The third word is _easy_, you can replace that with your organization name for example.
- The fourth word is either the _category_ of the deployment type or _wrapper_.
- The fifth and consequent words are the _purpose_, which needs to be unique within the organization. We are suffixing the input parameters and variables with this using snake_case, eg. for:
`terraform-azurerm-easy-container-aks-cluster` all variables will be prefixed with `aks_cluster`