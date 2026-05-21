---
icon: lucide/pocket-knife
---

# Decompiling Survivalcraft
Decompiling Survivalcraft is a relatively simple process. This process involves obtaining the game from official sources and using .NET disassembly tools to yield a .NET solution. After patching any missing or platform specific code, the solution can be compiled successfully and any modifications can be made.

## Selecting a version to decompile
The version that you choose to decompile might depends on the operating system that you would like to run the game on. The Microsoft store version of the game uses the DirectX xxx(?) graphics backend while the Android release of the game uses the OpenGL (version?) graphics backend.

## Download ILSpy
Install ILSpy only from the [official Github repo](https://github.com/icsharpcode/ILSpy). 

## Install ILSpy
First, you should install the .NET version required by ILSpy. Currently [.NET 10.0](https://dotnet.microsoft.com/en-us/download/dotnet/10.0) is required. The self-contained build bundles its own .NET runtime and installing .NET should not be necessary.

### Windows
Run the installer and install the program like any other. If using the self contained or binaries build, extract to a safe location and run `ILSpy.exe` to run the software.

### Linux
In the past AvaloniaILSpy would have been used on linux, but that version of ILSpy has been archived as no maintainers stepped forward. We will run ILSpy using Wine.

1. Download and extract the ILSpy selfcontained release.
2. Install [Bottles](https://usebottles.com/)
3. Create a bottle for ILSpy. Any runner should work fine.
4. Copy or move the ILSpy selfcontained release into the `C:/` drive of the bottle. The bottle quick actions should let you quickly open this directory by clicking "Browse C:/ drive"
6. Add a shortcut to `ILSpy.exe` for ease of use.
7. Disable DXVK and VKD3D for the shortcut, or the entire bottle.
8. Launch the program

## Extract the game files
### Microsoft Store
Download the latest release using a website such as [DanStore](https://danstore-ms.vercel.app/), [store.rg](https://store.rg-adguard.net/), or [StoreWeb](https://msft-store.tplant.com.au/).

<div style="text-align: center;" markdown>
## Official game links for Microsoft Store
</div>

<div class="grid cards" markdown>
- :material-numeric-1: [__Survivalcraft__](https://apps.microsoft.com/detail/9wzdncrfhvnl) store page
- :material-numeric-2: [__Survivalcraft 2__](https://apps.microsoft.com/detail/9phc48p58nb2) store page
</div>

Then, extract the app package. Change the file extension to `.zip` if necessary.

### Android
Todo

### iOS
Todo

## Decompile the Game
In ILSpy, select File/Open and navigate to wherever you extracted the game binaries. Select and open both `Survivalcraft.exe` and all accompanying `.DLL` files.

Next, select assemblies `Survivalcraft`, `Engine`, and `EntitySystem` from the assemblies pane. Then right click and select "Save Code"

ILSpy will then decompile and save all code and resources, including a Visual Studio solution.

## Patching
