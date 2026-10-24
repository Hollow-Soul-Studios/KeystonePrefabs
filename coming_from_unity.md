---
title: Coming from Unity
---

# Coming from Unity

Workflow inspired by Unity prefabs and prefab variants. Keystone Prefabs is not affiliated with or endorsed by Unity Technologies; "Unity" is used only to name the workflow this plugin's concepts map to. No Unity code, icons or logos are used.

## Concept mapping

| Unity | Keystone Prefabs | Notes |
|---|---|---|
| Prefab asset | `UKPrefabAsset` (a `.uasset` in the Content Browser) | Created from selected actors. Nodes are stored with stable GUIDs and text property values. |
| Prefab instance in a scene | `AKPrefabActor` ("Keystone Prefab") | Component nodes become components on this one actor. Blueprint and gameplay actors become attached actors tagged with `KPrefabNodeComponent`. |
| Open Prefab (prefab mode) | Edit / Commit / Discard on an instance | Edit happens in the level, like Level Instance editing. Overrides on that instance are suspended while editing. |
| Instance overrides | Per-instance overrides | Property, node transform, and "Remove Child" overrides. Stored on the instance (in its actor file). |
| Override indicators (blue bar, +/-) | Revert and Apply buttons on overridden Details rows; override list and status in the instance's *Prefab* category; state icon in the Outliner's *Prefab* column | There is no blue bar: an overridden row shows the buttons only while it is overridden. |
| Apply / Revert for one override | *Apply to &lt;prefab&gt;* / *Revert to Prefab* in the row's context menu or row buttons | |
| Apply All / Revert All | *Apply All* (with target dropdown) / *Revert All* in the Details *Prefab* category, or in the actor context menu | Revert All removes every override. Apply All keeps overrides that cannot target the chosen asset. |
| Apply target choice (Prefab Variant parent, nested prefab) | Apply target list, outermost first | Targets are the instance's prefab, its variant parents, and nested prefabs the node comes from. |
| Prefab Variant | Variant (`UKPrefabAsset` with a `ParentPrefab`) | Stores only deltas and added nodes. Variants of variants are supported. |
| Nested prefab | Prefab node (`EKPrefabNodeKind::Prefab`) | Flattened into the owner when resolved. Cycles are rejected. |
| Remove Component / Remove GameObject override | *Remove Child*: Hide or Delete | Hide builds the node but makes it invisible (collision is unchanged). Delete does not build it. |
| Unpack Prefab | *Bake to Actors* | One-way. The result is plain actors. |
| Root transform is not overridden | Same | The instance's root transform is just the instance's transform; it is never stored as an override. |
| Collections matched by index | Nodes matched by stable GUID | An override follows its node when you rename or reorder. Labels and indices are never used for matching. |
| Instantiate at runtime | `UKPrefabLibrary::SpawnPrefab` / Blueprint *Spawn Prefab* | Game worlds build from cooked binary data. |

## Differences to expect

- **A prefab is not a Blueprint class.** There is no class to subclass. Behaviour comes from the actors and components in the prefab (which can themselves be Blueprint classes).
- **Overrides on components created by construction scripts are not supported**, because those components have no stable identity. Components that are default subobjects or Blueprint (SCS) components are addressed by name and are overridable. See [Limitations](limitations.md).
- **Add Child overrides are not in 1.0.** You can hide or delete a node per instance; adding a new node to just one instance is planned for 1.1.
- **Unreal worlds use World Partition.** An instance that is in an unloaded cell is not updated immediately when the prefab changes; see [Source control behaviour](source_control.md).
- **Overrides are text.** Values are stored with `ExportText`, so prefab and actor files diff readably in source control.
