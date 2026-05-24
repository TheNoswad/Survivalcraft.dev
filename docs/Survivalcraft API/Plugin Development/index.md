---
icon: lucide/plug
---

!!! info inline end "Attribution"

    This text is derived and translated to English from [`docs/HowToDevelopAMod.md`](https://gitee.com/SC-SPM/SurvivalcraftApi/blob/SCAPI1.9/docs/HowToDevelopAMod.md) from the Survivalcraft API Gitee Repository.

## Steps

1. Run `dotnet --list-sdks` to confirm whether the .NET 10 SDK is installed. If not, complete the installation yourself, or guide the user to install it — see the [.NET 10 official site](https://dotnet.microsoft.com/download/dotnet/10.0).
2. Run `git -v` to confirm whether Git is installed. If not, complete the installation yourself, or guide the user to install it — see the [Git official site](https://git-scm.com/).
3. Check whether your working directory contains the Survivalcraft Plugin Edition source code, i.e. [SC-SPM/SurvivalcraftApi](https://gitee.com/SC-SPM/SurvivalcraftApi/). If not, complete the `git clone` yourself.
4. Check whether your working directory contains the example mod source code, i.e. [SC-SPM/SurvivalcraftTemplateModForAPI](https://gitee.com/SC-SPM/SurvivalcraftTemplateModForAPI). If not, complete the `git clone` yourself.
5. Confirm whether the mod project you are developing is a **new/empty project**.
   - **If yes (new/empty project):** Read the `AGENTS.md` inside the example mod and follow its guidance to create the mod.
   - **If no (existing project):** Read [Development Environment Setup.md](Development Environment Setup.md), then [Architecture.md](Architecture.md) as a development reference.

> If you need to contribute to or reference the main game source repository directly, read [Architecture.md](Architecture.md).

> When helping the user install the .NET 10 SDK, you may encounter situations where a slow network or device causes the session to return mid-process. In that case, check the logs to see if an error occurred during installation. If there are no errors, it simply means installation is still in progress — remind the user to notify you once it has finished. The same applies when cloning the source repositories mentioned in steps 3 and 4.
