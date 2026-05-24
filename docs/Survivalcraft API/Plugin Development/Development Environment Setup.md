# Survivalcraft API Development Documentation
!!! info "Attribution"

    This text is derived and translated to English from [`docs/Development.md`](https://gitee.com/SC-SPM/SurvivalcraftApi/blob/SCAPI1.9/docs/Development.md) from the Survivalcraft API Gitee Repository.

## Getting Started

1. If the .NET 10 SDK is not installed, download it from the [official website](https://dotnet.microsoft.com/download/dotnet/10.0)
2. If this repository is not local, clone it using Git:

    ```bat
    git clone https://gitee.com/SC-SPM/SurvivalcraftApi.git
    ```

   > Don't have Git? [Download it here](https://git-scm.com/downloads)

3. Enter the repository and open `SurvivalCraftApi.sln` inside the `SurvivalcraftApi` directory using [Visual Studio](https://visualstudio.microsoft.com/) or [Rider](https://www.jetbrains.com/rider/)
4. If you are only debugging on Windows, right-click any folder other than Windows and select **Unload Project**. Then right-click the `Survivalcraft.Windows` project inside the Windows folder, select **Build Selected Project**, and start debugging

## Building the Game

**Windows / Linux:**
Build the corresponding `Survivalcraft.Windows` or `Survivalcraft.Linux` project directly.

**Android:**
To generate an APK installer for Android, right-click the `Survivalcraft.Android` project, select **Load Project**, then click **Archive for Publishing**, and follow the prompts.

> If an error appears saying a required feature is not installed, follow the prompts to install it. The same applies to other platforms below.

**Web:**
Build the `Survivalcraft.Browser` project directly.
To produce a final web build for release, run `.\scripts\PublishSurvivalcraftBrowser.bat` from the solution root.

**NuGet Package (nupkg):**
Run `.\scripts\PackNugetPackages.bat` from the solution root.

## Mod Developer Reference

1. First, copy the `nuget.config` file from the root of this repository to your solution directory (the same level as your `.sln` file).

2. There are two standard ways to add the reference package (nupkg) — choose whichever you prefer:

    * **Recommended:** Run the following command in your solution directory:

    ```bat
    dotnet add package SurvivalcraftAPI.Survivalcraft
    ```

    * Or manually add the following to your `.csproj` file inside `<Project>...</Project>` (the version number below may not be the latest):

    ```xml
    <ItemGroup>
      <PackageReference Include="SurvivalcraftAPI.Survivalcraft" Version="1.9.1.3"/>
    </ItemGroup>
    ```

3. **Methods other than the above are not recommended.**
4. If your network connection is too unreliable to download the nupkg automatically, go to the [releases page](https://gitee.com/SC-SPM/SurvivalcraftApi/releases/latest) and download the archive prefixed with `[Nupkgs]` and suffixed with `.7z`. Extract all `.nupkg` files to a directory of your choice, then follow the [Microsoft official guide](https://learn.microsoft.com/en-us/nuget/hosting-packages/local-feeds) to add them manually.
5. There is also a more involved method: after obtaining the nupkg files as described above (or by other means), extract them one by one, locate `Engine.dll`, `EntitySystem.dll`, and `Survivalcraft.dll`, place them wherever you like, and add the following to your `.csproj` file inside `<Project>...</Project>` (most IDEs support this through a GUI, achieving the same result):

```xml
<ItemGroup>
  <Reference Include="Engine" HintPath="(Enter the file path to Engine.dll here, without brackets)" />
  <Reference Include="EntitySystem" HintPath="(File path to EntitySystem.dll, without brackets)" />
  <Reference Include="Survivalcraft" HintPath="(File path to Survivalcraft.dll, without brackets)" />
</ItemGroup>
```

## AI Agent Development Example

```
Please read this document and help me create a Survivalcraft plugin-style mod
that displays the current in-game weather and how long until it becomes sunny or rainy.
https://gitee.com/SC-SPM/SurvivalcraftApi/raw/SCAPI1.9/docs/HowToDevelopAMod.md
```

> Compatible AI Agents include: [OpenClaw](https://openclaw.ai/), [Claude Code](https://claude.com/product/claude-code), [OpenCode](https://opencode.ai/), [CodeBuddy](https://www.codebuddy.cn/), and others.

## Development Guidelines

### For Plugin Developers
* New public methods should be designed with ease of use in mind
* Avoid modifying or removing existing public methods, as this breaks compatibility with older mods
* Avoid introducing any behavior that is inconsistent with the vanilla game
* Before committing, at minimum build the game, launch it, and enter a save file as a basic smoke test
* When modifying code under `Engine/`, be mindful of cross-platform compatibility; use `#if` conditional compilation to handle platform differences
* Do not use your IDE's project properties editor — manually edit `.csproj` files and the `.props` files under `build/`

### For Mod Developers
* Prefer adding new subsystems and components as the primary way to introduce new features
* Extend the `ModLoader` class and use hook methods to integrate efficiently with game logic
* Use the built-in `HarmonyX` library to patch game methods
* Avoid overriding existing classes, as this causes mod incompatibility
* If possible, run the game through your IDE debugger with "break on any exception" enabled — this helps catch errors in your mod that would otherwise not appear in logs
* If you have the bandwidth, consider providing localized strings (your native language + English)

> Next, consider reading the architecture documentation: [Architecture.md](Architecture.md)
> Mod developers are encouraged to start new projects from the template mod: [SC-SPM/SurvivalcraftTemplateModForAPI](https://gitee.com/SC-SPM/SurvivalcraftTemplateModForAPI)

## Code Style

### Formatting (from .editorconfig)

* **Indentation:** 4 spaces, no tabs
* **Line endings:** CRLF
* **Braces:** Same-line placement (K&R style)
* **Max line width:** 150 characters
* **Line wrapping:** Arrays or parameter lists with more than 6 elements should be broken across multiple lines
* **Encoding:** UTF-8

### Naming Conventions

| Element | Convention | Example |
| --- | --- | --- |
| Properties / Public static fields / Constants | PascalCase | `float LastFrameTime { get; set; }` |
| Instance fields | `m_` prefix + camelCase | `float m_frameBeginTime` |
| Methods | PascalCase | `public void Update()` |
| Parameters / Local variables | camelCase | `float deltaTime` |

### C# Preferences

* **Avoid `var`:** Use explicit types, e.g. `string path = ...` rather than `var path = ...`
* **Nullable types disabled:** The project uses `<Nullable>disable</Nullable>`
* **Unsafe code allowed:** `<AllowUnsafeBlocks>true</AllowUnsafeBlocks>`
* **Language version:** preview (latest C# features are permitted)

## Error Handling

* Use `Log.Error()` and `Log.Information()` for logging
* Exceptions in mods should be caught and logged — they must not crash the game
* Use `try/catch` around file operations and calls to external code

```csharp
try {
    // operation
}
catch (Exception e) {
    Log.Error($"Failed to load: {e}");
}
```
