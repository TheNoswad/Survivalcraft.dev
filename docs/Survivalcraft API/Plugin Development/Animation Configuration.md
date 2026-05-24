# Animation Configuration JSON Reference

!!! info inline end "Attribution"

    This text is derived and translated to English from [`docs/AnimationConfigReference.md`](https://gitee.com/SC-SPM/SurvivalcraftApi/blob/SCAPI1.9/docs/AnimationConfigReference.md) from the Survivalcraft API Gitee Repository.

This document is a complete format reference for glTF creature animation configuration files. Animation config files define how animations play, transition, and blend.

Config files are located in the `Assets/Animations/` directory and referenced via the `AnimationConfigPath` parameter in the database template (without the `.json` extension).

## Table of Contents

1. [Top-Level Structure](#1-top-level-structure)
2. [Template System](#2-template-system)
3. [Layers](#3-layers)
4. [Animation Definitions](#4-animation-definitions)
5. [State Rules](#5-state-rules)
6. [Drivers](#6-drivers)
7. [Animation Events](#7-animation-events)
8. [Root Motion Configuration](#8-root-motion-configuration)
9. [Expression Overview](#9-expression-overview)
10. [JSON Inheritance](#10-json-inheritance-extends)
11. [Complete Examples](#11-complete-examples)

---

## 1. Top-Level Structure

```json
{
  "template": "FourLegged",
  "rootBoneRotation": 180,
  "modelScale": 0.01,
  "layers": { ... },
  "animations": { ... },
  "states": { ... },
  "parameters": { ... }
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `template` | string | `"Simple"` | Animation template name or custom template path |
| `rootBoneRotation` | float | `0` | Root bone rotation correction (degrees). Used when model faces the wrong direction; common values are `0` or `180` |
| `modelScale` | float | `1` | Model scale. Use `0.01` for centimeter-unit models, `1.0` for meter-unit models |
| `layers` | object | `{}` | Layer configuration; overrides or supplements template layers |
| `animations` | object | `{}` | Mapping of animation aliases to AnimationReferences |
| `states` | object | `{}` | Mapping of state tracks to state rules |
| `parameters` | object | `{}` | Initial parameter values |

---

## 2. Template System

Templates pre-define layer, state track, and driver configurations. Choosing the right template significantly reduces the amount of configuration needed.

### Built-in Templates

#### Simple

For simple entities with only basic gait control.

```json
{
  "name": "Simple",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" }
  },
  "stateTracks": {
    "Gait": { "type": "Enum", "defaultValue": "Idle", "enumValues": ["Idle"] }
  }
}
```

#### FourLegged

For four-legged animals (wolves, foxes, cats, etc.).

```json
{
  "name": "FourLegged",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" },
    "Head": { "index": 1, "blendMode": "Override", "boneMask": ["Head", "Neck"] },
    "Death": { "index": 2, "blendMode": "Override" }
  },
  "stateTracks": {
    "Gait": { "type": "Enum", "defaultValue": "Idle", "enumValues": ["Idle", "Walk", "Trot", "Canter"] },
    "Activity": { "type": "Enum", "defaultValue": "None", "enumValues": ["None", "Feed", "Attack"] },
    "Death": { "type": "Float", "defaultValue": 0, "minValue": 0, "maxValue": 1 }
  }
}
```

#### Human

For humanoid creatures.

```json
{
  "name": "Human",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" },
    "Activity": { "index": 1, "blendMode": "Additive", "boneMask": ["Hand1", "Hand2"] },
    "Ride": { "index": 2, "blendMode": "Override" },
    "Death": { "index": 3, "blendMode": "Override" }
  },
  "stateTracks": {
    "Locomotion": { "type": "Enum", "defaultValue": "Idle", "enumValues": ["Idle", "Walk", "Fly"] },
    "Activity": { "type": "Enum", "defaultValue": "None", "enumValues": ["None", "Attack", "Aim"] },
    "Ride": { "type": "Enum", "defaultValue": "None", "enumValues": ["None", "Riding"] },
    "Death": { "type": "Float", "defaultValue": 0, "minValue": 0, "maxValue": 1 }
  }
}
```

#### Bird

For birds.

```json
{
  "name": "Bird",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" },
    "Head": { "index": 1, "blendMode": "Override", "boneMask": ["Head", "Neck"] },
    "Death": { "index": 2, "blendMode": "Override" }
  },
  "stateTracks": {
    "Locomotion": { "type": "Enum", "defaultValue": "Idle", "enumValues": ["Idle", "Walk", "Fly"] },
    "Activity": { "type": "Enum", "defaultValue": "None", "enumValues": ["None", "Peck", "Attack"] },
    "Death": { "type": "Float", "defaultValue": 0, "minValue": 0, "maxValue": 1 }
  }
}
```

#### FlightlessBird

For non-flying birds (chickens, penguins, etc.).

```json
{
  "name": "FlightlessBird",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" },
    "Head": { "index": 1, "blendMode": "Override", "boneMask": ["Head", "Neck"] },
    "Death": { "index": 2, "blendMode": "Override" }
  },
  "stateTracks": {
    "Locomotion": { "type": "Enum", "defaultValue": "Idle", "enumValues": ["Idle", "Walk"] },
    "Activity": { "type": "Enum", "defaultValue": "None", "enumValues": ["None", "Feed", "Attack"] },
    "Death": { "type": "Float", "defaultValue": 0, "minValue": 0, "maxValue": 1 }
  }
}
```

#### Fish

For fish.

```json
{
  "name": "Fish",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" },
    "Head": { "index": 1, "blendMode": "Override", "boneMask": ["Jaw"] },
    "Death": { "index": 2, "blendMode": "Override" }
  },
  "stateTracks": {
    "Swim": { "type": "Float", "defaultValue": 0 },
    "Activity": { "type": "Enum", "defaultValue": "None", "enumValues": ["None", "Bite"] },
    "Death": { "type": "Float", "defaultValue": 0, "minValue": 0, "maxValue": 1 }
  }
}
```

### Custom Templates

When built-in templates don't meet your needs, you can create custom template files in the `Assets/AnimationTemplates/` directory.

Custom template file format:

```json
{
  "name": "CustomCreature",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" },
    "UpperBodyAction": { "index": 1, "blendMode": "Override" },
    "Death": { "index": 2, "blendMode": "Override" }
  },
  "stateTracks": {
    "Gait": {
      "type": "Enum",
      "defaultValue": "Idle",
      "enumValues": ["Idle", "Walk", "Run", "Jump", "Fall", "Sit"]
    },
    "Activity": {
      "type": "Enum",
      "defaultValue": "None",
      "enumValues": ["None", "Feed", "Attack", "Bark", "Howl"]
    },
    "Death": {
      "type": "Bool",
      "defaultValue": false
    }
  }
}
```

Reference a custom template in your animation config by file path (without the `.template.json` extension):

```json
{
  "template": "AnimationTemplates/CustomCreature",
  ...
}
```

### Template Layer Fields

| Field | Type | Description |
|-------|------|-------------|
| `index` | int | Layer index; determines blend order (ascending) |
| `blendMode` | string | `"Override"` or `"Additive"` |
| `boneMask` | string[] | Bone filter list; only affects the specified bones |

### State Track Types

| type | Fields | Description |
|------|--------|-------------|
| `"Enum"` | `enumValues` (string[]), `defaultValue` | Discrete enumerated values, e.g. gait states |
| `"Bool"` | `defaultValue` | Boolean toggle, e.g. death |
| `"Float"` | `minValue`, `maxValue`, `defaultValue` | Continuous floating-point value |

---

## 3. Layers

Layers enable layered blending of animations. For example, base locomotion runs on the Base layer while head look-at runs on the Head layer — they operate independently and blend automatically.

In the config file, `layers` is used to override template layer defaults or to assign drivers to layers:

```json
"layers": {
  "Head": {
    "bones": ["b_Head_05"],
    "driver": {
      "type": "LookAt",
      "properties": {
        "TargetBoneName": "b_Head_05",
        "MaxAngleX": 40,
        "MaxAngleY": 15
      }
    }
  },
  "Death": {
    "driver": {
      "type": "Death",
      "properties": {
        "BodyDrop": -0.4
      }
    }
  }
}
```

### Layer Fields

| Field | Type | Description |
|-------|------|-------------|
| `bones` | string[] | Bone filter list (overrides template `boneMask`) |
| `driver` | object | Driver configuration (see [Drivers](#6-drivers)) |
| `blendMode` | string | Blend mode: `"override"` or `"additive"` |
| `blendCurve` | string | Blend curve: `"linear"` or `"smoothstep"` |

### Blend Modes

- **Override**: This layer's animation fully replaces the transforms of the specified bones. When multiple layers are present, later layers override earlier ones.
- **Additive**: This layer's animation is added on top of existing transforms. Suitable for layering local actions (e.g. arm motion on top of a full-body animation).

### Blending Rules

Layers are processed in ascending `index` order. Each layer's result is blended with the previous layer's output according to the blend mode and weight. The final bone transforms are used for rendering.

---

## 4. Animation Definitions

`animations` defines a mapping from animation aliases to concrete animation clips. State rules reference animations by alias.

### Basic Format

```json
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
  }
}
```

### AnimationReference Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | string | required | Animation source (see source formats below) |
| `speed` | float or string | `1.0` | Playback speed. Supports expression strings |
| `loop` | bool or string | `true` | Whether to loop. Supports expression strings |
| `blendDuration` | float or string | `0.3` | Blend transition time (seconds) when switching to this animation |
| `startPhase` | float or string | `0.0` | Playback start phase (0–1) |
| `endPhase` | float or string | `1.0` | Playback end phase (0–1) |
| `preservePose` | bool | `false` | Whether to hold the final pose after a non-looping animation ends |
| `events` | array | `[]` | List of animation events |
| `onComplete` | object | null | Action to take when the animation completes |
| `rootMotion` | object | null | Root motion configuration |

### Source Formats

`source` supports several formats:

| Format | Example | Description |
|--------|---------|-------------|
| Clip name | `"Walk"` | Directly references an animation clip in the model file |
| animation:// protocol | `"animation://Walk"` | Explicit clip reference (equivalent to the bare name) |
| driver: protocol | `"driver:LookAt"` | Uses a driver instead of an animation clip |
| External file | `"file:path/to/file.glb#AnimName"` | Loads an animation from an external GLB file |

### Phase Trimming (startPhase / endPhase)

Use different portions of a single animation clip as separate animations:

```json
"jump": {
  "source": "Jump",
  "loop": false,
  "blendDuration": 0.2
},
"jumpLandRecovery": {
  "source": "Jump",
  "startPhase": 0.652,
  "endPhase": 1.0,
  "loop": false,
  "blendDuration": 0.2
}
```

`jumpLandRecovery` reuses the 65.2%–100% portion of the Jump clip.

### preservePose

When set to `true`, bones hold their last-frame pose after a non-looping animation finishes, rather than reverting to the default pose. Useful for sustained poses following a one-shot action (e.g. death, a paused stealth stance):

```json
"death": {
  "source": "Death",
  "loop": false,
  "preservePose": true
}
```

---

## 5. State Rules

State rules define what animation plays under what conditions. They are the primary driving mechanism of the state machine.

### Basic Format

```json
"states": {
  "gait": {
    "layer": "Base",
    "rules": [
      { "condition": "[SpeedAbs] > 0.7 * [WalkSpeed]", "animation": { "source": "run" } },
      { "condition": "[SpeedAbs] > 0.3", "animation": { "source": "walk" } },
      { "condition": "true", "animation": { "source": "idle" } }
    ]
  }
}
```

Each state track contains:
- `layer`: which layer it controls
- `rules`: an ordered list of rules evaluated top-to-bottom; the first matching rule takes effect

### StateTrackConfig Fields

| Field | Type | Description |
|-------|------|-------------|
| `layer` | string | Name of the layer this track controls |
| `rules` | array | Ordered list of state rules |

### StateRuleConfig Fields

| Field | Type | Description |
|-------|------|-------------|
| `condition` | string | NCalc boolean expression |
| `animation` | object or null | AnimationReference when matched. `null` disables the layer |

### Condition Expressions

Conditions use NCalc expression syntax. Rules are evaluated in list order; the first rule that evaluates to `true` takes effect.

#### Referencing Parameters

Use `[ParameterName]` to reference values from AnimationParameters:

```
[SpeedAbs] > 0.3
[DeathPhase] > 0
[Gait] == 'Run'
[ButtFactor] > 0
```

#### Built-in Conditions

| Condition | Description |
|-----------|-------------|
| `IsDead` | Whether the creature is dead |
| `true` | Always matches; use as a default/fallback rule |

#### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `>`, `<`, `>=`, `<=` | Numeric comparison | `[SpeedAbs] > 0.3` |
| `==`, `!=` | Equality comparison | `[Gait] == 'Run'` |
| `and`, `or` | Logical AND / OR | `[A] > 0 and [B] < 1` |
| `not` | Logical NOT | `not IsDead` |

#### Overriding Animation Properties in Rules

The `animation` object in a rule can override properties from the animation definition:

```json
{
  "condition": "[SpeedAbs] > [WalkSpeed] * 0.7",
  "animation": {
    "source": "run",
    "speed": "[SpeedAbs] * 0.6"
  }
}
```

### Disabling a Layer

Setting `animation` to `null` disables the entire layer under the specified condition:

```json
{
  "condition": "[DeathPhase] > 0",
  "animation": null
}
```

### Multiple State Track Example

```json
"states": {
  "gait": {
    "layer": "Base",
    "rules": [
      { "condition": "IsDead", "animation": null },
      { "condition": "[SpeedAbs] > 0.7 * [WalkSpeed]", "animation": { "source": "run" } },
      { "condition": "[SpeedAbs] > 0.3", "animation": { "source": "walk" } },
      { "condition": "true", "animation": { "source": "idle" } }
    ]
  },
  "activity": {
    "layer": "UpperBodyAction",
    "rules": [
      { "condition": "[ButtFactor] > 0", "animation": { "source": "bite", "speed": "[ButtFactor]" } },
      { "condition": "[Activity] == 'Feed'", "animation": { "source": "fetch" } },
      { "condition": "true", "animation": null }
    ]
  },
  "death": {
    "layer": "Death",
    "rules": [
      { "condition": "[Death]", "animation": { "source": "death" } }
    ]
  }
}
```

---

## 6. Drivers

Drivers are bone transform sources that replace animation clips. Rather than keyframe data, they compute bone poses algorithmically.

> **Important**: Drivers exist primarily for compatibility with the C#-hardcoded animations used by `.dae` models in the base game. For glTF models, **using keyframe animations from the model file directly is recommended** over drivers. Drivers are only recommended in these cases:
> - **LookAt driver**: when you need a bone (e.g. head) to track a target direction in real time
> - **Death driver**: when you need a consistent procedural death physics effect

### Configuring a Driver on a Layer

Assign a driver to a layer inside `layers`:

```json
"layers": {
  "Head": {
    "bones": ["b_Head_05"],
    "driver": {
      "type": "LookAt",
      "properties": {
        "TargetBoneName": "b_Head_05",
        "MaxAngleX": 40,
        "MaxAngleY": 15,
        "PitchAxis": "Z",
        "YawAxis": "Y"
      }
    }
  }
}
```

### Using a Driver in a State Rule

Reference a driver in a state rule via the `driver:` prefix:

```json
{
  "condition": "true",
  "animation": { "source": "driver:LookAt" }
}
```

### Built-in Engine Drivers

#### LookAt Driver

Orients a specified bone toward a target direction. Commonly used for head tracking.

| Property | Default | Description |
|----------|---------|-------------|
| `TargetBoneName` | `"Head"` | Target bone name |
| `LookAngleXParam` | `"LookAngleX"` | Pitch angle parameter name |
| `LookAngleYParam` | `"LookAngleY"` | Yaw angle parameter name |
| `MaxAngleX` | `65` | Maximum pitch angle (degrees) |
| `MaxAngleY` | `55` | Maximum yaw angle (degrees) |
| `PitchAxis` | `"X"` | Pitch rotation axis |
| `YawAxis` | `"Z"` | Yaw rotation axis |
| `InvertPitch` | `false` | Whether to invert pitch |
| `InvertYaw` | `false` | Whether to invert yaw |

`LookAngleX` and `LookAngleY` must be provided in `parameters` or set via C# code.

#### Death Driver

Computes a collapse animation based on a death phase parameter.

| Property | Default | Description |
|----------|---------|-------------|
| `RootBoneName` | null (uses model root bone) | Root bone name |
| `DeathPhaseParam` | `"DeathPhase"` | Death phase parameter name |
| `BodyHeightParam` | `"BodyHeight"` | Body height parameter name |
| `BodyRightParam` | `"BodyRight"` | Body right-direction parameter name |
| `DeathCauseOffsetParam` | `"DeathCauseOffset"` | Death cause offset parameter name |
| `RollAngle` | `90` | Roll angle when falling (degrees) |
| `PitchAngle` | `0` | Pitch angle when falling (degrees) |
| `BodyDrop` | `0` | Drop amount as a fraction of body height |
| `AutoRollDirection` | `true` | Whether to auto-roll toward the damage source |

#### Expression Driver

Drives bone transforms via expressions.

```json
{
  "type": "Expression",
  "properties": {
    "boneConfigs": [
      {
        "boneName": "Head",
        "rotationX": "[LookAngleX]",
        "rotationY": "[LookAngleY]"
      }
    ]
  }
}
```

Per-bone config options:
- `boneName` — bone name
- `positionX/Y/Z` — position expression (default `"0"`)
- `rotationX/Y/Z` — rotation expression in degrees (default `"0"`)
- `scaleX/Y/Z` — scale expression (default `"1"`)

---

## 7. Animation Events

Animation events allow C# callbacks to be triggered at specific points during playback.

### Configuration Format

Add events to the `events` array inside an animation definition:

```json
{
  "source": "Bite",
  "loop": true,
  "events": [
    { "time": 0.577, "name": "AttackHit" },
    { "time": 0.3, "name": "PlaySound", "data": "Audio/Creatures/Bite" }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `time` | float | Trigger time, normalized 0–1 (0 = animation start, 1 = animation end) |
| `name` | string | Event name, used to identify it in C# |
| `data` | string | Optional additional data |

### OnComplete Actions

Non-looping animations can automatically trigger a state change or event when they finish:

```json
{
  "source": "Bark",
  "loop": false,
  "onComplete": {
    "type": "setState",
    "state": "Activity",
    "value": "None"
  }
}
```

Or fire an event:

```json
{
  "source": "Bark",
  "loop": false,
  "onComplete": {
    "type": "trigger",
    "name": "BarkComplete",
    "data": null
  }
}
```

| onComplete Field | Description |
|------------------|-------------|
| `type` | `"setState"` — set a state track value; `"trigger"` — fire an event |
| `state` | Target state track name (`setState` only) |
| `value` | Value to set (`setState` only) |
| `name` | Event name (`trigger` only) |
| `data` | Event additional data (`trigger` only) |

---

## 8. Root Motion Configuration

Root Motion allows animations to drive a creature's actual movement. The most common use case is animation-driven jumping.

### Configuration Format

Add a `rootMotion` config inside an animation definition:

```json
{
  "source": "Jump",
  "loop": false,
  "rootMotion": {
    "sourceBone": "Hips",
    "translation": {
      "mode": "AddImpulse",
      "impulsePhase": 0.182,
      "impulseMethod": "Peak",
      "velocityMask": [0, 1, 0],
      "impulseScale": 2.1,
      "maxImpulse": 15
    }
  }
}
```

### RootMotionConfig Fields

| Field | Description |
|-------|-------------|
| `sourceBone` | Bone to extract motion from (null = auto-detect) |
| `translation` | Translation configuration |
| `scale` | Scale configuration (optional) |

### TranslationMode

| Mode | Description |
|------|-------------|
| `"None"` | Do not use root motion (default) |
| `"Blend"` | Blend animation displacement into the creature's velocity |
| `"AddImpulse"` | Apply a velocity impulse at a specified animation phase (suited for jumping) |
| `"Override"` | Animation displacement directly overrides the creature's position |

### TranslationConfig Fields

| Field | Default | Description |
|-------|---------|-------------|
| `mode` | `"None"` | Translation mode |
| `blendMethod` | `"SmoothDamp"` | Blend method for Blend mode: `"SmoothDamp"`, `"WeightedAverage"`, `"SpringDamper"` |
| `smoothTime` | `0.3` | SmoothDamp smoothing time |
| `impulsePhase` | `-1` | Animation phase at which to apply the impulse (-1 = auto) |
| `impulseMethod` | `"Average"` | Impulse calculation method: `"Average"`, `"Peak"`, `"Weighted"` |
| `impulseScale` | `1.0` | Impulse scale factor |
| `velocityMask` | `[1, 1, 1]` | Per-axis influence (0 = unaffected, 1 = fully affected) |
| `maxSpeed` | `20` | Maximum speed cap |
| `maxImpulse` | `10` | Maximum impulse cap |

For detailed root motion usage, see [Advanced Animation.md](Advanced Animation.md).

---

## 9. Expression Overview

Several configuration fields support expression strings. Expressions use NCalc syntax.

### When Expressions Are Evaluated

A value is parsed as an expression when it is a string containing `[ParameterName]`, or when it begins with the `expr:` prefix:

```json
"speed": "[SpeedAbs] * 0.65"           // expression
"speed": 1.0                           // static value
"condition": "[SpeedAbs] > 0.3"        // condition expression
```

### Parameter References

`[ParameterName]` references a value from AnimationParameters. Commonly used parameters:

| Parameter | Type | Source |
|-----------|------|--------|
| `SpeedAbs` | float | Creature's current absolute movement speed (synced via C# code) |
| `WalkSpeed` | float | Creature's configured walk speed |
| `DeathPhase` | float | Death phase 0–1 |
| `LookAngleX` | float | Pitch angle |
| `LookAngleY` | float | Yaw angle |

Parameters are set on the AnimationController via the C# `SyncAnimationParameters()` method. `base.SyncAnimationParameters()` already syncs many built-in parameters (SpeedAbs, DeathPhase, IsDead, WalkSpeed, LookAngleX/Y, etc.); mods only need to add custom parameters.

### Custom Functions

Functions registered by the animation system:

| Function | Description |
|----------|-------------|
| `lerp(a, b, t)` | Linear interpolation |
| `smoothstep(t)` | Smooth step function |
| `clamp(v, min, max)` | Clamp a value |
| `degtorad(v)` | Degrees to radians |
| `radtodeg(v)` | Radians to degrees |
| `pi` | Pi constant (zero-argument function) |

Math functions natively provided by the NCalc engine are also available: `abs`, `min`, `max`, `sin`, `cos`, `sqrt`.

For more expression details, see [Advanced Animation.md](Advanced Animation.md).

---

## 10. JSON Inheritance (extends)

Animation configs support an `extends` property for config file inheritance. This is useful when creating creature variants within a family.

### Basic Usage

```json
{
  "extends": "Animations/BaseFox",
  "animations": {
    "idle": {
      "source": "CustomIdle"
    }
  }
}
```

This config inherits everything from `Animations/BaseFox.json` and overrides only the `idle` animation.

### Merge Rules

- **Objects**: deep merge (child properties are merged recursively)
- **Arrays**: replaced in their entirety by default

### Array Operators

When you need to modify rather than replace an array, use special operators:

| Operator | Description |
|----------|-------------|
| `$replace` | Replace the entire array (default behavior) |
| `$remove` | Remove matching elements from the array |
| `$prepend` | Insert elements at the front of the array |
| `$append` | Add elements at the end of the array |

### Chained Inheritance

Configs can inherit across multiple levels; the system automatically detects circular references.

---

## 11. Complete Examples

### Minimal Four-Legged Creature Config

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
        { "condition": "IsDead", "animation": null },
        { "condition": "[SpeedAbs] > 0.7 * [WalkSpeed]", "animation": { "source": "run" } },
        { "condition": "[SpeedAbs] > 0.3", "animation": { "source": "walk" } },
        { "condition": "true", "animation": { "source": "idle" } }
      ]
    },
    "death": {
      "layer": "Death",
      "rules": [
        { "condition": "[DeathPhase] > 0", "animation": { "source": "driver:Death" } },
        { "condition": "true", "animation": null }
      ]
    }
  },

  "parameters": {
    "WalkSpeed": 1.0,
    "DeathSpeed": 3.0,
    "LookAngleX": 0.0,
    "LookAngleY": 0.0
  }
}
```

### Advanced Creature Config (Custom Template, Root Motion, Events)

Custom template (`Assets/AnimationTemplates/CustomFox.template.json`):

```json
{
  "name": "CustomFox",
  "layers": {
    "Base": { "index": 0, "blendMode": "Override" },
    "UpperBodyAction": { "index": 1, "blendMode": "Override" },
    "Death": { "index": 2, "blendMode": "Override" }
  },
  "stateTracks": {
    "Gait": {
      "type": "Enum",
      "defaultValue": "Idle",
      "enumValues": ["Idle", "Walk", "Run", "Jump", "Fall", "Sit"]
    },
    "Activity": {
      "type": "Enum",
      "defaultValue": "None",
      "enumValues": ["None", "Feed", "Attack"]
    },
    "Death": {
      "type": "Bool",
      "defaultValue": false
    }
  }
}
```

Animation config (`Assets/Animations/CustomFox.json`):

```json
{
  "template": "AnimationTemplates/CustomFox",
  "rootBoneRotation": 180,
  "modelScale": 0.6,

  "animations": {
    "idle": { "source": "Idle", "loop": true, "blendDuration": 0.3 },
    "walk": { "source": "Walk", "loop": true, "blendDuration": 0.25 },
    "run": { "source": "Run", "loop": true, "blendDuration": 0.25 },
    "jump": {
      "source": "Jump",
      "loop": false,
      "blendDuration": 0.2,
      "rootMotion": {
        "sourceBone": "Hips",
        "translation": {
          "mode": "AddImpulse",
          "impulsePhase": 0.182,
          "impulseMethod": "Peak",
          "velocityMask": [0, 1, 0],
          "impulseScale": 2.1
        }
      }
    },
    "bite": {
      "source": "Bite",
      "loop": true,
      "blendDuration": 0.2,
      "events": [
        { "time": 0.577, "name": "AttackHit" }
      ]
    },
    "death": {
      "source": "Death",
      "loop": false,
      "preservePose": true,
      "blendDuration": 0.3
    }
  },

  "states": {
    "gait": {
      "layer": "Base",
      "rules": [
        { "condition": "[Gait] == 'Jump'", "animation": { "source": "jump" } },
        { "condition": "[Gait] == 'Fall'", "animation": { "source": "fall" } },
        { "condition": "[SpeedAbs] > [WalkSpeed] * 0.7", "animation": { "source": "run", "speed": "[SpeedAbs] * 0.6" } },
        { "condition": "[SpeedAbs] > 0.2", "animation": { "source": "walk", "speed": "[SpeedAbs] * 0.65" } },
        { "condition": "true", "animation": { "source": "idle" } }
      ]
    },
    "activity": {
      "layer": "UpperBodyAction",
      "rules": [
        { "condition": "[ButtFactor] > 0", "animation": { "source": "bite", "speed": "[ButtFactor]" } },
        { "condition": "true", "animation": null }
      ]
    },
    "death": {
      "layer": "Death",
      "rules": [
        { "condition": "[Death]", "animation": { "source": "death" } }
      ]
    }
  },

  "parameters": {
    "WalkSpeed": 1.0,
    "SpeedAbs": 0.0,
    "ButtFactor": 0.0
  }
}
```
