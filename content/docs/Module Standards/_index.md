---
categories: ["Modules", "Standards"]
tags: ["docs","standards"] 
title: "Module Standards"
linkTitle: "Module Standards"
weight: 20
description: Infrastructure as Code module standards
---
<hr>

## Background
The driver behind the changes are the followings:
- The increasing complexity of the IaC modules are making updating and testing the code more and more difficult and time consuming.
- There are need for common standards how these modules are developed, in order to make integration of these parts into each other seamless and _EASY_.
<hr>

## Goals
- Develop small sub-modules called Bricks and build up complex deployments with Wrapper modules.
- Develop reusable terratests, as a go module, based on [Gruntworks Terratest suite](https://terratest.gruntwork.io/)
- Develop automated README.MD generation with [terraform-docs](https://terraform-docs.io/)
- Develop automated CHANGELOG.MD generation - tool under discussion.
- Follow DevSecOps best practices as a default deployment model, so consumers having _secure by default_ deployments, which meets the Legal Requirements and Industry Best Practices.