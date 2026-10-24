---
title: FAQ
---

# FAQ

## Which engine versions and platforms are supported?

UE 5.7 and 5.8 with a Win64 editor; packaged games on Win64. Built and tested on UE 5.7.4 and 5.8.4. See [Limitations](limitations.md).

## Is a prefab a Blueprint?

No. A prefab is a `UKPrefabAsset` that stores a tree of nodes: components, actors and nested prefabs. An instance is an `AKPrefabActor` that builds those nodes. You cannot subclass a prefab, but its actor nodes can be Blueprint actors.

## Can I use prefabs at runtime?

You can spawn them. `UKPrefabLibrary::SpawnPrefab` (Blueprint *Spawn Prefab*) builds an instance from cooked data in game worlds. You cannot edit prefab structure at runtime, and prefab structure is not replicated. See [Limitations](limitations.md).

## What happens to my instances when I edit the prefab?

On **Commit**, every loaded instance re-syncs in the same undoable transaction, keeping its overrides. Instances in unloaded World Partition cells re-sync when their cell is loaded in the editor, or when you run the `UpdatePrefabInstances` commandlet. See [Source control behaviour](source_control.md).

## Why did my override disappear / show as stale?

Overrides are matched to nodes by stable GUID. If the node or the property was deleted from the prefab, the override is *stale*. The Health report lists them, and *Remove Stale Overrides* deletes them. If you only renamed a node, the override is kept.

## Why can't I override this property?

Not overridable: properties that are transient, edit-const, deprecated, not editable on instances, delegates, static arrays (`ArrayDim > 1`), and properties holding instanced subobjects. Also not overridable are components created by construction scripts. Object references to actors outside the prefab are not stored. See [Limitations](limitations.md).

## Why is the instance's root transform not an override?

Same as in Unity: moving the instance in the level just moves the instance. The transforms of nodes inside the instance can be overridden.

## Does Pack change my prefab permanently?

Pack rewrites the prefab's nodes into ISM/HISM components and stores a record of the original nodes. **Unpack** restores them. Both are undoable.

## Can I get out of Keystone Prefabs?

Yes. *Bake to Actors* replaces an instance with plain actors, and *Bake to Level Instance* / *Bake to Packed Level Actor* write a level and place a Level Instance or PLA. Baking is one-way.

## Can I import Prefabricator prefabs or Level Instances?

Not in 1.0. Importers are planned for 1.1. See [Coming from Prefabricator and Level Instances](coming_from_prefabricator_and_level_instances.md).

## Can I make a prefab contain itself?

No. Cycles through nesting or variant parents are detected and the operation fails with an error.

## Where does the Health report go?

To the message log named **Keystone Prefabs** (`Window > Developer Tools > Message Log`).

## Can I automate this?

Yes: Python (`unreal.KPrefabEditorLibrary`, `unreal.KPrefabLibrary`) and the `UpdatePrefabInstances` commandlet. See [Python and CI](python_and_ci.md).
