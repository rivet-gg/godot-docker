# Godot + Docker

These Dockerfiles help get up and running with Godot 4.

## Upgrading Godot

The container can be built with

```bash
docker build --build-arg GODOT_VERSION=4.2 .
```

[See available Godot versions](https://github.com/godotengine/godot/releases/)

## Recommended usage

### Isolated Build Environments

Godot projects can be built quickly by mounting volumes like:

```bash
mkdir -p build/linux
docker run -v "$(pwd):/app" rivetgg/godot:4.2 \
    --export-release "Linux/X11" \
    --headless ./build/linux/game.x86_64
```

See also [godot-ci](https://github.com/abarichello/godot-ci) for Dockerfiles tailored for CI environments.

### Containerizing For Production

If running Godot as a dedicated server in production, we recommend using the following Dockerfile:

```dockerfile
# MARK: Builder
FROM rivetgg/godot:4.2 AS builder
COPY . .
RUN mkdir -p build/linux \
    && godot -v --export-release "Linux/X11" --headless ./build/linux/game.x86_64

# MARK: Runner
FROM ubuntu:22.04
RUN apt update -y \
    && apt install -y expect-dev \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/build/linux/ /app

# Unbuffer output so the logs get flushed
CMD ["sh", "-c", "unbuffer /app/game.x86_64 --verbose --headless -- --server | cat"]
```

## FAQ

### Why use Docker Hub instead of GitHub Container Registry?

GHCR requires authentication with your GitHub account in order to pull images. Many developer don't have this set up, so we opt to use Docker Hub, since it does not require authentication.

