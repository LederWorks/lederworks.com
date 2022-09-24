---
categories: ["Modules", "Standards","Wrapper"]
tags: ["docs","standards","naming",terraform"]
title: "Terraform Wrappers"
linkTitle: "Terraform Wrappers"
weight: 12
date: 2012-09-24
description: Terraform wrapper module naming standard
---
<hr>

### Terraform Wrapper Modules
- The second word is the provider the majority of the code is written in, eg. _azurerm_, _azuread_, _kubernetes_, _helm_ etc.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The fourth word is _wrapper_.
- Any subsequent words should clearly state, what that module is used for and delimited by _. If it is deploying an AKS Cluster, then it should be aks-cluster.

Example: `terraform-azurerm-easy-wrapper-aks-cluster`