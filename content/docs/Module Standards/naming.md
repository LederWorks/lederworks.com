---
categories: ["Modules", "Standards"]
tags: ["docs","standards","naming", "terraform","bicep","golang"] 
title: "Module Naming"
linkTitle: "Naming"
weight: 21
description: Module naming standards
---
<hr>

## Naming
We are following the default [naming conventions](https://www.terraform.io/registry/modules/publish) enforced on the [Terraform Registry](https://registry.terraform.io/) for terraform modules.

Each Brick and Wrapper needs its own repository in the [LederWorks Organization](https://github.com/LederWorks) based on the following rules for Bricks:

`language-provider-easy-brick-category-purpose`

and for Wrappers:

`language-provider-easy-wrapper-purpose`


- Every word is delimited by hyphens.
- The first word of every module is the _language_ the majority of the code is written, eg. _terraform_, _bicep_, _ansible_, _golang_ etc.
- The second word is either a terraform provider or a class used by the cloud API.
- The third word is _easy_, you can replace that with your organization name for example.
- The fourth word is either the _brick_ or _wrapper_, depends on the type of the module.
- The fifth word for Bricks is the category under the provider structure.
- The sixth (or fifth for Wreappers) and consequent words are the _purpose_, which needs to be unique within the organization. We are suffixing the input parameters and variables with this using snake_case, eg. for:
`terraform-azurerm-easy-container-aks-cluster` all variables will be prefixed with `aks_cluster`
<hr>

### Terraform Bricks

`terraform-azurerm-easy-brick-compute-linux-vm`

- The second word is the provider the majority of the code is written in, eg. _azurerm_, _azuread_, _kubernetes_, _helm_ etc.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The fourth word is either the _brick_ or _wrapper_, depends on the type of the module.
- The fifth word is the category the resource under on the [Terraform Registry](https://registry.terraform.io/) documentation, eg. _compute_, _container_, _storage_ etc. If that is more than 2 word eg. _Logic App_, the do not use any delimiters, write in a single word, eg. _logicapp_.
- Any subsequent words should clearly state, what that module is used for eg., if it's deploying a Linux VM, then it should be linux-vm.
<hr>

### Terraform Wrappers

`terraform-azurerm-easy-wrapper-aks-cluster`

- The second word is the provider the majority of the code is written in, eg. _azurerm_, _azuread_, _kubernetes_, _helm_ etc.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The fourth word is _wrapper_.
- Any subsequent words should clearly state, what that module is used for and delimited by _. If it is deploying an AKS Cluster, then it should be aks-cluster.
<hr>

 ### Bicep Bricks

 `bicep-managedidentity-easy-user-managed-identity`

- The second word is the second part of the [ARM API Documentations](https://docs.microsoft.com/en-us/azure/templates/) resource, eg. for Microsoft.Network/ it is going to be _network_.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- Any subsequent words should clearly state, what that module is used for. If it is deploying a DataBricks workspace it is going to be _databricks-workspace_.
<hr>

 ### Bicep Wrappers

 `bicep-easy-wrapper-databricks-isolated-vnet`

- The second word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The third word is _wrapper_.
- Any subsequent words should clearly state, what that module is used for. If it is deploying a DataBricks deployment in an island vnet, then it is going to be _databricks-isolated-vnet_.
<hr>

### GoLang Modules

`golang-easy-terratests`

As Go Modules in general importing several packages, I do not think that we need to include this information in the name of the respective repository.
- The second word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- Any subsequent words should clearly state, what that module is used for. If it is used for terratests, then it should be _terratests_.