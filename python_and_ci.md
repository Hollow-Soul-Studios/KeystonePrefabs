---
title: Python and CI
---

# Python and CI

Keystone Prefabs exposes two Blueprint function libraries to Unreal's Python API. To use them from Python, enable the engine's *Python Editor Script Plugin* (`PythonScriptPlugin`, Beta) in your project; the plugin itself does not depend on it. The CI tests below need it.

| Python class | C++ class | Module | Contents |
|---|---|---|---|
| `unreal.KPrefabLibrary` | `UKPrefabLibrary` | `KeystonePrefabs` (runtime) | `spawn_prefab`, `get_prefab_node_by_name`, `get_owning_prefab_actor`, `is_prefab_variant`, `get_variant_parent`, `get_cooked_node_count` |
| `unreal.KPrefabEditorLibrary` | `UKPrefabEditorLibrary` | `KeystonePrefabsEditor` (editor only) | Editor operations below |

Python names are the snake_case form of the C++ names. Use `help(unreal.KPrefabEditorLibrary)` in the editor's Python console for the exact signatures in your build. Output parameters follow Unreal's Python conventions: `create_prefab_from_actors` returns `(prefab, instance)`, `find_owning_instance` returns `(instance, node_id)`, and functions that return `bool` plus an output parameter return only the output, or `None` on failure (`pack_prefab` returns the `KDrawCallEstimate` or `None`). Struct members named `Property` are read with `get_editor_property("property")` because `property` is a Python builtin. Every mutating call is undoable; errors are logged to `LogKeystonePrefabs`.

## `unreal.KPrefabEditorLibrary`

| Group | Functions |
|---|---|
| Create and place | `create_prefab_from_actors`, `create_variant`, `place_prefab`, `add_nested_prefab` |
| Propagation | `propagate_to_loaded_instances`, `flush_pending_work`, `is_instance_stale`, `resync_instance` |
| Edit in place | `begin_edit`, `begin_nested_edit`, `commit_edit`, `discard_edit` |
| Overrides | `refresh_overrides`, `get_overrides`, `get_apply_targets`, `apply_override`, `apply_all_overrides`, `revert_override`, `revert_all_overrides`, `remove_stale_overrides`, `set_node_removal` |
| Lookup | `find_node_id_by_name`, `find_owning_instance`, `get_resolved_node_count`, `get_local_nodes`, `get_variant_overrides`, `prefab_depends_on` |
| Pack | `estimate_pack`, `pack_prefab`, `unpack_prefab` |
| Bake | `bake_to_actors`, `bake_to_level_instance` (the `packed` flag selects a Packed Level Actor) |
| Health | `build_health_report`, `show_health_report`, `count_health_issues` |

Related Python-visible types: enums `unreal.KPrefabPivot`, `unreal.KNodeRemoval`, `unreal.KOverrideKind`, `unreal.KHealthIssueType`; structs `unreal.KOverrideKey`, `unreal.KOverrideEntry`, `unreal.KPackSettings` (`min_instances`, `hism_threshold`), `unreal.KDrawCallEstimate`, `unreal.KHealthReport`, `unreal.KHealthIssue`.

### Example

```python
import unreal

actors = unreal.EditorLevelLibrary.get_selected_level_actors()

# Create a prefab from the selection and replace the actors with an instance.
# The second tuple element is the placed instance (output parameter OutInstance).
prefab, instance = unreal.KPrefabEditorLibrary.create_prefab_from_actors(
    actors, "/Game/Prefabs/P_House", unreal.KPrefabPivot.BOUNDS_BOTTOM_CENTER, True)

# Variant and placement
variant = unreal.KPrefabEditorLibrary.create_variant(prefab, "/Game/Prefabs/PV_HouseRed")
unreal.KPrefabEditorLibrary.place_prefab(variant, unreal.Transform())

# Overrides
for entry in unreal.KPrefabEditorLibrary.get_overrides(instance):
    unreal.log(f"{entry.node_name}: {entry.key.get_editor_property('property')} = {entry.value}")
unreal.KPrefabEditorLibrary.revert_all_overrides(instance)

# Edit in place
unreal.KPrefabEditorLibrary.begin_edit(instance)
# ... change actors / components of the instance ...
unreal.KPrefabEditorLibrary.commit_edit(instance)

# Pack with a draw-call estimate
settings = unreal.KPackSettings()
estimate = unreal.KPrefabEditorLibrary.estimate_pack(prefab, settings)
unreal.log(f"draw calls {estimate.draw_calls_before} -> {estimate.draw_calls_after}")

# Health report
report = unreal.KPrefabEditorLibrary.build_health_report()
unreal.log(f"{len(report.issues)} issues")
```

Runtime-side calls work from Python too, for example `unreal.KPrefabLibrary.get_cooked_node_count(prefab)`.

## CI

The regression suite (`Plugins/KeystonePrefabs/Content/Python/keystone_tests`) runs inside a headless editor, because some operations it covers (actor duplication, copy/paste) need the full editor rather than the `pythonscript` commandlet. `Scripts/RunTests.ps1` runs it and then exercises the `UpdatePrefabInstances` commandlet against the World Partition map the tests leave stale on disk (`-Verify` must fail, `-DryRun` must list the files, the update must succeed, `-Verify` must then pass):

```
powershell -ExecutionPolicy Bypass -File Plugins/KeystonePrefabs/Scripts/RunTests.ps1 -Engine "D:\Program Files\Epic Games\UE_5.8" -Project "D:\Path\MyProject.uproject"
```

- Exit code 0 when everything passes, 1 when a test or commandlet stage fails, 2 when the editor produced no result.
- `-Tests "test_variant_chain,test_pack_1000_meshes"` runs a subset (the commandlet stage only runs when the World Partition test ran).
- The editor run is `UnrealEditor-Cmd.exe <uproject> -ExecutePythonScript="<abs>/keystone_tests/run_all.py" -unattended -nullrhi -nosplash -nosound`; results go to `Saved/KeystoneTests/result.json` and `run.log`. Test content is written under `/Game/__KeystoneTests` and deleted afterwards.
- A third stage runs the editor UI automation tests (`Keystone.UI.*`: Details panel, property-row Revert/Apply indicators, context menus, edit mode, Outliner column, thumbnail, drag-drop placement) with `-RenderOffscreen -ExecCmds="Automation RunTests Keystone.UI; Quit"`. It needs a GPU; pass `-SkipUI` on GPU-less runners. Widget and thumbnail captures are written to `Saved/KeystoneTests/UI/*.png`.
- `-Offscreen` renders off-screen with `-RenderOffscreen` instead of `-NullRHI` (needs a GPU). On UE 5.7 use it: the editor crashes (integer divide by zero inside the engine) when a script spawns an actor under `-NullRHI`.
- The runner quits the editor a few ticks after the tests instead of in the same tick. Quitting immediately left the Content Browser sync that the engine schedules after creating a Packed Level Actor blueprint pending, and on UE 5.7 that made the editor crash while exiting.
- Run it once per engine version you ship (5.7 and 5.8). Both pass on UE 5.7.4 and 5.8.4.

### Packaged runtime smoke test

`Scripts/RunRuntimeSmoke.ps1` creates `/Game/KeystoneSmoke` (a prefab, a variant with a mesh override and a removed node, and a map placing both), packages the project for Win64 with that map, runs the packaged game headless with
`-ExecCmds="Keystone.SpawnPrefab /Game/KeystoneSmoke/P_Smoke_V.P_Smoke_V 0 2000 0, Keystone.ListPrefabs, quit"` and checks the log: the placed instances load from the cooked level and the spawned variant is built from cooked data.

It needs a C++ project: a content-only project packages the generic game executable, which does not include the code modules of project plugins. On UE 5.7 pass `-Offscreen` (see above) for the same reason as `RunTests.ps1`; it needs a GPU.

The two console commands are part of the runtime module and are available in any non-shipping build:

| Command | Effect |
|---|---|
| `Keystone.SpawnPrefab <ObjectPath> [X Y Z]` | Spawns a prefab or variant through `UKPrefabLibrary::SpawnPrefab`. |
| `Keystone.ListPrefabs` | Logs every prefab instance in the world and the object built for each node. |

### Fab submission archive

`Scripts/PackageForFab.ps1 -Engine <engine dir> -EngineVersion 5.8.0 [-OutDir <dir>] [-ContentFrom <Content dir>]` stages a clean copy of the plugin (with `Docs/` copied from `docs/site`), checks it against the Fab code-plugin rules (required folders, `FilterPlugin.ini`, `PlatformAllowList`, `FabURL`, copyright notices, path length and naming, no executables), checks that every `.uasset`/`.umap` was saved by the target engine version, compiles it with `RunUAT BuildPlugin -Rocket` for Win64 and fails on any warning from plugin sources, then writes `KeystonePrefabs_<version>.zip`. Without `-OutDir` the output goes to the project's `Saved/Fab/<version>` folder (the folder is emptied first). Run it once per engine version: Fab needs one archive per supported version.

Assets load only in the engine version that saved them or a newer one, so the demo content in the repository (saved by 5.8) cannot go into the 5.7 archive. For 5.7, run `Content/Python/keystone_demo/build_demo_map.py` in a 5.7 project that has the plugin, then pass that project's `Plugins/KeystonePrefabs/Content` folder as `-ContentFrom`; it replaces the staged `Content` folder.

## UpdatePrefabInstances commandlet

Re-syncs prefab instances in maps, including instances in unloaded World Partition cells, and re-saves only the files that change.

```
UnrealEditor-Cmd.exe "<uproject>" -run=UpdatePrefabInstances -Map=/Game/Maps/Town [-Map=...] [-AllMaps] [-Prefab=/Game/Prefabs/P_House] [-DryRun] [-Verify] [-CheckOut] [-BatchSize=50]
```

| Switch | Meaning |
|---|---|
| `-Map=` | Map to process. May be repeated or given as a comma list. Accepts a package name, an object path or a short name. |
| `-AllMaps` | Also process every map (World asset) under `/Game`. |
| `-Prefab=` | Only instances whose prefab is, or depends on (variant parent or nesting), this prefab asset. May be repeated. |
| `-DryRun` | Prints the files that would change and modifies nothing. Exit code 0. |
| `-Verify` | Prints the same listing and returns exit code 1 if any instance is stale. Use it as a CI gate. |
| `-CheckOut` | Checks files out through the revision control provider before saving (or makes read-only files writable when there is no provider). Without it, read-only files are not saved and that is reported as an error. |
| `-BatchSize=` | Stale instances loaded and saved per batch in World Partition maps, with garbage collection between batches; also the size of check-out and save chunks. Default 50, minimum 1. |

Behaviour:

- In World Partition maps, stale instances are found from actor descriptors without loading them. **The list of external actor files that will change is printed before anything is loaded.** Node actors (actors attached to an instance) are loaded and saved together with their instance, and files of node actors destroyed by the re-sync are deleted.
- In non-World Partition maps, the map is loaded, instances re-synced and the map and external actor packages saved.
- Exit code 1 on: usage error, map not found or failed to load, re-sync or save failure, or `-Verify` with stale instances. Otherwise 0.
- The log (category `LogKeystonePrefabs`) ends with a summary: maps, instances checked, stale and updated, failures, files saved and deleted.

See [Source control behaviour](source_control.md) for what each operation touches.
