---
title: Keystone Prefabs
---

# Keystone Prefabs

Keystone Prefabs adds prefabs to Unreal Engine 5.7 and 5.8 (Win64 editor): nested prefabs, per-instance overrides with Apply and Revert, prefab variants, ISM/HISM components inside prefabs, a Pack step that turns static meshes into instances, and a Bake step that exports to plain actors, Level Instances or Packed Level Actors.

Pages in this site:

- [Coming from Unity](coming_from_unity.md)
- [Coming from Prefabricator and Level Instances](coming_from_prefabricator_and_level_instances.md)
- [Source control behaviour](source_control.md)
- [Python and CI](python_and_ci.md)
- [Limitations](limitations.md)
- [FAQ](faq.md)

Workflow inspired by Unity prefabs and prefab variants.

---

## Contents

1. [Install](#1-install)
2. [Create a prefab from the selection](#2-create-a-prefab-from-the-selection)
3. [Place a prefab](#3-place-a-prefab)
4. [Edit, Commit, Discard](#4-edit-commit-discard)
5. [Overrides](#5-overrides)
6. [Remove Child: hide or delete](#6-remove-child-hide-or-delete)
7. [Variants](#7-variants)
8. [Nested prefabs and cycle detection](#8-nested-prefabs-and-cycle-detection)
9. [ISM and HISM inside prefabs](#9-ism-and-hism-inside-prefabs)
10. [Pack and Unpack](#10-pack-and-unpack)
11. [Bake to native](#11-bake-to-native)
12. [Spawn at runtime](#12-spawn-at-runtime)
13. [Health report](#13-health-report)
14. [Updating every instance](#14-updating-every-instance)

---

## 1. Install

1. Install *Keystone Prefabs* from Fab through the Epic Games Launcher. Fab installs it as an engine plugin (`Engine/Plugins/Fab/`) for the engine version you pick. Alternatively, for source use, copy the `KeystonePrefabs` folder into your project's `Plugins/` folder and build the project.
2. Open your project and enable *Keystone Prefabs* under `Edit > Plugins` (category *Editor*), then restart the editor.
3. Optional: the Python API and the bundled CI tests need the engine's *Python Editor Script Plugin* (`PythonScriptPlugin`, a Beta plugin). Keystone Prefabs itself does not depend on it.

Target engines: UE 5.7 and 5.8, Win64 editor. This build was compiled and tested against UE 5.7.4 and 5.8.4. All modules are restricted to Win64 in the `.uplugin` (`PlatformAllowList`), including the runtime module, because Win64 is the only tested platform. See [Limitations](limitations.md).

The plugin ships a demo map, `/KeystonePrefabs/Demo/Maps/KeystonePrefabs_Demo` (Content Browser: enable *Show Plugin Content*). It contains a prefab (`P_Gazebo`), a variant (`P_Gazebo_Square`), a nested prefab (`P_Plaza`) and a packed prefab (`P_Fence`), with per-instance overrides.

## 2. Create a prefab from the selection

1. Select one or more actors in the level.
2. Right-click, then **Keystone Prefab > Create Prefab**, and choose the **pivot**:
   - **Bounds Bottom Center**: center of the selection bounds, at the lowest Z.
   - **Bounds Center**: center of the selection bounds.
   - **First Selected Actor**: location of the first selected actor.
   - **World Origin**.
3. In the save dialog pick the asset path (the default is `/Game/Prefabs/P_<first actor label>`).

The selected actors are replaced by an instance of the new prefab, and the Content Browser jumps to the new asset.

How actors are turned into nodes:

- Actors that only need components (static meshes, ISM/HISM, lights, decals, audio) become **component nodes**. They live as components on one `Keystone Prefab` actor per instance.
- Blueprint and gameplay actors become **actor nodes**. They are real actors attached to the instance actor, tagged with a `KPrefabNodeComponent` that stores the owner and the node GUID.
- Prefab instances inside the selection become **nested prefabs**.

You can also create an empty prefab from the Content Browser's Add menu (*Basic > Keystone > Keystone Prefab*). The same creation operation is available from Python as `create_prefab_from_actors` (see [Python and CI](python_and_ci.md)).

## 3. Place a prefab

Drag a prefab asset from the Content Browser into the viewport. Each placed instance is an `AKPrefabActor` ("Keystone Prefab") that references the asset and builds its nodes from it. The **Prefab** column in the World Outliner shows each instance's state: up to date, has overrides, out of date, or being edited.

You do not edit an instance's children to change the prefab. Use [Edit](#4-edit-commit-discard) to change the prefab, and [overrides](#5-overrides) to change one instance.

## 4. Edit, Commit, Discard

Edit-in-place works like Level Instance editing.

- **Edit**: select an instance and click **Edit** in the *Prefab* category of the Details panel, or right-click and choose **Keystone Prefab > Edit Prefab**. The instance's overrides are suspended while you edit, and everything you change targets the prefab asset. The **Keystone Prefab** editor mode opens a toolbar overlay ("Editing prefab: ...") with **Commit** and **Discard** buttons.
- **Commit** (overlay button, Details button, or **Commit Prefab Edit** in the context menu): writes the edited instance back to the prefab asset, updates every loaded instance in one undoable transaction, and leaves edit mode.
- **Discard**: throws the edits away and leaves edit mode.

Each of these is one undoable transaction.

**Nested prefabs**: select a node that belongs to a nested prefab and choose **Keystone Prefab > Edit Nested Prefab**. A temporary instance of the nested prefab opens at its position inside the outer instance, and the matching subtree is hidden until you commit or discard.

## 5. Overrides

Changing a property on one instance (a component property, an actor-node property, or a node's transform) creates an override on that instance. Overrides are tracked automatically; there is nothing to enable.

- The override is stored only where the instance value differs from the resolved node. Resolution order: class default, base prefab node, variant deltas in chain order, instance overrides.
- Overrides are matched by the node's stable GUID, never by name, label or index. Renaming an actor or reordering components does not move overrides to the wrong node.
- The **root transform of the instance is not an override**. Moving the instance in the level is just moving the instance.

### Details-panel indicators and the row context menu

- A property that is overridden on an instance's component or actor node gets two small buttons at the end of its Details row: **Revert to Prefab** (the "differs from default" icon) and **Apply to Prefab** (a save icon). They appear only while the property is overridden.
- Right-click the row: **Apply to &lt;prefab name&gt;** (one entry per available target) and **Revert to Prefab**.
- The *Prefab* category of the instance lists every override with a per-entry **Revert** and **Apply** button. Stale entries are shown in a warning colour.
- The same category has **Apply All** (with a dropdown to pick the target), **Revert All**, **Remove Stale Overrides** and **Update**.
- The actor context menu has **Apply All Overrides**, **Revert All Overrides**, **Update Instance** and **Remove Stale Overrides** for the selected instances.

### Apply target

For a node that comes from a variant parent or a nested prefab, the override can be applied to more than one asset. The menus list the targets, outermost first: the instance's prefab, its variant parents, and the nested prefabs the node comes from. Applying to a variant parent changes the parent and therefore every variant and instance below it. *Apply All* applies every override that can target the chosen asset; overrides that cannot target it are kept.

### Revert All

**Revert All** removes every override on the instance.

## 6. Remove Child: hide or delete

Select a component or actor node of an instance, then right-click and choose **Keystone Prefab > Remove Child**:

- **Hide**: the node stays built but is invisible in the editor and in game (component nodes: *Visible* off and *Hidden in Game* on; actor nodes: *Actor Hidden In Game* on and temporarily hidden in the editor). Collision settings are not changed.
- **Delete**: the node and its children are not built on this instance.
- **Restore**: shows the node again.

The prefab asset is not changed. The removal is stored as an override on the instance and can be reverted like any other override.

## 7. Variants

A variant is a prefab whose parent is another prefab. It stores only deltas, plus nodes it adds.

1. Right-click a prefab in the Content Browser and choose **Keystone Prefab > Create Variant**. The default name is `<parent>_Variant`.
2. Edit the variant like any prefab. Only differences from the parent are saved.
3. Variants of variants are allowed (base, variant, variant...).

Changes to the base prefab flow into every variant and instance. Overrides can be applied to the variant itself or to its parent (see [Apply target](#apply-target)). `UKPrefabLibrary::IsPrefabVariant` and `GetVariantParent` expose this to Blueprint and C++. The Details status line of an instance reads, for example, "Up to date - Variant of P_House".

## 8. Nested prefabs and cycle detection

A prefab can contain other prefab instances. When you create a prefab from a selection that includes prefab instances, those become nested prefab nodes. A nested prefab is flattened into its owner when resolved; the nested node becomes a pivot scene component on the instance.

Cycles are refused: a prefab cannot, directly or through variant parents and nested prefabs, contain itself. The operation fails with a message such as "Can't nest 'A' inside 'B': it would create a prefab cycle."

You can override properties of nodes that come from a nested prefab. The override is stored on the outer instance, or applied into the nested prefab when you choose that as the [apply target](#apply-target).

## 9. ISM and HISM inside prefabs

`UInstancedStaticMeshComponent` and `UHierarchicalInstancedStaticMeshComponent` can be nodes in a prefab. Their instance transforms and custom data are stored with the node (the virtual property `@Instances`) and can be overridden per instance like any other property.

## 10. Pack and Unpack

**Pack** converts groups of identical static mesh nodes in a prefab into ISM or HISM components. Right-click a prefab in the Content Browser and choose **Keystone Prefab > Pack**.

- Meshes that share mesh, materials and collision setup are packed when at least *Min Instances* of them exist (default 2).
- A group becomes an HISM when it has at least *HISM Threshold* instances (default 64; 0 means always ISM).
- Before packing, a dialog shows the **draw-call estimate**: estimated draw calls before and after, the number of static mesh nodes packed, and the number of instanced components created. Click *Yes* to pack.
- The menu uses the default settings above. To use other values, call `estimate_pack` / `pack_prefab` from Python with a `KPackSettings`.

**Unpack** (same menu) restores the original nodes from a record stored in the prefab asset. A prefab that is already packed must be unpacked before it can be packed again.

Pack and Unpack are undoable and propagate to loaded instances like a commit.

## 11. Bake to native

Bake turns a prefab instance into engine-native content, so you are never locked in. It is one-way: the result is not linked to the prefab afterwards. Right-click an instance and choose **Keystone Prefab > Bake**:

- **Actors**: replaces the instance with plain actors.
- **Level Instance...**: asks for a level asset path (default `/Game/Prefabs/Baked/LI_<prefab name>`), writes the level, and replaces the instance with a Level Instance that references it. The level is created without dialogs.
- **Packed Level Actor...**: same, but creates a Packed Level Actor. Packed Level Actors combine static meshes only; see the engine's [Level Instancing documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/level-instancing-in-unreal-engine).

## 12. Spawn at runtime

Prefabs and variants can be spawned in game. At cook time each prefab is flattened into a binary `FKCookedPrefab` (editor-only data stripped); spawning in a game world builds the instance from that data and never parses text.

Blueprint: call **Spawn Prefab** (category *Keystone > Prefab*) with a world context, the prefab asset, a transform, and *Movable* if the spawned instance has to move at runtime.

C++:

```cpp
#include "KPrefabLibrary.h"

AKPrefabActor* Instance = UKPrefabLibrary::SpawnPrefab(this, MyPrefab, SpawnTransform, /*bMovable=*/ false);
UObject* Door = UKPrefabLibrary::GetPrefabNodeByName(Instance, TEXT("Door"));
```

Other helpers in `UKPrefabLibrary`:

| Function | Purpose |
|---|---|
| `SpawnPrefab` | Spawns a prefab or variant. Game worlds use cooked data; editor worlds build a normal editable instance. |
| `GetPrefabNodeByName` | The component or actor built for the first node with that name. |
| `GetOwningPrefabActor` | The instance that owns an actor node or a component node (null if none). |
| `IsPrefabVariant` | True if the asset is a variant. |
| `GetVariantParent` | The variant parent (null for base prefabs). |
| `GetCookedNodeCount` | Number of nodes in the flattened cooked prefab. |

Prefab structure cannot be edited or replicated at runtime; see [Limitations](limitations.md).

## 13. Health report

**Tools > Keystone Prefabs > Prefab Health Report** builds a report for the current world and shows it on the *Prefab Health* page of the **Keystone Prefabs** message log. It checks loaded instances, unloaded World Partition instances (through actor descriptors, without loading them), and every prefab asset. Issue types:

| Issue | Meaning |
|---|---|
| Missing Prefab | An instance refers to a prefab asset that cannot be loaded. |
| Broken Reference | A node refers to an object that no longer exists. |
| Stale Override | An override targets a node or property that no longer exists in the prefab. |
| Out Of Date Instance | A loaded instance was built from an older version of its prefab. |
| Out Of Date Unloaded Instance | The same, for an unloaded World Partition instance. It re-syncs when its cell loads, or through the commandlet. |
| Orphaned Node Actor | A node actor whose owning instance is gone. |
| Resolve Error | The prefab cannot be resolved (for example a cycle or missing data). |

Stale overrides can be removed per instance with **Remove Stale Overrides**.

## 14. Updating every instance

- **Loaded instances**: a commit re-syncs them automatically. Out-of-date instances can also be refreshed with **Tools > Keystone Prefabs > Update All Prefab Instances**, with **Update Instances** in a prefab's Content Browser menu, or with **Update Instance** on selected instances.
- **Unloaded World Partition instances** are not touched by a commit. They re-sync when their cell loads in the editor, or in bulk with the `UpdatePrefabInstances` commandlet. The commandlet prints the list of files it is going to change before it loads or changes anything.

```
UnrealEditor-Cmd.exe "<path>/MyProject.uproject" -run=UpdatePrefabInstances -Map=/Game/Maps/Town -DryRun
UnrealEditor-Cmd.exe "<path>/MyProject.uproject" -run=UpdatePrefabInstances -AllMaps -CheckOut -BatchSize=50
```

All switches are listed in [Python and CI](python_and_ci.md#updateprefabinstances-commandlet). See [Source control behaviour](source_control.md) for exactly which files are touched.
