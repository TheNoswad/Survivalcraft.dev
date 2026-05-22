Survivalcraft has a small but dedicated community of players and tinkerers who want to push the game beyond its defaults with custom textures, new mechanics, multiplayer tooling, and more. However, the game ships with no official modding documentation, leaving would-be modders to reverse-engineer everything from scratch.

This site exists to change that. It will centralize what the community has learned through decompilation, experimentation, and collaboration, covering everything from plugin development with the Survivalcraft API to low-level internals like the world format, shader pipeline, and content packages.

## The Game
Survivalcraft is a 3D voxel game written in C#. Initially it used the MonoGame Game Engine, but the developer later switched to his own engine which seems similar to MonoGame. This new engine, named "Engine" uses OpenTK graphics backend on iOS and Android, and SharpDX on Windows and Windows Phone.

Modding the game typically involves either using the Survivalcraft API client, or modifying the source directly and redistributing the entire game from build artifacts.

# Running Locally

This project uses [Zensical](https://zensical.org) to build the documentation site. You can serve it locally either by installing Zensical directly with pip/uv, or by running the official Docker image with Podman or Docker — no local Python install required.

---

## Option 1: pip / uv (native install)

[Getting Started](https://zensical.org/docs/get-started/)

### Serve

Run this from the root of the repository (where your `zensical.toml` lives):

```bash
zensical serve
```

The site will be available at **http://localhost:8000** and will live-reload as you edit files.

---

## Option 2: Podman (no local Python required)

This uses the official `zensical/zensical` container image and mounts your working directory into it, so no local Python or Zensical install is needed.

### Prerequisites

- [Podman](https://podman.io/) installed

### Serve

Run this from the root of the repository:

```bash
podman run --rm -it --network host -v ${PWD}:/docs:z zensical/zensical serve
```

The site will be available at **http://localhost:8000**.

**Flag reference:**

| Flag | Purpose |
|---|---|
| `--rm` | Remove the container automatically when it stops |
| `-it` | Interactive terminal (lets you Ctrl+C to stop) |
| `--network host` | Share the host network so `localhost:8000` works directly |
| `-v ${PWD}:/docs:z` | Mount the current directory into the container; `:z` sets the correct SELinux label |

> **Windows / PowerShell note:** Replace `${PWD}` with `${PWD.Path}` or use the full absolute path, e.g. `C:\Users\you\project`.

### Docker (alternative)

If you prefer Docker over Podman, the command is identical — just swap `podman` for `docker`:

```bash
docker run --rm -it --network host -v ${PWD}:/docs:z zensical/zensical serve
```

> **macOS / Windows Docker Desktop note:** `--network host` is not supported on these platforms. Use `-p 8000:8000` instead:
> ```bash
> docker run --rm -it -p 8000:8000 -v ${PWD}:/docs zensical/zensical serve
> ```

---

## Troubleshooting

**Port already in use** — If port 8000 is taken, pass a different port:
```bash
zensical serve --port 8080
# or with container:
podman run --rm -it --network host -v ${PWD}:/docs:z zensical/zensical serve --port 8080
```

**Changes not reloading** — Make sure you are running from the directory that contains your `zensical.toml`. The dev server watches files relative to that location.

**SELinux errors (Podman)** — The `:z` volume flag handles this on most systems.
