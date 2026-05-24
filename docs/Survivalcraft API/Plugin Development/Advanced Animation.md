!!! info inline end "Attribution"

    This text is derived and translated to English from [`docs/AnimationAdvancedTopics.md`](https://gitee.com/SC-SPM/SurvivalcraftApi/blob/SCAPI1.9/docs/AnimationAdvancedTopics.md) from the Survivalcraft API Gitee Repository.

This document covers advanced API usage for the animation system, aimed at mod developers who need deep customization of animation behavior in C# code.

## Table of Contents

1. [AnimationController API](#1-animationcontroller-api)
2. [Parameter System](#2-parameter-system)
3. [Manual Animation Control](#3-manual-animation-control)
4. [Animation Event Handling](#4-animation-event-handling)
5. [IK System](#5-ik-system)
6. [Root Motion](#6-root-motion)
7. [Expression System](#7-expression-system)
8. [Animation Mirroring](#8-animation-mirroring)
9. [Blend Spaces](#9-blend-spaces)
10. [Custom Drivers](#10-custom-drivers)

---

## 1. AnimationController API

`AnimationController` is the core controller of the animation system. It is automatically created when `ComponentModel.SetModel()` is called, triggered by the `AnimationConfigPath` or `AnimationTemplateName` parameter.

### Accessing the Controller

Access it via a property inside a custom `ComponentCreatureModel` subclass:

```csharp
var controller = AnimationController;
if (controller != null)
{
    // Use the API ...
}
```

### Core API Overview

| API | Description |
|-----|-------------|
| `Update(float deltaTime)` | Per-frame update (called automatically by the system) |
| `ComputeBoneTransforms(Matrix?[] boneTransforms)` | Computes bone transforms (called automatically by the system) |
| `SetState(string trackName, object value)` | Sets a state track value |
| `GetState(string trackName)` | Gets a state track value |
| `PlayAnimation(...)` | Manually plays an animation |
| `ReleaseManualControl(...)` | Releases manual control |
| `RegisterAndBuildIKChain(...)` | Registers an IK chain |
| `SetIKAim(...)`, `SetIKTarget(...)` | Sets IK targets |
| `SetRootMotionConfig(RootMotionConfig)` | Sets the Root Motion configuration |
| `SetDriver(string layerName, IAnimationDriver driver)` | Sets a layer driver |

### Lifecycle

```
Per frame:
  1. SyncAnimationParameters()  → Developer syncs parameters
  2. AnimationController.Update(deltaTime)
     - Evaluates state rules
     - Updates each animation layer
     - Applies Root Motion
     - Checks completion events
  3. AnimationController.ComputeBoneTransforms(boneTransforms)
     - Layer blending
     - IK solving
     - Morph Targets
```

### Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `Model` | `Model` | The target model |
| `Template` | `AnimationTemplate` | The template in use |
| `Parameters` | `AnimationParameters` | Parameter container |
| `Layers` | `AnimationLayer[]` | Layer array |
| `ExpressionEvaluator` | `ExpressionEvaluator` | Expression evaluator |
| `IKSolver` | `IKSolver` | IK solver (lazily initialized) |
| `RootBoneRotation` | float | Root bone rotation correction (radians) |
| `ModelScale` | float | Model scale |
| `Velocity` | Vector3? | Velocity vector (modified by Root Motion) |
| `HasRootMotion` | bool | Whether a Root Motion config is present |

---

## 2. Parameter System

`AnimationParameters` is a typed parameter store with dirty-flag tracking. Parameters are the data source for expressions and state rules.

### Setting Parameters

```csharp
var p = controller.Parameters;

p.SetFloat("SpeedAbs", speedValue);
p.SetBool("Death", isDead);
p.SetVector3("LookDirection", direction);
p.SetString("GaitState", "Run");

// Generic set (auto-dispatches by type)
p.SetParameter("CustomValue", 3.14f);
```

### Reading Parameters

```csharp
float speed = p.GetFloat("SpeedAbs");        // Returns 0 if not found
bool dead = p.GetBool("Death");              // Returns false if not found
Vector3 dir = p.GetVector3("LookDirection"); // Returns Zero if not found

// Safe read
if (p.TryGetFloat("SpeedAbs", out float speed)) { ... }
```

### Dirty Flags

```csharp
p.IsDirty;      // Whether any parameter was modified this frame
p.ClearDirty(); // Clear the dirty flag
p.SetDirty();   // Force mark as dirty
```

### Common Parameter Names

| Parameter Name | Type | Description | Source |
|----------------|------|-------------|--------|
| `SpeedAbs` | float | Absolute movement speed | `SyncAnimationParameters()` |
| `WalkSpeed` | float | Walk speed setting | Config file parameters |
| `DeathPhase` | float | Death phase 0–1 | `SyncAnimationParameters()` |
| `LookAngleX` | float | Pitch angle (degrees) | `SyncAnimationParameters()` |
| `LookAngleY` | float | Yaw angle (degrees) | `SyncAnimationParameters()` |
| `BodyHeight` | float | Body height | `SyncAnimationParameters()` |
| `BodyRight` | float | Body right direction | `SyncAnimationParameters()` |

Parameter names have no hard-coded restrictions — config files and C# code communicate by using the same parameter names.

---

## 3. Manual Animation Control

Manual control allows temporarily bypassing state rules to directly specify which animation plays. Useful for attacks, special abilities, and other scenarios requiring precise control.

### PlayAnimation

Forces an animation to play on a specified layer, putting that layer into manual control mode and skipping state rule evaluation:

```csharp
// Play the bite animation on the UpperBodyAction layer
bool success = controller.PlayAnimation(
    layerName: "UpperBodyAction",
    animationNameOrAlias: "bite",
    loop: true,
    blendDuration: 0.3f
);

// Play an animation from an external file
controller.PlayExternalAnimation(
    layerName: "Base",
    filePath: "Models/Extra/SpecialAnims.glb",
    animationName: "Dance",
    loop: false,
    blendDuration: 0.5f
);
```

`animationNameOrAlias` can be:
- An alias defined in the config file's `animations` section (e.g. `"bite"`)
- An animation clip name from the model file

### ReleaseManualControl

Releases manual control and restores state-rule-driven behavior:

```csharp
// Release a specific layer
controller.ReleaseManualControl("UpperBodyAction");

// Release all layers
controller.ReleaseManualControl();
```

### IsManualControl

Check whether a layer is currently under manual control:

```csharp
if (controller.IsManualControl("UpperBodyAction"))
{
    // Currently in manual control
}
```

### Typical Pattern: Attack Animation

```csharp
// Begin attack
public void StartAttack()
{
    controller.PlayAnimation("UpperBodyAction", "bite", loop: true);
}

// Detect attack hit in animation event callback
private void HandleEvent(AnimationEvent evt)
{
    if (evt.Name == "AttackHit")
    {
        PerformDamage();
    }
}

// End attack
public void EndAttack()
{
    controller.ReleaseManualControl("UpperBodyAction");
}
```

### StopAnimation

Stops playback on a layer without releasing manual control:

```csharp
controller.StopAnimation("UpperBodyAction");
// The layer stops playing but remains in manual control mode.
// Call ReleaseManualControl to restore state rules.
```

---

## 4. Animation Event Handling

### Subscribing to Events

Subscribe to AnimationController events inside a model component:

```csharp
public override void OnModelLoaded()
{
    base.OnModelLoaded();
    if (AnimationController != null)
    {
        AnimationController.OnAnimationEvent += OnAnimationEvent;
    }
}

private void OnAnimationEvent(AnimationEvent evt)
{
    switch (evt.Name)
    {
        case "AttackHit":
            // Perform attack judgment at the time point defined in the animation
            PerformAttack();
            break;
        case "PlaySound":
            // Play a sound effect
            SubsystemAudio.PlaySound((string)evt.Parameter, ...);
            break;
        case "JumpLandRecoveryComplete":
            // Jump recovery finished
            m_jumpState = JumpState.None;
            break;
    }
}
```

### AnimationEvent Structure

| Field | Type | Description |
|-------|------|-------------|
| `Name` | string | Event name (defined in config) |
| `Time` | float | Normalized trigger time 0–1 |
| `Parameter` | object | Additional data (the `data` field in config) |

### OnComplete Automatic Actions

When a non-looping animation completes, state changes can be triggered automatically without any C# code:

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

Listening in C#:

```csharp
// onComplete type="trigger" fires through OnAnimationEvent
private void OnAnimationEvent(AnimationEvent evt)
{
    if (evt.Name == "BarkComplete") { ... }
}
```

---

## 5. IK System

IK (Inverse Kinematics) allows joint angles to be computed in reverse by specifying a target position or direction. Commonly used for head tracking, hand grabbing, and similar behaviors.

### Overview

IK runs as a post-processing step after animation blending:

```
Animation blending → IK solving → Final bone transforms
```

### Registering an IK Chain

Register an IK chain after the model has loaded:

```csharp
public override void OnModelLoaded()
{
    base.OnModelLoaded();
    if (AnimationController != null)
    {
        // Register an IK chain from the root up to the head bone, using the SingleBoneIK algorithm
        AnimationController.RegisterAndBuildIKChain(
            name: "Head",
            endBoneName: "b_Head_05",
            algorithmName: "SingleBoneIK",
            maxChainLength: 3
        );
    }
}
```

Parameter descriptions:
- `name`: IK chain name; used when setting targets later
- `endBoneName`: End bone name. The system automatically walks up from this bone to build the chain.
- `algorithmName`: Algorithm name; null uses the default. Options: `SingleBoneIK`, `TwoBoneIK`, `CCD`, `FABRIK`
- `maxChainLength`: Maximum chain length (number of bones), default 3

### Setting IK Targets

Update IK targets each frame:

```csharp
public override void SyncAnimationParameters()
{
    base.SyncAnimationParameters();

    var controller = AnimationController;
    if (controller == null) return;

    // Aim constraint: orient the head toward a target direction
    Vector3 aimDirection = CalculateAimDirection();
    float weight = CalculateIKWeight(); // 0–1, typically falls off with distance
    controller.SetIKAim("Head", aimDirection, weight);

    // Position constraint: move a bone to a target position
    // controller.SetIKTarget("Hand", targetPosition, weight);
}
```

### Clearing IK Targets

```csharp
controller.ClearIKTarget("Head");
```

### IK Target Types

| Method | Description |
|--------|-------------|
| `SetIKAim(name, direction, weight)` | Aim constraint — orients the end bone toward the specified direction |
| `SetIKTarget(name, position, weight)` | Position constraint — moves the end bone to the specified position |
| `SetIKTarget(name, IKTarget)` | Full target (position + direction + hint direction) |

### IKTarget Detailed Fields

```csharp
var target = IKTarget.CombinedTarget(
    position: targetPos,       // Target position
    aimDirection: aimDir,      // Target direction
    positionWeight: 1.0f,      // Position weight
    aimWeight: 0.8f            // Direction weight
);
target.Hint = elbowHintDir;        // Bend hint direction (e.g. elbow direction)
target.PositionSmoothTime = 0.1f;  // Position smoothing time
target.AimSmoothTime = 0.15f;      // Direction smoothing time
controller.SetIKTarget("Hand", target);
```

### IK Algorithms

| Algorithm | Chain Length | Supports Aim | Description |
|-----------|-------------|--------------|-------------|
| **SingleBoneIK** | 1–2 | Yes | Single-bone orientation rotation. Good for neck–head. |
| **TwoBoneIK** | 3 | Yes | Analytical solution (law of cosines), precise and efficient. Good for upper arm–forearm–hand. |
| **CCD** | Any | No | Cyclic Coordinate Descent, iterative solver. Good for multi-joint chains. |
| **FABRIK** | Any | No | Forward And Backward Reaching IK, good convergence. Good for long chains. |

### Joint Limits

Add rotation limits to bones in a chain:

```csharp
IKChain chain = controller.GetIKChain("Head");
chain.SetJointLimit("Neck", JointLimit.FromDegrees(
    minDeg: new Vector3(-40, -30, -20),
    maxDeg: new Vector3(40, 30, 20),
    EulerRotationOrder.YXZ
));
```

### Unreachable Strategy

Behavior when the target is outside the IK chain's reachable range:

```csharp
chain.UnreachableStrategy = UnreachableStrategy.ExtendTowardTarget;
```

| Strategy | Description |
|----------|-------------|
| `ExtendTowardTarget` | Fully extend toward the target (default) |
| `KeepCurrentPose` | Maintain the current pose unchanged |
| `UseLastValidResult` | Use the last valid result |

### Complete Example: Head Tracking

```csharp
public class ComponentFoxStareBehavior : ComponentStareBehavior
{
    private const string HeadIKChain = "Head";

    public override void OnModelLoaded()
    {
        base.OnModelLoaded();
        var controller = ComponentCreatureModel?.AnimationController;
        if (controller != null)
        {
            controller.RegisterAndBuildIKChain(
                HeadIKChain,
                "b_Head_05",
                "SingleBoneIK",
                maxChainLength: 3
            );
        }
    }

    public override void Update(float dt)
    {
        base.Update(dt);
        var controller = ComponentCreatureModel?.AnimationController;
        if (controller == null || m_stareTarget == null) return;

        // Calculate direction toward target
        Vector3 modelDir = WorldToModelDirection(m_stareTarget.Value);

        // Weight falls off with distance
        float dist = Vector3.Distance(Position, m_stareTarget.Value);
        float weight = MathHelper.Clamp(1.0f - (dist - MinRange) / (MaxRange - MinRange), 0.3f, 1.0f);

        controller.SetIKAim(HeadIKChain, modelDir, weight);
    }
}
```

---

## 6. Root Motion

Root Motion allows root bone movement baked into an animation to drive the creature's actual physical movement.

### Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `None` | Disabled (default) | — |
| `Blend` | Blends animation displacement into velocity | Smooth movement transitions |
| `AddImpulse` | Applies an impulse velocity at a specified phase | Jump takeoff |
| `Override` | Animation displacement directly overrides position | Precise path movement |

### Using Root Motion in C#

The controller needs a velocity vector reference before Root Motion takes effect:

```csharp
// Associate in OnModelLoaded
controller.Velocity = ref componentBody.Velocity;
controller.EntityRotation = componentBody.Rotation;
controller.SetCollisionBox = (size) => componentBody.BoxSize = size;
controller.DefaultCollisionSize = originalBoxSize;
```

### AddImpulse Mode Details

AddImpulse mode extracts velocity at a specified animation phase and applies it as an impulse:

1. When the animation reaches `impulsePhase`
2. The system computes the peak/average velocity near that point
3. The velocity is multiplied by `impulseScale` and filtered through `velocityMask`
4. The resulting impulse is applied to the creature's velocity vector

```json
"jump": {
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

- `impulsePhase: 0.182` — the takeoff point in the jump animation
- `velocityMask: [0, 1, 0]` — affects only the Y axis (vertical)
- `impulseMethod: "Peak"` — uses peak velocity

### RootMotionCache API

Animation movement data can be queried in C#:

```csharp
// Get the average velocity over the entire animation
Vector3 avgVel = cache.GetAverageVelocity();

// Get the average velocity over a specified phase range
Vector3 rangeVel = cache.GetAverageVelocity(0.0f, 0.3f);

// Get peak velocity
Vector3 peakVel = cache.GetPeakVelocity();

// Get total displacement
Vector3 totalDisp = cache.GetTotalTranslation();
```

### Switching Root Motion at Runtime

The Root Motion configuration can be changed at runtime:

```csharp
// Enable Blend mode
controller.SetRootMotionConfig(new RootMotionConfig
{
    SourceBone = "Hips",
    Translation = new TranslationConfig
    {
        Mode = TranslationMode.Blend,
        BlendMethod = BlendMethod.SmoothDamp,
        SmoothTime = 0.3f
    }
});

// Disable
controller.SetRootMotionConfig(null);
```

---

## 7. Expression System

The expression system is based on NCalc and is used to write dynamic logic inside config files.

### Expression Detection

Strings in the following formats are recognized as expressions:
- Strings containing `[ParameterName]`
- Strings prefixed with `expr:`

```json
"speed": "[SpeedAbs] * 0.65"   // Contains parameter reference → expression
"speed": "expr:1.0 + sin(pi)"  // expr: prefix → expression
"speed": 1.0                   // Plain number → static value
```

### Animation System Custom Functions

The following functions are registered by `AnimationExpressionFunctions`:

| Function / Constant | Description | Example |
|--------------------|-------------|---------|
| `lerp(a, b, t)` | Linear interpolation a + (b-a)*t | `lerp(0, 1, 0.5)` = 0.5 |
| `smoothstep(t)` | Smooth step t*t*(3-2t) | `smoothstep(0.5)` = 0.5 |
| `clamp(v, min, max)` | Clamp | `clamp([x], 0, 1)` |
| `degtorad(v)` | Degrees → radians | `degtorad(90)` ≈ 1.571 |
| `radtodeg(v)` | Radians → degrees | `radtodeg(3.14)` ≈ 179.9 |
| `pi` | Pi (zero-argument function call) | `pi` ≈ 3.14159 |

### NCalc Built-in Functions

The NCalc expression engine provides the following commonly used math functions out of the box, with no additional registration required:

| Function | Description |
|----------|-------------|
| `abs(v)` | Absolute value |
| `min(a, b)` / `max(a, b)` | Minimum / maximum |
| `sin(v)` / `cos(v)` | Sine / cosine |
| `sqrt(v)` | Square root |

### DynamicProperty

In C#, `DynamicProperty<T>` can handle config values that may be either static or expression-based:

```csharp
var prop = DynamicProperty<float>.FromExpression("[SpeedAbs] * 0.5");
float value = prop.GetValue(controller.Parameters, controller.ExpressionEvaluator);
```

---

## 8. Animation Mirroring

Animation mirroring allows reusing one side's animation on the other (e.g. mirroring a left-hand animation to the right hand) through bone remapping.

### Configuration

```csharp
// Set bone remapping on a ClipAnimationSource
var boneRemapping = new Dictionary<string, string>
{
    { "Hand_L", "Hand_R" },
    { "Arm_L", "Arm_R" },
    { "Leg_L", "Leg_R" }
};
```

Mirroring is configured at the `ClipAnimationSource` level via code rather than JSON config.

---

## 9. Blend Spaces

Blend Spaces allow multiple animations to be smoothly blended based on parameter values.

### 1D Blend Space

Blend between multiple animations based on a single parameter value:

```csharp
// Create programmatically
var blendSpace = new AnimationBlendSpaceDefinition
{
    Name = "Movement",
    ParameterName = "SpeedNorm",
    SyncTime = true,
    Samples = new[]
    {
        new AnimationBlendSample { Value = 0.0f, AnimationName = "Idle" },
        new AnimationBlendSample { Value = 0.5f, AnimationName = "Walk" },
        new AnimationBlendSample { Value = 1.0f, AnimationName = "Run" }
    }
};
```

When `SpeedNorm` = 0, Idle plays; at 0.5, Walk plays; at 1.0, Run plays — intermediate values blend automatically.

`SyncTime = true` (default) keeps all animations synchronized in time progress, preventing frame pops during blending.

### 2D Blend Space

Blend based on two parameter values:

```csharp
var blendSpace2D = new AnimationBlendSpaceDefinition2D
{
    ParameterNameX = "MoveX",
    ParameterNameY = "MoveY",
    Samples = new[]
    {
        new AnimationBlendSample2D { ValueX = 0,  ValueY = 0,  AnimationName = "Idle" },
        new AnimationBlendSample2D { ValueX = 1,  ValueY = 0,  AnimationName = "WalkFwd" },
        new AnimationBlendSample2D { ValueX = -1, ValueY = 0,  AnimationName = "WalkBack" },
        new AnimationBlendSample2D { ValueX = 0,  ValueY = 1,  AnimationName = "WalkLeft" },
        new AnimationBlendSample2D { ValueX = 0,  ValueY = -1, AnimationName = "WalkRight" }
    }
};
```

The 2D Blend Space uses inverse-distance weighting to blend the nearest 4 sample points.

---

## 10. Custom Drivers

A custom driver can be created by implementing the `IAnimationDriver` interface.

> **Note**: Drivers are primarily a compatibility mechanism designed for C#-hardcoded animations on the game's own `.dae` models. glTF models should prefer keyframe animations. Custom drivers are only recommended when there is a specific algorithmic need.

### IAnimationDriver Interface

```csharp
public interface IAnimationDriver
{
    string Name { get; }
    AnimationBlendMode BlendMode { get; }
    string[] TargetBones { get; }
    void Update(float deltaTime, AnimationParameters parameters);
    void SampleTransforms(Matrix?[] boneTransforms, Model model);
}
```

### Implementation Example

```csharp
public class CustomHeadBobDriver : IAnimationDriver
{
    public string Name => "CustomHeadBob";
    public AnimationBlendMode BlendMode => AnimationBlendMode.Override;
    public string[] TargetBones => ["Head"];

    private float m_time;

    public void Update(float deltaTime, AnimationParameters parameters)
    {
        m_time += deltaTime;
    }

    public void SampleTransforms(Matrix?[] boneTransforms, Model model)
    {
        ModelBone head = model.FindBone("Head");
        if (head == null) return;

        float bob = MathF.Sin(m_time * 5f) * 0.05f;
        boneTransforms[head.Index] = Matrix.CreateTranslation(0, bob, 0);
    }
}
```

### Registering a Driver

```csharp
// Register the driver type
AnimationDriverManager.Register<CustomHeadBobDriver>("CustomHeadBob");

// Use it in code
var driver = new CustomHeadBobDriver();
controller.SetDriver("Base", driver);
```

### Referencing in Config

Once registered, the driver can be referenced by name in JSON config:

```json
{
  "source": "driver:CustomHeadBob"
}
```
