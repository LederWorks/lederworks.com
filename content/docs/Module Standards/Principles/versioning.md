---
categories: ["Modules", "Standards"]
tags: ["docs","standards","principles"]
title: "Versioning"
linkTitle: "Versioning"
weight: 17
date: 2012-09-24
description: Module versioning principles
---
<hr>

### Versioning
- Modules are versioned with git release tags.
- Use semantic versioning. Syntax is major_version.minor_version.patch_version, eg. 1.0.0
- patch_version needs to be raised, whenever we fixing some issues or adjusting terraform code to mirror latest provider changes
- minor_version needs to be raised, whenever we are implementing a new feature. We should support preview features as well, so teams can opt-in.
- major_version needs to be raised, whenever we are introducing a breaking change, which requires rebuild of a given resource
- Due to the limitation that module sourcing can't use variables, Wrappers needs to use the same logic. When we version any of the Bricks, the Wrappers consuming it should be updated as well.