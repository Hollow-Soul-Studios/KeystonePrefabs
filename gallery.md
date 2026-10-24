---
title: Screenshots
---

# Screenshots

Taken from the demo map that ships with the plugin (`/KeystonePrefabs/Demo/Maps/KeystonePrefabs_Demo`; in the Content Browser, enable *Show Plugin Content*) in the UE 5.8.4 editor. Each section links to the matching part of the [documentation](index.md).

## Overview

Nested prefabs, variants, per-instance overrides and packed instancing in one level. The selected instance is the nested prefab `P_Plaza`; the Outliner shows the prefab status column and the Details panel shows the *Prefab* category.

![Overview](images/02_overview.png)

## Overrides

The roof of this instance was changed to a cone. The Details panel lists the override with Revert and Apply. See [Overrides](index.md#5-overrides).

![Overrides](images/03_overrides.png)

## Variants

A variant with square pillars next to its parent. A variant stores only its differences from the parent prefab. See [Variants](index.md#7-variants).

![Variant](images/04_variant.png)

## Pack

60 fence posts packed into one instanced mesh component; the demo label shows the estimated draw calls before and after. See [Pack and Unpack](index.md#10-pack-and-unpack).

![Pack](images/05_pack.png)

The Pack command shows the estimate before it changes the asset:

![Pack dialog](images/10_pack_dialog.png)

## Edit in place

Editing the prefab in the level; Commit writes the changes to the asset and updates every instance, Discard throws them away. See [Edit, Commit, Discard](index.md#4-edit-commit-discard).

![Edit in place](images/06_edit_in_place.png)

## Menus

Actor context menu with the *Keystone Prefab* section and the Bake submenu. See [Bake to native](index.md#11-bake-to-native) and [Remove Child](index.md#6-remove-child-hide-or-delete).

![Actor context menu](images/08_context_menu.png)

Prefab asset menu in the Content Browser:

![Asset menu](images/09_asset_menu.png)

## Content Browser

Prefab assets and a variant with thumbnails.

![Content Browser](images/07_content_browser.png)
