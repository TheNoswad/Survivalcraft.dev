# Decompiling Survivalcraft
Decompiling Survivalcraft is a relatively simple process. This process involves obtaining the game from official sources and using .NET disassembly tools to yield a .NET solution. After patching any missing or platform specific code, the solution can be compiled successfully and any modifications can be made.

## Selecting a version to decompile
The version that you choose to decompile might depends on the operating system that you would like to run the game on. The Microsoft store version of the game uses the DirectX xxx(?) graphics backend while the Android release of the game uses the OpenGL (version?) graphics backend.

## Install ILSpy
Install ILSpy only from the [official Github repo](https://github.com/icsharpcode/ILSpy). 

## Extract the game files
### Microsoft Store
1. Download the latest release using a website such as [DanStore](https://danstore-ms.vercel.app/), [store.rg](https://store.rg-adguard.net/), or [StoreWeb](https://msft-store.tplant.com.au/).

Official game links for Microsoft Store

[Survivalcraft](https://apps.microsoft.com/detail/9wzdncrfhvnl)

[Survivalcraft 2](https://apps.microsoft.com/detail/9phc48p58nb2)

2. Extract the app package. Change the file extension to `.zip` if necessary.