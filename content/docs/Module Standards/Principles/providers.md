---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"]
title: "Terraform Providers"
linkTitle: "Providers"
weight: 14
date: 2012-09-24
description: Module provider principles
---
<hr>

### Providers
- Do not set providers inside a module, as that results in an error, when you remove module call from deployments. An example for exception could be, when you need to enforce certain deployments (Kured, Wiz Connector etc.) with a cluster deployment, than you can add a kubernetes or helm provider called deployment to your aks-cluster module. The main rule that it needs to share the lifecycle of the underlying resource.
- Set the minimum required terraform version. Currently we are using 1.1.0 across all modules, but might be bumped to 1.2.0 to leverage Output [Custom Condition Checks](https://www.terraform.io/language/expressions/custom-conditions#preconditions-and-postconditions) functionality.
- Set the minimum required provider source and version, in a terraform.tf file. Each resource should be set to the latest [azurerm provider version](https://github.com/hashicorp/terraform-provider-azurerm/blob/main/CHANGELOG.md), where there is an update or bugfix to the resource.
- Set experiments block if it is required.
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

      experiments = [module_variable_optional_attrs]
    }
    ```