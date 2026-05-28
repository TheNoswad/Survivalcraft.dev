---
icon: lucide/rocket
---

# Home

Survivalcraft has a small but dedicated community of players who want to push the game beyond its defaults with custom textures, new mechanics, multiplayer tooling, and more. However, the game ships with no official modding documentation, leaving would-be modders to reverse-engineer everything from scratch.

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

## The Community
<div class="grid cards" markdown>

-   :fontawesome-brands-wordpress:{ .lg .middle } __Read the Official Blog__

    ---

    Stay up to date with development news and announcements on the official __Survivalcraft Blog__ by Kaalus.

    [:octicons-arrow-right-24: Read the Blog](https://kaalus.wordpress.com/)

-   :fontawesome-brands-discord:{ .lg .middle } __Join the SCC Discord__

    ---

    Chat, share content, and connect with the Survivalcraft community on the __SCC Discord Server__.

    [:octicons-arrow-right-24: Join the Discord](https://discord.gg/VqRf37BYkw)

-   :fontawesome-brands-youtube:{ .lg .middle } __Watch the Official YouTube Channel__

    ---

    Watch official Survivalcraft videos and development updates straight from the developer.

    [:octicons-arrow-right-24: Watch on YouTube](https://www.youtube.com/user/kaalus)

-   :fontawesome-brands-x-twitter:{ .lg .middle } __Follow the Developer on X__

    ---

    Keep up with the latest thoughts and updates from the developer on X (formerly Twitter).

    [:octicons-arrow-right-24: Follow on X](http://twitter.com/#!/CandyRufusGames)

-   :fontawesome-solid-comment:{ .lg .middle } __Visit the Official Forum__

    ---

    Browse and participate in discussions on the long-running official Survivalcraft forum, hosted on Tapatalk.

    [:octicons-arrow-right-24: Official Forum](https://www.tapatalk.com/groups/survivalcraft/)

-   :fontawesome-brands-discourse:{ .lg .middle } __Visit the Unofficial Forum__

    ---

    Join the newer community-run forum on Discourse. A modern space for discussion, chat, and content sharing. :sparkles:

    [:octicons-arrow-right-24: Unofficial Forum](https://survivalcraft.forum)

-   :simple-fandom:{ .lg .middle } __Read and Contribute to the Wiki__

    ---

    Look up game mechanics, items, and lore. Or help expand the community knowledge base on the Fandom Wiki.

    [:octicons-arrow-right-24: Fandom Wiki](https://survivalcraftgame.fandom.com/wiki/SurvivalCraft_Wiki)

-   :fontawesome-brands-reddit:{ .lg .middle } __Visit or Join the Subreddit__

    ---

    Browse screenshots, discussion, and community posts on the unofficial Survivalcraft subreddit.

    [:octicons-arrow-right-24: Subreddit](https://www.reddit.com/r/SurvivalCraft/)

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
* [x] API Development environment setup
* [ ] Decompile development environment setup
