---
linkTitle: White Box
title: White Box Gem
description: The White Box Gem provides White Box rapid design components for Open 3D Engine (O3DE).
toc: true
---

The White Box Gem provides two tools implemented as components; the **White Box** component, and the **White Box Collision** component. You can use the White Box component to quickly sketch proxy geometry for your entities. Use the White Box Collision component with the White Box component to provide a collision surface for PhysX simulations.

## What is white boxing?

*White boxing* is a rapid, iterative design process. Simple modeling tools are used to create proxy geometry that defines the shape and dimension of 3D objects and environments. The proxy geometry created in this process has a very low polygon count, but reasonably approximates the size and shape of the final assets. White box entities have a basic material, and can have simple collision and physics materials if required. The entities created in the white box process can be handed off to artists to be developed into polished production entities.

In **Open 3D Engine (O3DE)**, you create entities with the White Box component, using its edit functionality to rapidly sketch 3D assets to serve as building blocks for a level. Because you create white box meshes in O3DE's **Editor** with the component entity system, the entities can also contain functionality from other components such as scripts. You can use a rapid build and test iteration cycle to create refined and functional entities before committing to developing production-ready art assets. The resulting white box geometry and entities are used as templates for production assets. This development process is fast, highly iterative, and cost-effective.

## White Box component information

For information on the White Box component, see [White Box component](/docs/user-guide/components/reference/shape/white-box).

For information on the White Box Collision component, see [White Box Collision component](/docs/user-guide/components/reference/shape/white-box-collider).
