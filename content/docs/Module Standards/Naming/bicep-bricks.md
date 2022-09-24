---
categories: ["Modules", "Standards", "Brick"]
tags: ["docs","standards","naming","bicep"]
title: "Bicep Bricks"
linkTitle: "Bicep Bricks"
weight: 13
date: 2012-09-24
description: Bicep sub module naming standard
---
<hr>

### Bicep Sub Modules
- The second word is the second part of the [ARM API Documentations](https://docs.microsoft.com/en-us/azure/templates/) resource, eg. for Microsoft.Network/ it is going to be _network_.
- The third word is _easy_. If you implementing these standards somewhere else, this most possible will be yout company, organization etc. name.
- Any subsequent words should clearly state, what that module is used for. If it is deploying a DataBricks workspace it is going to be _databricks-workspace_.

Example: `bicep-managedidentity-easy-user-managed-identity`