---
icon: lucide/rocket
---

# Home

Survivalcraft has a small but dedicated community of players and tinkerers who want to push the game beyond its defaults with custom textures, new mechanics, multiplayer tooling, and more. However, the game ships with no official modding documentation, leaving would-be modders to reverse-engineer everything from scratch.

This site exists to change that. It will centralize what the community has learned through decompilation, experimentation, and collaboration, covering everything from plugin development with the Survivalcraft API to low-level internals like the world format, shader pipeline, and content packages.

## The Game
Survivalcraft is a 3D voxel game written in C#. Initially it used the MonoGame Game Engine, but the developer later switched to his own engine which seems similar to MonoGame. This new engine, named "Engine" uses OpenTK graphics backend on iOS and Android, and SharpDX on Windows and Windows Phone.

Modding the game typically involves either using the Survivalcraft API client, or modifying the source directly and redistributing the entire game from build artifacts.

<div class="grid cards" markdown>

-   :lucide-circle-dollar-sign:{ .lg .middle } __Buy the Game__

    ---

    Download __Survivalcraft__ from the Microsoft Store, App Store, Google Play, and Amazon.

    [:octicons-arrow-right-24: Get the Game](Get the Game.md)

-   :lucide-box:{ .lg .middle } __Download the Survivalcraft API__

    ---

    Install the __Survivalcraft API Mod Client__ from the official Gitee Repository.

    [:octicons-arrow-right-24: Get the API](Survivalcraft API/index.md)

-   :material-hammer-screwdriver:{ .lg .middle } __Decompile the Game__

    ---

    Decompile the game using ILSpy or other tools

    [:octicons-arrow-right-24: Decompiling Survivalcraft](Decompiling the Game.md)

-   :material-hammer-screwdriver:{ .lg .middle } __Browse Technical Documentation (Coming Soon)__

    ---

    Browse technical documentation for Survivalcraft

    [:octicons-arrow-right-24: Technical Documentation](Technical Documentation/index.md)

</div>

## Todo

> We have a lot of work to do. Please consider contributing to this project

* [x] Setup Survivalcraft.dev domain and static site
* [ ] Survivalcraft API plugin development guide
* [ ] Multilingual Support
* [ ] iOS and Android Decompilation guides
* [ ] Mono porting guide
* [ ] Texture and skin creation guide
* [ ] `Content.pak` documentation
* [ ] Network multiplayer documentation
* [ ] Shader development documentation
* [ ] Community content API documentation
* [ ] MOTD format documentation
* [ ] `BlocksData.txt` documentation
* [ ] `Database.xml` detailed documentation
* [ ] Crafting recipes documentation
* [ ] World format documentation
* [ ] Furniture format documentation
* [ ] Terrain generator documentation
* [ ] Development enviroment setup
