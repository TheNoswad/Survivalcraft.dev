# Survivalcraft Mod Info File Configuration Guide

!!! info "Attribution"

    This text is derived and translated to English from [`docs/ModInfoConfig.md`](https://gitee.com/SC-SPM/SurvivalcraftApi/blob/SCAPI1.9/docs/ModInfoConfig.md) from the Survivalcraft API Gitee Repository.

This document explains the parameters and formatting rules for Survivalcraft modded edition mod configuration files (typically JSON format).

## Parameter Overview

| Field Name | Type | Description | Required |
| :---: | :---: | :----- | :--: |
| **Name** | String | Mod name | Yes |
| **Version** | String | Mod version. Recommended to follow [Semantic Versioning (SemVer)](https://semver.org/) | Yes |
| **ApiVersion** | String | The API version the mod targets. Mods targeting below 1.8 will warn that they may not work, but loading will still be attempted | Yes |
| **Description** | String | Mod description | No |
| **ScVersion** | String | The Survivalcraft game version the mod targets. Has no practical effect | Yes |
| **LoadOrder** | Integer | Mod load order. Lower values load first; defaults to 0.<br/>Recommended: theme mods −100000 to −10000, utility mods 10000 to 100000.<br/>Note: players can manually adjust the order in-game | No |
| **NonPersistentMod** | Boolean | Non-persistent mod; defaults to false.<br/>If true, the mod will not be recorded in saves, so removing it will not trigger a "mod not installed" warning when loading a save. Intended for mods that do not store data in saves, add blocks, or add entities | No |
| **GameplayImpactLevel** | String | Gameplay impact level. Indicates the degree to which the mod affects game balance. Saved to the save file; defaults to `Cosmetic`. See details below | No |
| **Link** | String | Mod link, openable from the mod management screen. Recommended to point to a page with detailed information or a bug report channel | No |
| **Author** | String | Mod author | No |
| **PackageName** | String | Mod package name, used to distinguish between mods. If multiple mods share the same package name, none of them will load | Yes |
| **Dependencies** | Object | Other mods this mod depends on. See formatting rules below | No |

### GameplayImpactLevel Values

| Value | Name | Examples |
| :----: | :----: | :--- |
| `Cosmetic` | Cosmetic only | Texture packs, font packs, shaders |
| `Assist` | Light assistance | Minimap (no x-ray), chest sorting, mob health display, reasonable-cost player upgrades |
| `Turbo` | Strong assistance | One-tap tree felling, automation, ore radar, low-cost player upgrades, reasonable-cost rule-breaking |
| `Break` | Rule-breaking | Drastically boosting player ability at no/low cost, flight, portals, drop/yield multipliers |
| `Godmode` | God mode | Invincibility, teleportation, infinite resources |

## Dependencies

In modded editions prior to 1.8.2, each `Dependencies` entry only supported a single fixed version number. Starting from 1.8.2, `Dependencies` supports version ranges, using NuGet-style and a subset of SemVer-style syntax, as described below.

### NuGet Style

| Example | Logical Range | Description |
| :--: | :------: | :--- |
| `1.0` | $x \ge 1.0$ | A bare version number indicates a minimum version requirement |
| `[1.0,)` | $x \ge 1.0$ | `[` means greater than or equal to |
| `(1.0,)` | $x > 1.0$ | `(` means strictly greater than |
| `(,1.0]` | $x \le 1.0$ | `]` means less than or equal to |
| `(,1.0)` | $x < 1.0$ | `)` means strictly less than |
| `[1.0, 2.0]` | $1.0 \le x \le 2.0$ | Closed interval |
| `[1.0, 2.0)` | $1.0 \le x < 2.0$ | Left-closed, right-open interval |
| `(1.0)` | $/$ | Invalid |

### SemVer Style

| Example | Logical Range | Description |
| --- | --- | --- |
| `=1.0.0` | $x = 1.0.0$ | Exact match |
| `>1.0.0` | $x > 1.0.0$ | Greater than |
| `>=1.0.0` | $x \ge 1.0.0$ | Greater than or equal to |
| `<2.0.0` | $x < 2.0.0$ | Less than |
| `<=2.0.0` | $x \le 2.0.0$ | Less than or equal to |
| `^1.0.0` | $1.0.0 \le x < 2.0.0$ | Same major version |
| `~1.0.0` | $1.0.0 \le x < 1.1.0$ | Same minor version |

> **Note:** NuGet style and SemVer style **cannot be mixed**.  
> `ApiVersion` also supports the syntax above, but as before, it has no practical effect.

Version numbers follow the format `Major.Minor.Patch.Revision[-Suffix]` — that is, major, minor, patch, and revision numbers, all numeric and dot-separated, with an optional suffix preceded by `-`. Example: `1.2.3.4-beta`.  
If parsing fails, the loader will fall back to plain string comparison rather than range evaluation (matching the behavior of older modded editions).

Version 1.8.2 also introduced a new object syntax for `Dependencies`. The old array syntax:

```json
{
    "Dependencies": [
        "PackageNameOfModA:1.2",
        "PackageNameOfModB:3.4",
        "PackageNameOfModC:5.6"
    ]
}
```

The new object syntax:

```json
{
    "Dependencies": {
        "PackageNameOfModA": "1.2",
        "PackageNameOfModB": "[3.4,)",
        "PackageNameOfModC": ">=5.6"
    }
}
```

Both forms are equivalent, but the new syntax is more intuitive and readable.
