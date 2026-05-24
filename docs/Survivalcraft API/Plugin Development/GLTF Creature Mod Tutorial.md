!!! info inline end "Attribution"

    This text is derived and translated to English from [`docs/GltfCreatureModTutorial.md`](https://gitee.com/SC-SPM/SurvivalcraftApi/blob/SCAPI1.9/docs/GltfCreatureModTutorial.md) from the Survivalcraft API Gitee Repository.

This tutorial explains how to create creature mods using glTF/GLB models in the Survivalcraft API. Two approaches are supported: a pure data-driven mode requiring no code, and an advanced mode requiring C# components.

## Table of Contents

1. [Overview](#1-overview)
2. [Prerequisites](#2-prerequisites)
3. [Pure Data-Driven: Minimal Creature Mod](#3-pure-data-driven-minimal-creature-mod)
4. [Project Structure Breakdown](#4-project-structure-breakdown)
5. [Database Template Configuration](#5-database-template-configuration)
6. [Animation Configuration File](#6-animation-configuration-file)
7. [Adding Custom Behavior (C# Code)](#7-adding-custom-behavior-c-code)
8. [Building and Installing](#8-building-and-installing)
9. [Complete Example Reference](#9-complete-example-reference)

---

## 1. Overview

### Advantages of glTF Models

Compared to the game's original `.dae` (Collada) models, glTF models offer the following advantages:

- **Keyframe animation support**: Use animation clips embedded directly in the model file, no need to drive bones manually via C# code
- **Codeless creature creation**: A complete creature mod can be built using only configuration files
- **Rich model resources**: A large library of public glTF models is available (e.g. [glTF-Sample-Assets](https://github.com/KhronosGroup/glTF-Sample-Assets))
- **Standardized format**: glTF is a 3D asset standard defined by Khronos

### Supported Model Formats

| Format | Extension | Notes |
|--------|-----------|-------|
| glTF | `.gltf` + `.bin` + texture files | Text format with external resource references |
| GLB | `.glb` | Binary format with all assets packed into a single file |
| Collada | `.dae` | The game's original format; still supported but glTF is recommended |

---

## 2. Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- Survivalcraft API mod template: [SurvivalcraftTemplateModForAPI](https://gitee.com/SC-SPM/SurvivalcraftTemplateModForAPI)
- A glTF or GLB model file with skeletal rigging (animation clips required)

---

## 3. Pure Data-Driven: Minimal Creature Mod

This is the simplest approach — **no C# code required**. You only need a model file, animation config, and a database template.

### Required Files

```
YourMod/
├── Assets/
│   ├── Animations/
│   │   └── YourCreature.json      # Animation config
│   ├── Lang/
│   │   ├── en-US.json             # English localization
│   │   └── zh-CN.json             # Chinese localization
│   ├── Models/
│   │   └── YourCreature/
│   │       ├── Creature.gltf      # glTF model
│   │       ├── Creature.bin       # Vertex/index data
│   │       └── Texture.png        # Texture
│   └── YourDatabase.xdb           # Database template
├── modinfo.json                   # Mod metadata
└── YourMod.csproj                 # Project file
```

### modinfo.json

```json
{
  "Name": "YourCreatureMod",
  "Version": "1.0.0",
  "ApiVersion": "1.9.1.3",
  "Description": "Adds a creature using glTF model",
  "ScVersion": "1.90.3.0",
  "GameplayImpactLevel": "Cosmetic",
  "PackageName": "yourname.YourCreatureMod",
  "Author": "YourName"
}
```

> Setting `GameplayImpactLevel` to `Cosmetic` indicates this mod only adds visual content and does not affect game balance.

### Localization Files

`Assets/Lang/en-US.json`:
```json
{
  "DisplayName:YourCreature": "Your Creature",
  "Description:YourCreature": "A creature with glTF model"
}
```

`Assets/Lang/zh-CN.json`:
```json
{
  "DisplayName:YourCreature": "你的生物",
  "Description:YourCreature": "使用 glTF 模型的生物"
}
```

---

## 4. Project Structure Breakdown

### Model Files

Place your glTF model files under the `Assets/Models/` directory:

- **glTF format**: Preserve the directory structure of the `.gltf`, `.bin`, and texture files
- **GLB format**: A single `.glb` file is sufficient

Reference the model in-game using the `ModelName` parameter, omitting the file extension:

```xml
<!-- References Assets/Models/YourCreature/Creature.gltf -->
<Parameter Name="ModelName" Value="Models/YourCreature/Creature" Type="string" />
```

### Animation Clips in the Model

The game automatically loads all animation clips found in the glTF file. In the animation config file, you reference them by their clip name.

To view animation clip names in a model, use [gltf-report](https://github.com/nicebyte/gltf-report) or any glTF viewer.

### csproj Configuration

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <EnableDefaultItems>false</EnableDefaultItems>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="SurvivalcraftAPI.Survivalcraft" Version="1.9.1.3" />
  </ItemGroup>

  <ItemGroup>
    <Content Include="Assets\**" CopyToOutputDirectory="PreserveNewest" />
    <Content Include="modinfo.json" CopyToOutputDirectory="PreserveNewest" />
  </ItemGroup>

  <Target Name="PackageMod" AfterTargets="Build">
    <ZipDirectory SourceDirectory="$(OutputPath)"
                  DestinationFile="$(OutputPath)..\$(AssemblyName).scmod"
                  Overwrite="true" />
  </Target>
</Project>
```

---

## 5. Database Template Configuration

The database template (`.xdb` file) defines a creature's properties and behaviors. The key is to inherit from an existing template and only override the parameters you need to change.

### Inheriting from an Existing Creature

The simplest approach is to inherit from an existing creature template and override only the parameters you want to modify. Common parent templates:

| Parent Template | Guid | Use Case |
|-----------------|------|----------|
| Wolf | `e4275171-a39f-413f-8888-4c472868364d` | Four-legged ground creature |
| LandAnimal | `3f077159-f492-419b-859a-bb051de6339f` | Generic land creature |
| Bird | See game database | Flying creature |

### Four-Legged Animal Example

Inherit the Wolf template, replace the model and animation config, and disable the old procedural animations:

```xml
<EntityTemplate Name="YourCreature"
               Guid="your-guid-here"
               InheritanceParent="e4275171-a39f-413f-8888-4c472868364d">

  <!-- Body dimensions and mass -->
  <MemberComponentTemplate Name="Body" Guid="...">
    <Parameter Name="BoxSize" Value="0.6,0.6,0.6" Type="Vector3" />
    <Parameter Name="Mass" Value="40" Type="float" />
  </MemberComponentTemplate>

  <!-- Movement parameters -->
  <MemberComponentTemplate Name="Locomotion" Guid="...">
    <Parameter Name="WalkSpeed" Value="5" Type="float" />
    <Parameter Name="TurnSpeed" Value="10" Type="float" />
  </MemberComponentTemplate>

  <!-- Key: configure the glTF model and animation -->
  <MemberComponentTemplate Name="FourLeggedModel" Guid="..."
                           InheritanceParent="76d24f74-5f25-4d30-92ef-658c1a4d66c8">
    <!-- Path to the model file (without extension) -->
    <Parameter Name="ModelName" Value="Models/YourCreature/Creature" Type="string" />

    <!-- Path to the animation config (without .json extension) -->
    <Parameter Name="AnimationConfigPath" Value="Animations/YourCreature" Type="string" />

    <!-- Disable old procedural animation parameters -->
    <Parameter Name="WalkFrontLegsAngle" Value="0" Type="float" />
    <Parameter Name="WalkHindLegsAngle" Value="0" Type="float" />
    <Parameter Name="CanTrot" Value="False" Type="bool" />
  </MemberComponentTemplate>

  <!-- Creature classification -->
  <MemberComponentTemplate Name="Creature" Guid="...">
    <Parameter Name="Category" Value="LandPredator" Type="Game.CreatureCategory" />
    <Parameter Name="DisplayName" Value="[DisplayName:YourCreature]" Type="string" />
    <Parameter Name="Description" Value="[Description:YourCreature]" Type="string" />
  </MemberComponentTemplate>

  <!-- Spawn egg -->
  <ParameterSet Name="CreatureEggData" Guid="...">
    <Parameter Name="TextureSlot" Value="15" Type="int" />
    <Parameter Name="Color" Value="204,102,0" Type="Color" />
    <Parameter Name="EggTypeIndex" Value="682" Type="int" />
  </ParameterSet>
</EntityTemplate>
```

### Key Configuration Notes

- **`InheritanceParent`**: Inherits from an existing creature template, automatically acquiring all its components and behaviors
- **`FourLeggedModel`'s `InheritanceParent`**: Must inherit from `76d24f74-5f25-4d30-92ef-658c1a4d66c8` (the FourLeggedModel base template)
- **Disabling old animation parameters**: Set `WalkFrontLegsAngle` and `WalkHindLegsAngle` to `0` and `CanTrot` to `False` to prevent conflicts with glTF keyframe animations

---

## 6. Animation Configuration File

The animation config file (`.json`) is the core of a glTF creature mod. It defines how animations play and transition.

### Minimal Configuration

The following is a minimal config using the built-in FourLegged template:

```json
{
  "template": "FourLegged",
  "rootBoneRotation": 180,
  "modelScale": 0.01,

  "animations": {
    "idle": {
      "source": "Survey",
      "speed": 1.0,
      "loop": true,
      "blendDuration": 0.3
    },
    "walk": {
      "source": "Walk",
      "speed": "[SpeedAbs] * 0.65",
      "loop": true,
      "blendDuration": 0.2
    },
    "run": {
      "source": "Run",
      "speed": "[SpeedAbs] * 0.5",
      "loop": true,
      "blendDuration": 0.15
    }
  },

  "states": {
    "gait": {
      "layer": "Base",
      "rules": [
        {
          "condition": "IsDead",
          "animation": null
        },
        {
          "condition": "[SpeedAbs] > 0.7 * [WalkSpeed]",
          "animation": { "source": "run" }
        },
        {
          "condition": "[SpeedAbs] > 0.3",
          "animation": { "source": "walk" }
        },
        {
          "condition": "true",
          "animation": { "source": "idle" }
        }
      ]
    },
    "death": {
      "layer": "Death",
      "rules": [
        {
          "condition": "[DeathPhase] > 0",
          "animation": { "source": "driver:Death" }
        },
        {
          "condition": "true",
          "animation": null
        }
      ]
    }
  }
}
```

### Top-Level Property Reference

| Property | Type | Description |
|----------|------|-------------|
| `template` | string | Animation template name: `Simple`, `FourLegged`, `Human`, `Bird`, `FlightlessBird`, `Fish`, or a custom template path |
| `rootBoneRotation` | float | Root bone rotation correction (degrees), used to fix model facing direction. Common values: `0` (model faces +Z) or `180` (model faces -Z) |
| `modelScale` | float | Model scale. `1.0` is original size; `0.01` is for models authored in centimeter units |
| `layers` | object | Layer override config (optional; overrides template defaults) |
| `animations` | object | Animation alias definitions |
| `states` | object | State rules |
| `parameters` | object | Initial parameter values |

### Choosing a Template

| Template | Creature Type | Preset Layers | Preset State Tracks |
|----------|--------------|---------------|---------------------|
| `Simple` | Simple entities | Base | Gait (Idle) |
| `FourLegged` | Four-legged animals | Base, Head, Death | Gait, Activity, Death |
| `Human` | Humanoid | Base, Activity, Ride, Death | Locomotion, Activity, Ride, Death |
| `Bird` | Birds | Base, Head, Death | Locomotion, Activity, Death |
| `FlightlessBird` | Flightless birds | Base, Head, Death | Locomotion, Activity, Death |
| `Fish` | Fish | Base, Head, Death | Swim, Activity, Death |

For full template details and JSON format reference, see [Animation Configuration.md](Animation Configuration.md).

### Speed Expressions

The `speed` property of an animation supports expressions, referencing runtime parameters using `[ParameterName]` syntax:

```json
"speed": "[SpeedAbs] * 0.65"
```

Commonly used parameters:
- `[SpeedAbs]`: Absolute value of the creature's current movement speed
- `[WalkSpeed]`: The creature's configured walk speed
- `[DeathPhase]`: Death phase (0–1)

For more expression usage, see [Animation Configuration.md](Animation Configuration.md).

---

## 7. Adding Custom Behavior (C# Code)

When pure data-driven is not flexible enough, you can write C# components to implement custom behavior. This requires:

1. Creating a custom `ComponentCreatureModel` subclass to sync animation parameters
2. Creating behavior components for complex game logic
3. Registering custom components in the database template

### 7.1 Project Configuration

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <EnableDefaultCompileItems>false</EnableDefaultCompileItems>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="SurvivalcraftAPI.Survivalcraft" Version="1.9.1.3" />
  </ItemGroup>

  <ItemGroup>
    <Compile Include="Components\*.cs" />
    <Content Include="Assets\**" CopyToOutputDirectory="PreserveNewest" />
    <Content Include="modinfo.json" CopyToOutputDirectory="PreserveNewest" />
  </ItemGroup>

  <Target Name="PackageMod" AfterTargets="Build">
    <ZipDirectory SourceDirectory="$(OutputPath)"
                  DestinationFile="$(OutputPath)..\$(AssemblyName).scmod"
                  Overwrite="true" />
  </Target>
</Project>
```

### 7.2 Custom Model Component

The custom model component is responsible for syncing game state to the AnimationController's parameters:

```csharp
using Engine.Animation;

namespace Game;

public class ComponentYourCreatureModel : ComponentCreatureModel
{
    public override void SyncAnimationParameters()
    {
        // Must call the base method — it syncs a large number of built-in parameters
        // including: Speed, SpeedAbs, DeathPhase, IsDead, WalkSpeed, LookAngleX/Y, etc.
        base.SyncAnimationParameters();

        var controller = AnimationController;
        if (controller == null) return;

        // Add mod-specific custom parameters (not provided by the base class)
        // Use Parameters.SetString to set state track values, triggering state rule evaluation
        var locomotion = m_componentCreature.ComponentLocomotion;
        float speed = Vector3.Dot(
            m_componentCreature.ComponentBody.Velocity,
            m_componentCreature.ComponentBody.Forward
        );
        float walkSpeed = locomotion?.WalkSpeed ?? 5f;
        string gait = speed > 0.7f * walkSpeed ? "Run" :
                      speed > 0.2f ? "Walk" : "Idle";
        controller.Parameters.SetString("Gait", gait);
    }
}
```

**`SyncAnimationParameters()`** is the key method. It is called every frame and is responsible for writing game state into the AnimationController's parameter system.

> **Important**: You must call `base.SyncAnimationParameters()`. The base class syncs a large number of built-in parameters (Speed, SpeedAbs, DeathPhase, IsDead, WalkSpeed, LookAngleX/Y, BodyHeight, etc.). Skipping this call will break basic functionality.

To set state track values, use `Parameters.SetString("Gait", "Run")` or `Parameters.SetBool("Death", true)` — not `SetState()`. Parameter changes drive animation switching indirectly through the condition expressions in state rules.

### 7.3 Manual Animation Control

For special actions like attacks, you can use the manual animation control API:

```csharp
// Force-play the attack animation
AnimationController.PlayAnimation("UpperBodyAction", "bite", loop: true);

// Release manual control after the attack, returning to state-rule-driven playback
AnimationController.ReleaseManualControl("UpperBodyAction");

// Check whether a layer is currently under manual control
bool isManual = AnimationController.IsManualControl("UpperBodyAction");
```

### 7.4 Animation Event Handling

Define events in the animation config and handle them in C#:

```json
"bite": {
  "source": "Bite",
  "loop": true,
  "events": [
    { "time": 0.577, "name": "AttackHit" }
  ]
}
```

```csharp
public class ComponentYourCreatureModel : ComponentCreatureModel
{
    // The base class ComponentCreatureModel.SetModel() already subscribes
    // AnimationController.OnAnimationEvent to HandleAnimationEvent automatically.
    // Simply override this method to handle events.

    public override void HandleAnimationEvent(AnimationEvent animationEvent)
    {
        // Call the base class first to handle built-in events (Footstep, AttackHit, etc.)
        base.HandleAnimationEvent(animationEvent);

        switch (animationEvent.Name)
        {
            case "MyCustomEvent":
                // Handle custom event
                DoSomething();
                break;
        }
    }
}
```

> `ComponentCreatureModel.SetModel()` already subscribes `AnimationController.OnAnimationEvent` to `HandleAnimationEvent` automatically. Subclasses only need to override `HandleAnimationEvent` — do not subscribe manually. The base class already handles built-in events such as `AttackHit`, `Footstep`, `AttackStart`, and `AttackEnd`.

### 7.5 Registering Custom Components in the Database

```xml
<!-- Define the component template -->
<ComponentTemplate Name="YourCreatureModel"
                   Description="Custom creature model"
                   Guid="your-guid"
                   InheritanceParent="681a5886-5bff-418a-bf5f-ac84f290a311">
  <Parameter Name="Class" Value="Game.ComponentYourCreatureModel" Type="string" />
  <Parameter Name="ModelName" Value="" Type="string" />
  <Parameter Name="TextureOverride" Value="" Type="string" />
  <Parameter Name="AnimationConfigPath" Value="" Type="string" />
</ComponentTemplate>

<!-- Use it in an entity -->
<MemberComponentTemplate Name="YourCreatureModel" Guid="..."
                         InheritanceParent="your-component-template-guid">
  <Parameter Name="ModelName" Value="Models/YourCreature/Model" Type="string" />
  <Parameter Name="AnimationConfigPath" Value="Animations/YourCreature" Type="string" />
</MemberComponentTemplate>
```

> `InheritanceParent="681a5886-5bff-418a-bf5f-ac84f290a311"` points to the ComponentCreatureModel base template.

---

## 8. Building and Installing

### Building

```bash
dotnet build
```

After a successful build, the `.scmod` file will be generated in the `bin/Debug/` directory.

### Installing

Copy the `.scmod` file to the game's Mods directory:

- **Windows**: `Survivalcraft.Windows/bin/Debug/Mods/` (development environment)
- Or the `Mods/` folder inside the game's installation directory

### Testing

1. Launch the game
2. Use the corresponding creature's spawn egg to spawn the entity
3. Observe model loading and animation playback

---

## 9. Complete Example Reference

The content of this tutorial is based on two complete example mods:

| Mod | Description |
|-----|-------------|
| **BaseGltfFoxMod** | A codeless, pure data-driven four-legged creature. Uses the FourLegged template with Survey, Walk, and Run animations. |
| **AdvancedGltfFoxMod** | An advanced four-legged creature. Includes a custom model component, IK head tracking, animation-driven jumping, howling/sitting behaviors, attack animations, and Root Motion jumping. |

For more API details, see:
- [Animation Configuration.md](Animation Configuration.md) — Complete animation config JSON reference
- [Advanced Animation.md](Advanced Animation.md) — Advanced topics: IK, Root Motion, expressions, and more
- [Creature Model System.md](Creature Model System.md) — Overall architecture of the model system
