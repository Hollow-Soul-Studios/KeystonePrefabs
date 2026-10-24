---
title: Coming from Prefabricator and Level Instances
---

# Coming from Prefabricator and Level Instances

## Importer status

**There is no importer in 1.0.** A Prefabricator importer and a Level Instance importer are on the 1.1 roadmap; the Level Instance importer depends on what is technically possible. Until then, rebuild prefabs by selecting the actors of an existing prefab or Level Instance in the level and using *Create Prefab* on the selection.

## Differences to Level Instances

| | Level Instances (LI) / Packed Level Actors (PLA) | Keystone Prefabs |
|---|---|---|
| Storage | A separate level asset per LI | A prefab asset (`UKPrefabAsset`); no extra level |
| Content | LI: any actor. PLA: static meshes only | Component nodes (meshes, ISM/HISM, lights, decals, audio) and actor nodes (Blueprint and gameplay actors) |
| Variants | None | Variants store deltas on a parent |
| Overrides | Engine property overrides | Per-property Apply, Revert and Revert All, Details indicators, Remove Child |
| Files per instance | One external actor per LI actor | Component nodes: one external actor file per instance. Actor nodes: one file per node actor |
| Instancing | PLAs combine meshes, LIs do not | *Pack* converts mesh nodes to ISM/HISM, with a draw-call estimate and Unpack |
| Level editing | Edit the level | *Edit* in place on the instance |

Keystone does not replace Level Instances: *Bake to Level Instance* and *Bake to Packed Level Actor* export a prefab instance to them.

## Differences to Prefabricator

- Variants, per-property overrides with Apply/Revert, and ISM/HISM nodes are part of the prefab model.
- Instances are matched by GUID to the prefab's nodes, and instance overrides are stored on the instance.
- Cycles (a prefab inside itself) are detected and refused.
- Unloaded World Partition instances are found through actor descriptors and re-synced lazily; see [Source control behaviour](source_control.md).
- There is no randomized collection or PCG node in 1.0.
- Landscape inside prefabs is not supported ([Limitations](limitations.md)).

## Migrating by baking

You are never locked into Keystone. *Bake to Actors* replaces an instance with plain actors; *Bake to Level Instance* / *Bake to Packed Level Actor* write a level and place a Level Instance or PLA. Baking is one-way: the result is not linked back to the prefab.

## Inspiration and credits

- Workflow inspired by Unity prefabs and prefab variants (see [Coming from Unity](coming_from_unity.md)).
- The idea of "Save branch as scene" in Godot is the model for creating a prefab from a selected group of actors; Godot also shows a revert control per overridden property.
- RecourseDesign's performance posts in the [Level Instances / Packed Level Instance Actor forum thread](https://forums.unrealengine.com/t/level-instances-packed-level-instance-actor-need-serious-work/1433274) are prior art on Level Instance versus instancing performance, and a baseline for judging the Pack draw-call numbers.
- Prefabricator (MIT, Code Respawn) is the established free tool in this space. Keystone Prefabs is a clean-room design. When the Prefabricator importer ships, the notices required by Prefabricator's MIT licence will ship with it.
