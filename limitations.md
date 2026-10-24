---
title: Limitations
---

# Limitations

Known limits of Keystone Prefabs 1.0.0.

## Content

- **No Landscape inside prefabs.** Landscape actors are not supported as prefab content.
- **Components created by construction scripts cannot be overridden.** They have no stable identity between runs. Components that are default subobjects or Blueprint (SCS) components are addressed by name and can be overridden; overrides are re-applied after construction scripts rerun.
- **Static arrays and instanced-subobject properties are not overridable.** Properties with `ArrayDim > 1` and properties holding instanced subobjects (`Instanced` / `EditInline` objects) are skipped by the override system.
- **The Instancing Random Seed of instanced mesh components is not stored or overridden.** The engine picks a new seed for each component whose seed is 0, so every instance gets its own; tracking it would mark every packed instance as overridden.
- **Object references from prefab nodes to level actors outside the prefab are not stored.** References between nodes of the same prefab are stored as node GUIDs and re-mapped on every build; a reference to an actor elsewhere in the level is not saved in the prefab.

## Features not in 1.0

- **"Add child" overrides** (adding a node to a single instance) are planned for 1.1. Per-instance Hide and Delete of existing nodes is supported.
- **Prefabricator and Level Instance importers** are planned for 1.1.
- **Runtime prefab editing and replication of prefab structure are not supported.** `SpawnPrefab` builds an instance from cooked data; the structure cannot be changed afterwards through the prefab API, and prefab structure is not replicated.
- Randomized collections, PCG spawn nodes, per-prefab Data Layers, and foliage capture are not in 1.0.

## Baking

- **Bake to Level Instance and Bake to Packed Level Actor are one-way.** The result is not connected to the prefab. Packed Level Actors combine static meshes only.
- Bake to Actors is also one-way.

## Engine and platform

- Supported engines: UE 5.7 and 5.8; built and tested on UE 5.7.4 and 5.8.4. Versions before 5.7 are not supported.
- Platforms: Win64 only. Every module, including the runtime module, has `"PlatformAllowList": ["Win64"]` because Win64 is the only tested platform. The runtime code has nothing platform-specific; other platforms can be enabled once they are tested.

## World Partition

- Unloaded instances are not updated when a prefab is committed. They re-sync when their cell loads in the editor, or with the `UpdatePrefabInstances` commandlet. See [Source control behaviour](source_control.md).
