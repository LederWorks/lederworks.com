---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"]
title: "Output Parameters"
linkTitle: "Outputs"
weight: 12
date: 2012-09-24
description: Module output parameters principles
---
<hr>

### Outputs
- Always use sensitive=true if you need to. Terraform 0.15.x and above will fail your plan, but on lower versions sensitive data is outputted in clear text.
- Output complete objects, and it's properties only if it is _sensitive_ and going to be called with a terraform function on the wrapper level. It is limiting the number of objects in the statefile, also can be better consumed in wrapper-modules.
    ```hcl
    #outputs.tf

    output "aks" {
        value     = azurerm_kubernetes_cluster.aks_cluster
        sensitive = true
    }

    output "udr" {
        value = azurerm_route_table.udr
    }
    ```
- When you are working with resource types, which having direct dependency on each other - for example a Virtual Machine and it's NIC, create and output complex maps. This way later you can have a direct binding between resources which having a _one-to-one_ or _one-to-many_ relationship.
    ```hcl
    output "windows_vm_list" {
        value = { for o in azurerm_windows_virtual_machine.windows_vm : o.name => {
            name : o.name,
            id: o.id,
            nic : { 
                name: azurerm_network_interface.interface[o.name].name,
                id: azurerm_network_interface.interface[o.name].id
            }
        }}
    }
    ```