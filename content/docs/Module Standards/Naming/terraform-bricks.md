---
categories: ["Modules", "Standards", "Brick"]
tags: ["docs","standards","naming","terraform"]
title: "Terraform Bricks"
linkTitle: "Terraform Bricks"
weight: 11
date: 2012-09-24
description: Terraform sub module naming standard
---
<hr>

### Terraform Sub-Modules
- The second word is the provider the majority of the code is written in, eg. _azurerm_, _azuread_, _kubernetes_, _helm_ etc.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- The fourth word is the category the resource under on the [Terraform Registry](https://registry.terraform.io/) documentation, eg. _compute_, _container_, _storage_ etc. If that is more than 2 word eg. _Logic App_, the do not use any delimiters, write in a single word, eg. _logicapp_.
- Any subsequent words should clearly state, what that module is used for eg., if it's deploying a Linux VM, then it should be linux-vm.

Example: `terraform-azurerm-easy-compute-linux-vm`