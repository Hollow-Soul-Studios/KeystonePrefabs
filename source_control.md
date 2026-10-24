---
title: Source control behaviour
---

# Source control behaviour

This page lists which files each operation touches, so you can plan check-outs under Perforce or Git LFS. World Partition maps use One File Per Actor (OFPA): each actor is saved in its own external actor file.

## Why few files change

A prefab instance is **one** `AKPrefabActor`.

- **Component nodes** (static meshes, ISM/HISM, lights, decals, audio) are components on that actor. One instance is one external actor file, however many component nodes the prefab has.
- **Actor nodes** (Blueprint and gameplay actors) are real actors attached to the instance. Each node actor has its own external actor file. A prefab with 3 actor nodes is 4 files per instance (instance plus 3 node actors).
- **Overrides live in the instance's own actor file** (the `Overrides` property of the `AKPrefabActor`). Overriding a property never touches the prefab asset. For an override on an actor node's component, the override is stored on the owning instance, not on the node actor.
- Prefab data is stored as text values and GUIDs, so assets and actor files diff readably.

## What each operation touches

| Operation | Files touched |
|---|---|
| **Edit** on an instance | None until saved. Edit mode is an editor state; the instance's overrides are suspended. |
| **Commit** (base prefab edit) | The prefab `.uasset`, plus the external actor files of the **loaded** instances that re-sync (and of their node actors, if the prefab has actor nodes). |
| **Discard** | None. |
| **Commit of a nested prefab** | The nested prefab `.uasset`, plus loaded instances of every prefab, variant or nesting prefab that depends on it. Outer prefab assets are not rewritten. |
| **Apply** override | The target prefab `.uasset` (the instance's prefab, a variant parent, or a nested prefab, whichever you chose) and the loaded instances that re-sync. The instance's own file also changes because the override is removed from it. |
| **Revert** / **Revert All** | The instance's actor file only. |
| **Remove Child** (Hide / Delete / restore) | The instance's actor file only. |
| **Create Prefab** from actors | The new prefab `.uasset`. If the source actors are replaced by an instance: the external actor files of the source actors are removed and one new instance file is added. |
| **Create Variant** | The new variant `.uasset`. The parent is not changed. |
| **Pack / Unpack** | The prefab `.uasset`, plus loaded instances that re-sync. |
| **Place** (drag-drop) | One new external actor file for the instance. |
| **Bake to Actors** | The instance file is removed; one new external file per created actor. |
| **Bake to Level Instance / Packed Level Actor** | The new level `.umap`, the Level Instance actor file, and the removal of the baked instance file. |
| **Health report** | None. Read-only. |

## Unloaded World Partition instances

Instances in cells that are not loaded are **not** modified by a commit, so a commit never has to check out the whole map.

- Each instance stores the hash of the prefab content it was last resolved against, and publishes it (with its prefab path) in its actor descriptor. The Health report uses this to list out-of-date unloaded instances **without loading them**. They show up as *Out Of Date Unloaded Instance*.
- When such a cell is loaded in the editor, the stale instance is **re-synced lazily**. Only then does its file change, and only because someone is working in that region.
- To update everything in one pass, run the `UpdatePrefabInstances` commandlet ([Python and CI](python_and_ci.md#updateprefabinstances-commandlet)). **It prints the list of files it is going to modify first**, so you can check them out beforehand or review them with `-DryRun`. With `-CheckOut` it checks them out through the configured source control provider, in batches (`-BatchSize`).

## Cook and packaged builds

The cooked, flattened prefab data is stored inside the prefab asset and regenerated when the asset is saved and at cook time. Cooking warns if an instance is out of date, so run the commandlet (or open and save the affected regions) before a release build.

## Recommendations

- Commit prefab assets and the map files that changed in the same change list.
- Treat the prefab `.uasset` as a normal binary asset: Perforce exclusive checkout or Git LFS locking applies.
- Prefer component nodes over actor nodes where a full actor is not needed; it keeps the per-instance file count at one.
